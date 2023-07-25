---
title: obsidian git插件的安装及使用
description: obsidian git插件的安装及使用
date: 2023-7-4
tags:
  - 工具
---
## 1. 安装obsidian git插件
1. obsidian的设置-第三方插件
## 2. 从GitHub下载库
1. ctrl+p 打开命令面板，输入git clone
![](./images/1-git-clone.png)
2. 输入GitHub上的库地址  
>参考: https://forum.obsidian.md/t/the-easiest-way-to-setup-obsidian-git-to-backup-notes/51429

![](./images/2-enter-remote-url.png)
```html
https://<PERSONAL_ACCESS_TOKEN>@github.com/<USERNAME>/<REPO>.git 
```
**这里的PERSONAL_ACCESS_TOKEN是在github的setting，选择setting-developer，选择personal access tokens中生成的token，需要勾选repo权限**  
3. 填入一个文件夹，这个文件夹必须是空的，如果不存在这个文件夹，会创建一个   
4. 等待下载完成    
5. **下载好的文件夹存在本地，可以用vscode、github desktop打开，这个文件夹就是一个git仓库，可以在vscode，github desktop，obsidian使用git命令了，最重要的是这三者是同步的，可以说是很方便了。**  
## 3. 优点 
todo list 可以点击完成以及未完成
![](./images/3-todo-done.png)
