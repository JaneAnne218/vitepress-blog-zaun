---
title: axios
description: axios
date: 2022-11-1
tags:
  - axios
---

## 1.  首先，在命令行中创建一个基本的Node.js项目，并安装axios库。
```shell
npm init -y npm install axios
```
## 2.  在项目根目录下创建一个`index.js`文件，并在其中编写代码。该代码会向一个API服务器发送GET请求，并输出响应数据。
```js
const axios = require('axios');

axios.get('https://jsonplaceholder.typicode.com/posts/1')
  .then(response => {
    console.log(response.data);
  })
  .catch(error => {
    console.error(error);
  });

```
## 3.  运行该文件，查看控制台输出结果。
```shell
node index.js
```
以上代码演示了如何使用axios库发起HTTP请求并处理响应数据