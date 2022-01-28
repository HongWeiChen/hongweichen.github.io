---
layout:     post
title:      CocoaPods连接不上服务器
subtitle:   处理以及应对相关方法
date:       2016/05/26
author:     HongWeiChen
header-img: img/blog-banner-dark.jpg
catalog: true
tags:
    - CocoaPods
---

由于我们开发中使用的一些第三方框架基本是存储于Github或者国外某个代码仓库，所以我们在使用CocoaPods调用install命令时，经常会出现：

*CDN….Response: Couldn't connect to server*

也就是无法连接上服务器的提示

这时候我们打开项目文件夹中的Podfile文件，在文件顶部新增：

*source 'https://github.com/CocoaPods/Specs.git'*

就可以解决问题。

**source这个命令是指定库来源**。
