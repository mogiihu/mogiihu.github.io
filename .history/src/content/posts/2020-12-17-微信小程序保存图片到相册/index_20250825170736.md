---
title: 微信小程序保存图片到相册
published: 2020-12-17 22:29:22
description: ''
image: ''
tags: [小程序, 前端, Taro]
category: '小程序'
draft: false 
lang: ''
---


在开发过程中经常遇到保存图片到相册的需求，主要有两种一是保存服务器地址图片，还有一种是保存 canvas 生成的 base64 格式图片，下面来讲解分别怎么实现。

## 保存服务器地址图片到相册

```js
// 查看是否有 `scope.writePhotosAlbum` 权限
wx.getSetting({
  success: (res) => {
    if (!res.authSetting["scope.writePhotosAlbum"]) {
      // 申请所需权限
      wx.authorize({
        scope: "scope.writePhotosAlbum",
        success: () => {
          // 授权成功保存图片
          saveToAlbum();
        },
        fail: () => {
          // 授权失败
        },
      });
    } else {
      // 用户到设置中同意保存相册权限后再次保存到相册
      saveToAlbum();
    }
  },
});

// 保存图片到相册
function saveToAlbum() {
  // 下载图片资源到本地
  wx.downloadFile({
    url: imgUrl,
    success: function (res) {
      // 保存图片到相册
      wx.saveImageToPhotosAlbum({
        filePath: res.tempFilePath,
        success(res) {
          // 保存成功
        },
        fail: () => {
          // 保存失败
        },
      });
    },
    fail: () => {
      // 下载失败
    },
  });
}
```

## 保存 base64 格式图片到相册

```js
// 查看是否有 `scope.writePhotosAlbum` 权限
wx.getSetting({
  success: (res) => {
    if (!res.authSetting["scope.writePhotosAlbum"]) {
      // 申请所需权限
      wx.authorize({
        scope: "scope.writePhotosAlbum",
        success: () => {
          // 授权成功保存图片
          saveToAlbum();
        },
        fail: () => {
          // 授权失败
        },
      });
    } else {
      // 用户到设置中同意保存相册权限后再次保存到相册
      saveToAlbum();
    }
  },
});

// 保存图片到相册
function saveToAlbum() {
  // 把base64的图片转化成ArrayBuffer数据
  const buffer = wx.base64ToArrayBuffer(
    img.replace(/^data:\w+\/\w+;base64,/, "")
  );
  // 指定图片的临时路径
  const path = `${wx.env.USER_DATA_PATH}/image_name.png`;
  // 获取小程序的文件系统
  const fsm = wx.getFileSystemManager();
  // 把arraybuffer数据写入到临时目录中
  fsm.writeFile({
    filePath: path,
    data: params,
    encoding: "binary",
    success: () => {
      // 把临时路径下的图片，保存至相册
      wx.saveImageToPhotosAlbum({
        filePath: path,
        success: () => {
          // 保存成功
        },
        fail: () => {
          // 保存失败
        },
      });
    },
    fail: () => {
      // 写入失败
    },
  });
}
```
