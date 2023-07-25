---
title: vitepress的评论和搜索
description: vitepress的评论和搜索
date: 2023-7-10
tags:
  - 组件
---
参考链接:
http://f.zuo11.com/3-optimize/fast-create-website/
## 1. 评论valine
1. 教程
https://valine.js.org/cdn.html
2. leanCloud
https://console.leancloud.cn/apps
## 2. 搜索
1. 只添加了配置，搜索页面就出现了。
```js
 algolia: {
      appId: "xx",
      apiKey: "xxxxx",
      indexName: "xxx"
    },
```
2. 第二种添加搜索的方法
>参考：[jcamp-code](https://github.com/jcamp-code/vitepress-blog-theme/blob/main/docs/.vitepress/config.ts)
```js
 search: {
      provider: 'local',
    },
```
优点：省去了algolia的申请、配置，内容变化之后的爬虫

## feat：添加功能
>资料：https://blog.csdn.net/qq_42460209/article/details/125647447
1. 给网页添加标签页图标
```js
//添加到config.js
head: [["link", { rel: "icon", href: "..//watermelon.ico" }]]
```
2.关于标签页图标的尝试
```html
<!-- 在元素下查找favicon -->
<link rel="icon" href="//www.bilibili.com/favicon.ico">
```
>"favicon" 是 "favorite icon" 的缩写，指的是网站或应用程序在浏览器标签页、书签栏和其他界面中显示的小图标。它通常是一个小正方形的图标，用于识别特定网站或应用程序，并提供品牌识别和可视化标识。
网站的标签页图标是一个网址，我们无法拿到ico后缀的图片。
3. 内容更新后如何触发algolia爬取网站内容
  3.1 等24h之后自动爬取  
  3.2 手动爬取
  > 参考:[f.zuo11.com](http://f.zuo11.com/3-optimize/fast-create-website/#%E5%86%85%E5%AE%B9%E6%9B%B4%E6%96%B0%E5%90%8E%E6%80%8E%E4%B9%88%E8%A7%A6%E5%8F%91-algolia-%E9%87%8D%E6%96%B0%E7%88%AC%E5%8F%96%E5%86%85%E5%AE%B9)

  进入[官网](https://crawler.algolia.com/admin/crawlers )，点击Restart crawling即可。

  用申请algolia/ælˈɡoʊliə/的邮箱登录algolia，你会看到自己的网站，然后点击restart crawling就可以了。


## 图标库
1. font awesome
## Gittalk
1. comments.vue
>参考[个人站点-clark-cui](https://github.com/clark-cui/vitepress-blog-zaun/blob/53373e8baf38fd4fd4f99b313677b68c2d309aee/.vitepress/theme/components/Comments.vue#L4)

2. mylayout.vue
>参考[个人站点-clark-cui](https://github.com/clark-cui/vitepress-blog-zaun/blob/53373e8baf38fd4fd4f99b313677b68c2d309aee/.vitepress/theme/components/MyLayout.vue#L24)

3. 解析comment.vue代码
>参考[如何配置](https://www.jianshu.com/p/83a098b67977)

4. 获取clientID和clientSecret
>参考[动图](https://blog.csdn.net/xixihahalelehehe/article/details/125294535?ydreferer=aHR0cHM6Ly93d3cuZ29vZ2xlLmNvbS8%3D)

5. error not found
>参考[明察秋毫](https://blog.csdn.net/m0_46916422/article/details/124065600)
