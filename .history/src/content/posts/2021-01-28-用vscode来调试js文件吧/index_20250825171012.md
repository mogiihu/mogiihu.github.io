---
title: 用vscode来调试js文件吧
published: 2021-01-28
description: ''
image: ''
tags: [工具, VSCode]
category: '工具'
draft: false 
lang: ''
---

### 前言

最近经常在 LeetCode 上刷算法题，又菜又舍不得开会员在线调试，就可怜怜巴巴的用 chore 的控制台缓慢的调试。偶然发现 VSCode 也可以对 js 文件进行调试，并且拥有打断点、监听变量等功能，那就赶快弄起来吧~

### 操作

首先点击 run 面板 `create a launch.json file`，选择 node.js，VSCode 会帮我们在 `.vscode` 的文件夹中自动创建一个 launch.json 文件，
![](./img1.png)
![](./img2.png)

自动创建好的文件代码如下：

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "skipFiles": ["<node_internals>/**"],
      "program": "${file}"
    }
  ]
}
```

注意这里 `program` 是其他值的需要修改为 `${file}`。

然后打开我们需要运行的 js 文件按下 F5 或者 run 面板上的运行按钮就可以直接运行这个文件了。
![](./img3.png)

### 调试界面

![](./img4.png)
如上图所示调试过程中可以打断点进行进程调控。

VARIABLES 会显示变量信息。

WATCH 面板可以对变量进行监听或修改。

CALL STACK 显示的是当前调用堆栈。

BREAKPOINTS 可以对断点进行调控。

TERMINAL 中也会多出一个 DEBUG CONSOLE，显示我们 js 代码中 console 的内容。

### ENDING

拥有了这个功能调试 js 如有神助，妈妈再也不用担心我肝 chore 了，又节省了更多时间可以拿来下棋了呢~
