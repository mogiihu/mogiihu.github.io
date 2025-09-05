---
title: 完美解决 github push 失败
published: 2025-09-05
description: ''
tags: [git, 问题记录]
category: '问题记录'
draft: false 
lang: ''
---
在向 github 推送代码时，经常会遇到 push 失败的情况，报错信息如下：
```shell
fatal: unable to access ‘https://github.com/.../.git‘: Failed to connect to github.com port 443 after 21084 ms: Could not connect to server
```
### 通过配置hosts解决
host 来源：https://raw.hellogithub.com/hosts
复制该文件所有内容到 hosts 文件中。

### 各系统 hosts 文件位置与修改指南

hosts 文件在不同操作系统中的存放路径有所差异，具体如下：

- **Windows**：`C:\Windows\System32\drivers\etc\hosts`
- **Linux 系统**：`/etc/hosts`
- **macOS（苹果系统）**：`/etc/hosts`

### 修改步骤

将需要添加的内容复制到 hosts 文件的末尾段落：

- **Windows**：如无法保存，可以尝试将 hosts 复制到其他地方修改后粘贴进去覆盖原文件
- **Linux / macOS**：需要通过终端使用 Root 权限操作，例如执行：`sudo vi /etc/hosts`


### 如何使配置生效

一般情况下，修改保存后会自动生效。如果未生效，可以尝试以下方法刷新 DNS 缓存：

- **Windows**：打开命令提示符（CMD），输入：`ipconfig /flushdns`
- **Linux**：在终端中输入：`sudo nscd restart`。如果提示未找到命令，可先安装 nscd：`sudo apt install nscd`，或使用：`sudo /etc/init.d/nscd restart`
- **macOS**：在终端运行：`sudo killall -HUP mDNSResponder`

**提示**：如果上述操作仍未能使配置生效，可以尝试重启设备后再检查。

### 最后
尝试推送代码
```shell
git push
```
如果推送成功，恭喜你，问题解决了。






