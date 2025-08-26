---
title: 实现通过短信中的链接打开小程序
published: 2021-01-25
description: ''
image: ''
tags: [小程序, 前端, Taro]
category: '小程序'
draft: false 
lang: ''
---

在开发小程序的过程中，我遇到了这样的一个需求，通过短信中的链接打开一个h5，这个h5可以实现点击跳转小程序，或者扫码跳转，并且将链接上的参数填充到小程序界面上。下面来描述一下我的开发步骤及实现方案。

这个需求大部分的功能是基于 h5 页面实现的，所以首先进行对 h5 页面的攻破。样式部分不多说，开始攻破两大功能点。

### 1. 通过点击按钮跳转小程序
点击按钮跳转即通过链接实现跳转，这里使用了小程序官方 api - [urlscheme.generate](https://developers.weixin.qq.com/miniprogram/dev/api-backend/open-api/url-scheme/urlscheme.generate.html)

在这个api中提到获取 URL Scheme 需要 access_token。

access_token，是小程序全局唯一后台接口调用凭据。需要通过官方 api - [auth.getAccessToken](https://developers.weixin.qq.com/miniprogram/dev/api-backend/open-api/access-token/auth.getAccessToken.html) 去调取，文档中提示 access_token 有效期是 2个小时，所以我将获取的 access_token 存储在 redis 中，并设定过期时间，避免反复调用造成 access_token 刷新，产生不必要的冲突。

下面是获取 accessToken 的代码实现：
```js
const getAccessToken = (appId, appSecret) => {
    return new Promise((resolve, reject) => {
        // 从 redis 中查找是否有 accessToken，如果 accessToken 过期，则获取不到 accessToken
        getRedisData(appId).then((accessToken) => {
            if (!!accessToken) {
                // 有 accessToken 则直接返回
                resolve(accessToken);
            } else {
                // 没有则通过接口重新获取 accessToken
                request({
                    url: `https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=${appId}&secret=${appSecret}`,
                    method: 'GET',
                    timeout: 5000
                }, (err, response, body) => {
                    body = JSON.parse(body);
                    if (err || !body) reject({errCode: -10, errInfo: err.toString()});
                    if (body.access_token) {
                        let accessToken = body.access_token;
                        // 获取后将 accessToken 存储在 redis 中，并设定过期时间，如果过期，则获取不到 accessToken
                        setRedisDataEx(appId, accessToken, body.expires_in);
                        resolve(accessToken);
                    } else {
                        reject({errCode: body.errcode, errInfo: body.errmsg});
                    }
                });
            }
        });
    });
}
```

下面是获取 URL Scheme 代码实现：
```js
const getGenerateScheme = (req, res) => {
    let { tenantCode, ...rest } = req.body;
    // rest 为文档中的剩余参数
    let cfg = config[tenantCode]; // cfg中 存储了小程序的 appId 及 appSecret
    if (!cfg) return res.send(getErrInfo(errCode.ERR_PARAMETER_UNAVAILABLE, 'tenantCode'))
    
    let { appId, appSecret } = cfg
    getAccessToken(appId, appSecret).then((accessToken) => { // 获取 accessToken
        const params = {
            ...rest
        }
        request({
            url: `https://api.weixin.qq.com/wxa/generatescheme?access_token=${accessToken}`,
            method: 'POST',
            timeout: 5000,
            headers: {
                'content-type': 'application/json',
            },
            body: JSON.stringify(params)
        }, (err, response, body) => {
            body = JSON.parse(body);
            if (err || !body) return res.send({errCode: -10, errInfo: err.toString()});
            if (body.openlink) {
                // 获取跳转链接 openlink
                let openlink = body.openlink;
                return res.send({errCode: 0, result: openlink});
            } else {
                return res.send({errCode: body.errcode, errInfo: body.errmsg});
            }
        });
    }).catch((err) => {
        return res.send(err);
    });
}
```
上述代码中 `rest`，即为文档中的参数。

调用 `getGenerateScheme` 接口后即可获得 URL Scheme，如：`weixin://dl/business/?t=LcoXo1wSXIq`。

iOS系统支持识别URL Scheme，可在短信等应用场景中直接通过Scheme跳转小程序。
Android系统不支持直接识别URL Scheme，用户无法通过Scheme正常打开小程序，我们需要使用H5页面中转，再跳转到Scheme实现打开小程序，跳转代码示例如下：

`location.href = 'weixin://dl/business/?t=LcoXo1wSXIq`
该跳转方法可以在打开H5时立即调用，也可以在用户触发事件后调用。

### 2. 通过生成的二维码图片扫码跳转小程序
生成二维码需要用到官方 api - [wxacode.getUnlimited](https://developers.weixin.qq.com/miniprogram/dev/api-backend/open-api/qr-code/wxacode.getUnlimited.html)。
下面是获取带参小程序二维码图片代码：
```js
const getWxaCode = (req, res) => {
    let { tenantCode, ...rest } = req.body;
    // rest 为文档中的剩余参数
    let cfg = config[tenantCode];
    if (!cfg) return res.send(getErrInfo(errCode.ERR_PARAMETER_UNAVAILABLE, 'tenantCode'))
    
    let { appId, appSecret } = cfg; // cfg中 存储了小程序的 appId 及 appSecret
    getAccessToken(appId, appSecret).then((accessToken) => { // 获取 accessToken
        const params = {
            ...rest
        }
        request({
            url: `https://api.weixin.qq.com/wxa/getwxacodeunlimit?access_token=${accessToken}`,
            method: 'POST',
            timeout: 5000,
            headers: {
                'content-type': 'image/jpeg',
            },
            encoding: null,
            body: JSON.stringify(params)
        }, (err, response, body) => {
            if (err || !body) return res.send({errCode: -10, errInfo: err.toString()});
            if (body) {
                const type = response.headers['content-type'];
                const img = `data:${type};base64,${body.toString('base64')}`;
                return res.send({errCode: 0, result: img});
            }
        });
    }).catch((err) => {
        return res.send(err);
    });
    
}
```
这里返回的 `body` 是 图片 Buffer 文件，Buffer是一个像Array的对象，但它主要用于操作字节，也就是二进制数据，它的元素为16进制的两位数，即0到255的数值。

request 发送请求的编码格式 encoding，会默认为 UTF-8，而对于不能识别的byte串会解码成�，通过将encoding设为null，也就不对原始数据编码，保持原始的二进制图片数据，即可得到 Buffer 文件的正确格式。

获取到 Buffer 文件后我又将其转换成 base64 的格式，以便前端直接使用。

至此这个 h5 的功能基本完成点击 [链接](https://service-support-test.ikandy.cn/h5/#/shareMiniapp?fullName=120&tenantCode=common_tenant&query=phone%3D17770542880&page=pages/videoSuccess/index
) 看看效果吧！如果如要通过短信发送打开，那么将长链接转换成短链接通过短信发送出去即可。

### 3. 接收跳转参数
通过接口获取的链接及二维码，在跳转时会将其余参数转换成页面 ? 后面的部分，形如 `page='pages/index/index?foo=bar'`。我们可以在小程序的 App.onLaunch、App.onShow 和 Page.onLoad 的回调函数中获取到。


