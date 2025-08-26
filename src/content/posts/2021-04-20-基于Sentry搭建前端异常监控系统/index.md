---
title: 基于Sentry搭建前端异常监控系统
published: 2021-04-20
description: ''
image: ''
tags: [基建, 前端]
category: '基建'
draft: false 
lang: ''
---


## 背景

虽然在我们的项目上线前会有很多的测试流程，但是测试流程肯定无法保证 100%覆盖所有操作场景，在用户的使用过程中仍会有一些问题暴露出来。
但当线上用户出现问题，我们需要收到用户的一个反馈，才能去定位解决，这样会导致我们的问题解决不够及时。
并且有些疑难杂症，我们无法复现定位，这时候我们需要得知用户的环境、操作等信息，以便于对问题进行排查。
基于以上三点考虑，我认为在大部分项目中都需要接入一个异常监控系统，来实现收集异常、收集日志信息、及时警告、展示统计信息等功能。

## 为什么选择 Sentry？

- 免费
- Sentry 可直接使用也可自行搭建
- 兼容性强，基本不受语言限制，搭建一套系统可用于多个项目。
  ![Sentry可适用平台](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08056a3dbf414ba0a7ee7180952c4f96~tplv-k3u1fbpfcp-zoom-1.image)

## Sentry 简介

> Sentry's application monitoring platform helps every developer
> diagnose, fix, and optimize the performance of their code.

Sentry 的应用程序监视平台可帮助每个开发人员诊断，修复和优化其代码的性能。

## Sentry 部署及使用

Sentry 可直接使用也可自行搭建，下面将讲解 Sentry 的搭建过程。想要直接体验的可以跳过直接戳[官网](https://sentry.io/auth/login/)进行体验。

官方提供安装方式是使用 [Docker](https://www.docker.com/) 和 [Docker Compose](https://docs.docker.com/compose/) 以及基于 bash 的安装和升级脚本。

### 环境要求

- Docker 19.03.6+
- Compose 1.24.1+
- 4 CPU Cores
- 8 GB RAM
- 20 GB Free Disk Space
- Python 3

这里就不对环境的配置进行详细说明，想自己搭建的同学可以参考下以下文章。

- [yum 方式安装升级 docker](https://blog.csdn.net/weixin_37745913/article/details/99966937)
- [新装的 CentOS 7 安装 python3](https://blog.csdn.net/lovefengruoqing/article/details/79284573)

####

### 部署步骤

1. 从 github 上获取 Sentry 最新代码。

```shell
git clone https://github.com/getsentry/onpremise.git
```

2. 进入 onpremise ，运行 install.sh 脚本。

```shell
cd onpremise
./install.sh
```

3. 运行 `docker-compose up -d` 以启动 Sentry。

按照默认配置构建后，Sentry 默认绑定 9000 端口，访问 [http://127.0.0.1:9000](http://127.0.0.1:9000) 即可进入登录页面。

4. 打开页面填写信息后即可进入 Sentry 系统

![初始化界面](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4489c9938d9f4f5fb82d72273f878104~tplv-k3u1fbpfcp-zoom-1.image)

5. 自动发送警告邮件配置，可以从上图配置，也可以从配置文件配置，下面为通过 config.yml 配置教程。
   打开 onpremise/sentry 下的 config.yml 文件，修改 Mail Server 配置内容。![异常警告发送邮件配置](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59d7de2e95034ca6a3605d7f85344246~tplv-k3u1fbpfcp-zoom-1.image)

- mail.backend：邮件发送方式；
- mail.host: 邮件发送域名 ，使用的哪个邮箱可以去该邮箱文档中找到 smtp 发送域名；
- mail.port: 邮件发送的端口号；
- mail.username：用于 smtp 邮箱的账号；
- mail.password：用于 smtp 邮箱的密码；
- mail.use-tls：是否使用 tls 安全协议，这里填写 true 或 false，和 use-ssl 配置互斥；
- mail.use-ssl：是否使用 ssl 安全协议，这里填写 true 或 false，和 use-tls 配置互斥；
- mail.from：收到邮件时的发送人名称；

6. 通过配置文件修改邮件服务后，需要运行 `docker-compose up -d`  更新 Sentry，如果更新失败则需要关闭 docker-compose 后重新运行。

```shell
docker-compose down
docker-compose build
docker-compose up -d
```

重启后可通过 头像-> admin -> mail 进入 mail 配置页，点击发送测试邮件来检测是否配置成功。

![测试邮件服务配置](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efb59c11088a471da58ed76146b768ca~tplv-k3u1fbpfcp-zoom-1.image)

## Sentry 设置通过 HTTPS 访问

有没有发现上面 Sentry 的启动地址还有 DSN 地址都是 http，现在要给 Sentry 配置 ssl 让我们能通过 https 使用。

首先要修改 onpremise/sentry/config.yml `system.url-prefix` 配置，将其设置为我们访问的 Sentry 域名。 `url-prefix` 组成了项目的 DSN 地址，一定要保证格式正确。

```
system.url-prefix: 'https://sentry.xxxxx.com:60000'
```

然后是 onpremise/sentry/sentry.conf.py 文件下的 SSL/TLS 配置，将原来注释的部分全部打开。
![修改 SSL/TLS 配置](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e5675d158ce44948df9c4e732aafd87~tplv-k3u1fbpfcp-zoom-1.image)
修改完后同样要将 docker-compose 关掉重建。

```shell
docker-compose down
docker-compose build
docker-compose up -d
```

上面的配置只是让 Sentry 允许通过 SSL 代理，下面我们需要在服务器内搭建一个 nginx 用来转发 https 协议内容，搭建的过程不多说，百度或者请教运维大大。下面是我的 nginx 配置，配好 nginx 并重启后就算大功告成。我们的 Sentry 终于可以通过 https 访问了！

```nginx
server {
    # 配置监听端口
    listen       60000 ssl http2 default_server;
    listen       [::]:60000 ssl http2 default_server;

    # 域名
    server_name  sentry.xxxxx.cn;

    # nginx 默认根目录
    root         /usr/share/nginx/html;

    # 加载其他配置文件，这里是 nginx 默认配置，可以不需要
    include /etc/nginx/default.d/*.conf;

    # ssl 设置
    ssl on;
    # ssl 证书地址
    ssl_certificate     /etc/nginx/sslcert/server.crt;
    # ssl 密钥地址
    ssl_certificate_key /etc/nginx/sslcert/server.key;
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
    ssl_prefer_server_ciphers on;


    client_max_body_size 200M;
    client_body_buffer_size 1024k;

    # 这一段是最重要的，将域名代理到本机 http://localhost:9000 服务上，对应的就是 docker 内的 sentry 服务
    location / {
        proxy_pass http://localhost:9000;
    }

    gzip  on;
    gzip_http_version 1.1;
    gzip_vary on;
    gzip_comp_level 9;
    gzip_proxied any;
    gzip_types text/plain  text/css application/json  application/x-javascript text/xml application/xml application/xml+rss text/javascript application/x-shockwave-flash image/png image/x-icon image/gif image/jpeg;
    gzip_buffers 16 8k;

    error_page 404 /404.html;
        location = /404.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

## Sentry 接入实例

通过上面的过程 Sentry 系统就搭建好了，访问配置的地址即可进入 Sentry 系统。没有搭建的朋友们，也可以直接通过[官网](https://sentry.io/auth/login/)体验下面接入项目的过程。

### 创建项目

进入系统后点右上角的 `New Project`  创建项目。

![Sentry 创建项目](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/010301c5088049d1b322932ce42439a7~tplv-k3u1fbpfcp-zoom-1.image)

这里以 React 项目为例，建立一个 React 项目，按照页面提示即可在项目中注册 Sentry，对系项目执行中出现的异常进行捕捉。

![Sentry接入项目及初始化](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ad44d8621ba467c8d0da3b6d1a6371e~tplv-k3u1fbpfcp-zoom-1.image)

### 配置

Sentry 配置应在应用程序的生命周期中尽早进行。完成此操作后，Sentry 的 JavaScript SDK 会捕获所有未处理的异常和事务。init 参数详解：

```javascript
Sentry.init({
  // Sentry 项目的 dsn，可从项目设置中获取
  dsn: "https://xxxxxx@161.xxx.xxx.xxx:9000/2",
  // 初始参数配置内容
  integrations: [new Integrations.BrowserTracing()],
  // 触发异常后发送给 Sentry 的概率
  tracesSampleRate: 1.0,
  // 控制应捕获的面包屑(行为栈)的总量
  maxBreadcrumbs: 20,
  // 规定上下文数据结构的深度，默认为 3
  normalizeDepth: 100,
  // 版本信息
  release: "common@1.0.0",
  // 环境信息
  environment: process.env.NODE_ENV,
  // 钩子函数，在每次发送 event 前触发
  beforeSend(event) {
    // 网页应用刷新后设置的变量会消失，所以我选择在 beforeSend 触发时插入用户信息
    event.user = {
      userNick: "xiaohu",
    };
    return event;
  },
});
```

### 设置变量

在 Sentry 中，可以使用 `set` + `tags`，`extra`，`contexts`，`user`，`level`，`fingerprint` 形式，设置变量。下面将简单介绍几种方式，详情使用方法可见[文档](https://docs.sentry.io/platforms/javascript/enriching-events/context/#clearing-context)。

#### 通过 setUser 设置全局变量用户信息示例（不建议使用）

捕捉异常还需要采集用户信息，在用户登录后需要通过 `setUser` 设置一下用户信息全局变量，如下所示。**注意，通过这种方式设置的全局变量在页面刷新后会消失。**设置用户信息代码：

```javascript
Sentry.setUser({
  tenant: {
    code: 12345,
    name: "测试公司",
    _id: 12345,
  },
  orgAccount: {
    _id: 54321,
    orgName: "是机构啦",
  },
  user: {
    _id: "8910JQ",
    loginName: "测试人员小Q",
  },
});
```

设置全局变量后触发的 issue 中，可看到上传的用户数据。

![自定义 user 数据展示](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/787674fe40cb458fb4f2be0165046ab5~tplv-k3u1fbpfcp-zoom-1.image)

#### 通过 beforeSnd 插入用户信息

通过 Sentry 机制设置的全局变量在页面刷新后会消失，这边我建议通过 `beforeSend`  函数，修改 event 中的数据来插入需要的全局变量。

```javascript
Sentry.init({
    ...,
 	  // 钩子函数，在每次发送 event 前触发
    beforeSend(event) {
        // 在这里可根据业务情况发送用户信息
        event.user = {
            userNick: 'xiaohu'
        };
        return event;
    }
});
```

####

#### 设置全局变量

```javascript
// 以下是 Sentry 定义的全局变量，可以直接使用 Sentry api 设置
Sentry.setUser(object);
Sentry.tags(object);
Sentry.extra(object);
Sentry.level(object);
Sentry.fingerprint(object);

// 通过 setContext，设置 key 值，可自定义随事件传递的变量名
Sentry.setContext(key, context);
```

#### 设置局部变量

`captureException` 是我常用的一种设置局部变量并上传报错的方式，`Sentry.captureException(err[, obj])` 第一个参数为抛出的异常，第二个参数可附加错误信息。
第二个参数为对象，key 值可为 `tags`，`extra`，`contexts`，`user`，`level`，`fingerprint` 这 6 种。

```javascript
Sentry.captureException(error, {
  contexts: {
    message: {
      a: 1,
      b: { b: 1 },
    },
  },
});
```

#### 设置局部变量网络报错信息示例

根据目前前端框架所用的 axios，对网络请求出错的局部数据进行收集，代码示例：

```javascript
Sentry.captureException(error, {
  contexts: {
    message: {
      url: error.response.config.baseURL + error.response.config.url,
      data: error.response.config.data,
      method: error.response.config.method,
      status: error.response.status,
      statusText: error.response.statusText,
      responseData: JSON.stringify(error.response.data),
    },
  },
});
```

#### 清除全局变量

在用户退出登录后需要清除该用户的用户信息，有以下两种方式。

1. 通过设置为空，可以清除设置过的数据。

```javascript
Sentry.setUser();
```

2. 通过 `scope.clear()`  清除全局变量。

```javascript
Sentry.configureScope((scope) => scope.clear()); // 清除所有全局变量
Sentry.configureScope((scope) => scope.setUser(null)); // 清除 user 变量
```

### 上传日志信息

有时我们不仅仅要收集异常信息，还需要在页面中打 log 来收集页面运行数据，这时可以用 `Sentry.captureMessage(err[, obj])` api，进行传输日志。使用方法与 `captureException` 一致，建议将 level 设置为 Info，便于与异常区分开来，避免触发我们设置的异常警报。

```javascript
Sentry.captureMessage("Something went fundamentally wrong", {
  contexts: {
    text: {
      hahah: 22,
    },
  },
  level: Sentry.Severity.Info,
});
```

### 上传 sourceMap

Sentry 可将前端项目打包后的 SourceMap 上传，方便我们查看异常所在的代码位置。
[Sentry 集成 sourcemap](https://blog.csdn.net/weixin_34996214/article/details/112278553)

### 卸载 sentry

需要卸载可参考以下链接。
[如何卸载 Sentry？](https://segmentfault.com/q/1010000038449410)

## 触发一个异常看看

```js
onClick={() => {
	let a = {};
	console.log(a.b.c);
}}
```

在这里我制造了一个异常代码，点击后大家可想而知会出现一个 js error，如下图。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85ffec26dd974294a9bdde11c52f0856~tplv-k3u1fbpfcp-zoom-1.image)

在 network 中我们可以看到一条请求，Sentry 就是通过这样的方式上传错误的。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97a4b5ce5d7842de860ab4bf2778f2d5~tplv-k3u1fbpfcp-zoom-1.image)

然后转到 Sentry 系统页面上，可以看到 Issues 面板中多了一条信息：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebee098df308415e99d6933c69b0931b~tplv-k3u1fbpfcp-zoom-1.image)

点击进入后可以看到关于这条异常的详细信息。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42825f5c56b54ed399e71d65bd059301~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3efa7587b00492c9e8ada110b3ab9cb~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adde805d5ee64905ab42debfe541ada6~tplv-k3u1fbpfcp-zoom-1.image)

可以看到，详情里收集了大量的异常相关数据，如设备、浏览器版本、ip 地址、error 数据、用户操作栈等数据。但光有这些数据是不够的，原始的上传信息有时也不足以分析问题，很多时候我们需要根据业务场景定制所采集的数据。下面我们将从捕捉场景和采集内容这两方面进行分析。

## 异常信息捕捉场景及采集内容

### 捕捉场景

针对前端常见业务场景，总结了以下几个场景需要收集业务报错信息。

1. 接口错误
2. 网络错误
3. js error 报错
4. 主要业务流程异常状态
5. iframe/webview 内部错误

以上异常如果触发了 js error 或者 Promise.reject 并且没有被抛出，会被 Sentry 自动捕捉到，如果没有自动触发可手动 new Error 或 Promise.reject(error) 抛出异常，即可被 Sentry 收集。或者使用 captureMessage Api 收集运行日志。

### 采集内容

尽管 Sentry SDK 会捕获所有未处理的异常和事务汇报至系统上，并且会记录产生异常的页面及设备信息。但是有这些信息是远远不够的，还需要额外的业务信息，用于更好的区别定位问题。下面是我对上报异常所需要收集的内容进行总结。

异常信息：抛出异常的 error 信息，Sentry 会自动捕捉。

用户信息：用户的信息（登录名、id、level 等），所属机构信息，所属公司信息。

行为信息：用户的操作过程，例如登陆后进入 xx 页面，点击 xx 按钮，Sentry 会自动捕捉。

版本信息：运行项目的版本信息（测试、生产、灰度），以及版本号。

设备信息：项目使用的平台，web 项目包括运行设备信息及浏览器版本信息。小程序项目包括运行手机的手机型号、系统版本及微信版本。

时间戳：记录异常发生时间，Sentry 会自动捕捉。

异常等级：根据 Sentry 文档分为以下几个等级。

- Critical
- Debug
- Error
- Fatal
- Info
- Log
- Warning

平台信息：记录异常发生的项目。

有以上内容的补全，再加上原有的异常数据，对于一个报错是不是就更清晰明了了呢。

## 小程序接入 Sentry

Sentry 无官方 api 用于提供给小程序，好在有第三方根据提供的库，可使用原有 api 实现小程序端的异常监控。

### 安装

- npm install sentry-mina --save
- yarn add sentry-mina
- 将 browser/sentry-mina.js 拷贝到项目中

### 使用

```javascript
import * as Sentry from "sentry-mina/browser/sentry-mina.js";
// import * as Sentry from "sentry-mina";
// config Sentry
Sentry.init({
  dsn: "",
});
// Set user information, as well as tags and further extras
Sentry.configureScope((scope) => {
  scope.setUser({ id: "4711" });
  scope.setTag("user_mode", "admin");
  scope.setExtra("battery", 0.7);
  // scope.clear();
});
// Add a breadcrumb for future events
Sentry.addBreadcrumb({
  message: "My Breadcrumb",
  // ...
});
// Capture exceptions, messages or manual events
Sentry.captureMessage("Hello, world!");
Sentry.captureException(new Error("Good bye"));
Sentry.captureEvent({
  message: "Manual",
  stacktrace: [
    // ...
  ],
});
```

其他内容见文档 [Sentry 小程序 SDK](https://developers.weixin.qq.com/community/develop/doc/000cecefb009a0a951c727d6c55406)。

## 问题总结

下面是我在接入 Sentry 及后续使用时遇到的问题。

### consolelog 不显示 sourmap 地址

![consolelog 不显示 sourmap 地址](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3dbbbba40de1411f96a4fa0d629ff2e1~tplv-k3u1fbpfcp-zoom-1.image)

这个问题产生的原因是，Sentry 会修改原生 console 方法收集 console，所以这边不会显示 sourcemap 地址，可以在开发环境下取消注册 Sentry。

## 结尾

以上所述就是作者给大家介绍的基于 Sentry 搭建前端异常监控系统的过程，希望对大家有所帮助，如果大家有任何疑问请给我留言，我会及时回复大家的！
