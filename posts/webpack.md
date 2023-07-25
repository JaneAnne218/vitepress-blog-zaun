---
title: webpack
description: webpack
date: 2022-7-10
tags:
  - webpack
---

## 1 . `webpack-dev-middleware`和`webpack-hot-middleware`中间件
在开发环境下支持热重载和自动刷新浏览器
```js
const express = require('express');
const webpack = require('webpack');
const webpackDevMiddleware = require('webpack-dev-middleware');
const webpackHotMiddleware = require('webpack-hot-middleware');

const app = express();
const config = require('./webpack.config'); // 定义 webpack 配置

config.entry.unshift('webpack-hot-middleware/client'); // 添加 HMR client 到 entry 数组的前面
config.plugins.push(new webpack.HotModuleReplacementPlugin()); // 添加 HMR 插件

const compiler = webpack(config); // 创建编译器实例

app.use(
  webpackDevMiddleware(compiler, {
    publicPath: config.output.publicPath,
  })
);

app.use(webpackHotMiddleware(compiler));

// 定义路由
app.get('/', (req, res) => {
  res.sendFile(__dirname + '/index.html');
});

app.listen(3000, () => {
  console.log('App listening on port 3000!');
});


```