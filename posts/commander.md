---
title: commander
description: 命令行工具
date: 2023-5-15
tags:
  - 工具
---
```js
//node 1.js -s / --first a/b/c  ls/d/lss
const { program } = require('commander');

program
  .option('--first')
  .option('-s, --separator <char>');

program.parse();

const options = program.opts();
console.log(program.args)//[ 'a/b/c', 'ls/d/lss' ]

const limit = options.first ? 1 : undefined;
console.log(options.first)// true
console.log(limit)// 1
console.log(program.args[1].split(options.separator, limit));//[ 'ls' ]

```
## bug
1.解决在linux全局安装jane-deploy时，bin目录下没有出现janedeploy的方法
```js
program//多加一个program
 .commond() 
```
