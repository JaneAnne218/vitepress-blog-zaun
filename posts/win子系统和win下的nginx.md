---
title: win子系统和win下的nginx
description: win子系统和win下的nginx
date: 2023-6-10
tags:
  - win子系统
---
# 1. win 下安装 ubuntu 子系统
在win商店安装
## 1.1 启用/关闭 win 系统子功能
1. 打开 适用 Linux 的 win 子系统  
2. 打开虚拟机平台  

>查看 npm 的安装源
> npm config get registry    
>设置淘宝镜像  
> npm config set registry https://registry.npm.taobao.org  
>设置为原镜像  
>npm config set registry https://registry.npmjs.org/  
>查看全局安装包   
>npm list -g --depth=0  
>删除 npm 包  
>npm uninstall <包名>    
>npm uninstall -g <包名>
>给密码 sudo  
>echo 密码 | sudo -S npm uninstall -g jane-deploy 

**ubuntu server 不安装 desktop，慢**

Delete `␍` 如何修改??  
axios /ˈæk.si.oʊs/    
parameter /_pəˈræmɪtə(r)_/ 
# 2. win 也可以安装 nginx
启动 nginx.exe 即可，访问localhost:80

