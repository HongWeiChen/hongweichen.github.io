---
layout:     post
title:      CocoaPods运行pod install时提示：CDN….Response: Couldn't connect to server
subtitle:   CocoaPods运行pod install时提示：CDN….Response: Couldn't connect to server
date:       2016/05/26
author:     HongWeiChen
header-img: img/blog-banner-dark.jpg
catalog: true
tags:
    - CocoaPods
---

# CocoaPods运行pod install时提示：CDN….Response: Couldn't connect to server

在运行pod install的时候提示
```
CDN….Response: Couldn't connect to server
```

解决方法：

打开podfile文件，在顶部新增
```
source 'https://github.com/CocoaPods/Specs.git'
```
