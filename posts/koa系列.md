---
title: Koa
description: koa系列
date: 2023-7-10
tags:
  - node.js
--- >有效率的学习 看直播开发学习+文档+源码 
## 1. 给文件写内容  fs.writeFileSync(文件路径，文件内容)
```js
fs.writeFileSync(shellFilePath, shellText);
```

```
 C:\Users\janeAnne\Desktop\zuo-deploy\server\index.js
```

## 2. 前端发送请求
```js
//将 shell 内容存储到一个临时文件 temp.sh，用 node 执行该 shell
 axios.post('/runShell', { shellText: this.shellText, timeoutMinute: this.timeoutMinute })
            .then((response) => {
              // handle success
              console.log(response.data);
              let { code, msg } = response.data
              if (code !== 0) {
                this.$message.error(msg)
                return
              }
              this.$message.success('运行成功')// 找不到值
            })
            .catch(function (err) {
              console.log(err);
              this.$message.error(err.message)
            })
        }, 300),
```
来源：
```
C:\Users\janeAnne\Desktop\zuo-deploy\frontend\index.html
```
## 3.  koa-router响应请求
```js
 router.post("/runShell", async (ctx) => {
 //这个ctx是什么？
 let { shellText, shellName = "temp.sh", timeoutMinute = 2 } =     ctx.request.body;
 console.log("shellText", shellText);
 }
```
 来源：
 ```
  C:\Users\janeAnne\Desktop\zuo-deploy\server\index.js
```

## 4. CommonJS转化为ES6
4.1 package.json 添加"type": "module"  
4.2 文件名写为mjs  
4.3 文件中不能有require引入，需要改为import，导出用export  
```js
//ES6
// utils/runCmd.js
export default function runCmd() {
  // Your code here
}
import runCmd from "./utils/runCmd";
// commonJS
function runCmd(){
//Your code here
}
module.exports = runCmd;
const runCmd = require("./utils/runCmd");


```
此外，还可以把ES6的文件命名为.mjs，而commonJS命名为.cjs
ES6的默认导入和命名导入，这里出问题了，把默认导入，转化为命名导入，就解决了问题。
```js
// 导入所需的模块
import fs from "fs";
import logger from "./logger.js";
import { spawn } from 'child_process';

// 声明 runCmd 函数
function runCmd(cmd, args, callback, socketIo, msgTag = "common-msg", timeoutMinute = 2) {
  // 函数实现...
}

// 导出 runCmd 函数
export { runCmd };

```
命名式导入
```js
import { runCmd } from "./runCmd.js";

// 调用 runCmd 函数，并传递所需的参数
runCmd("ls", ["-l"], (output) => {
  console.log(output);
});

```

4.4 CommonJS 的特定变量 `__dirname`，在ES6中是未定义的。
```js
import { fileURLToPath } from "url";
import { dirname } from "path";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

```
4.5 
```js
// const server = require("http").Server(app.callback());
// const socketIo = require("socket.io")(server);
import http from "http";
//这个socket.io比较特殊，不是import socket from "socket.io" 或者
//import {socket} from "socket.io"
import { Server } from "socket.io"; 
const server = http.createServer(app.callback()); 
const socketIo = new Server(server); 
socketIo.on("connection", (socket) => { socketList.push(socket); logger.info("a user connected"); });
```
4.6默认导入和命名导入的区别
      一个模块可以同时有默认和命名两种
      默认在导入时可以去其他名字，命名有明确的名字
      默认不需要{}，命名需要{}
5.  common JS和ES6的区别
5.1 语法差异 导出和导入的语法不一样 import export/  module.export require
5.2 ES 6 静态导入和Common JS动态导入
5.3 浏览器兼容性  ES6 浏览器可以在浏览器中使用，而cjs最初是为node.js设计的，在浏览器中需要使用webpack转化代码
5.4 es6 用于现代js开发,commonjs用于node js

- [ ] socket 
- [ ] 实时shell
- [ ] 子线程?子进程?
- [ ] koa-session
- [ ] koa-bodyparser
- [ ] 不是ES没有别名么?
```js
import { join, resolve as _resolve } from "path";//错误
```  
举例
```JS
import { join, resolve } from "path";
const filePath = join("/root", "dir", "file.txt"); 
console.log(filePath); // 输出：/root/dir/file.txt
const absolutePath = resolve("relative/path/file.txt"); console.log(absolutePath); 
//输出：/current/working/directory/relative/path/file.txt`
```
6. 这的resolve是什么？
```js
let execFunc = () => {
    return new Promise((resolve, reject) => {
      try {
        runCmd(
          "sh",
          ["./deploy-master.sh"],
          function (text) {
            resolve(text);
          },
          socketIo,
          "deploy-log"
        );
      } catch (e) {
        logger.info(e);
        reject(e);
      }
    });
  };

```
7. SSH命令
```
ls -l 

```
1h27min 如何实现shell功能
如何清除日志：
```js
this.msgList.splice(0, this.msgList.length)
```
flex布局-垂直居中-两端对齐
```html
<div style="display: flex;align-items: center;justify-content: space-between;">
            <p>日志 <el-button size="small" @click="msgList.splice(0, msgList.length)">清空日志</el-button>
            </p>
</div>
```
行溢出换行
white-space:normal /no-wrap 
element-Tabs 实现tabs切换





