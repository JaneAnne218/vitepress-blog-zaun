---
title: http-proxy
description: http代理
date: 2022-7-10
tags:
  - 工具
---
## 1. 使用以下命令来初始化一个新的 Node.js 项目，并创建 `package.json` 文件：
```bash 
npm init -y
```
## 2. 安装 `http-proxy` 模块。在终端或命令提示符窗口中运行以下命令：
```shell
npm install http-proxy
```
## 3. 创建代理服务器的代码文件，并将上面提供的示例代码复制粘贴到该文件中。
```js
const http = require('http');
const httpProxy = require('http-proxy');

// 创建反向代理服务器
const proxyServer = httpProxy.createServer({
  target: 'http://example.com', // 设置目标服务器
});

// 启动代理服务器并监听8080端口
proxyServer.listen(8080, () => {
  console.log('代理服务器已启动，正在监听 8080 端口...');
});

```
## 4. 在终端或命令提示符窗口中执行以下命令来启动代理服务器：
```shell
node <filename>.js
```
其中，`<filename>` 是你的代码文件名（不包含扩展名）。
现在你可以在浏览器中访问 `http://localhost:8080`，代理服务器就会将请求转发到目标服务器上，并将响应返回给浏览器。
5. 问题
目标服务器，只能时http，不可以时https