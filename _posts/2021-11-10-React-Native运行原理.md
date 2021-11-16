---
layout:     post
title:      React-Native运行原理
subtitle:   React-Native运行原理
date:       2021/11/10
author:     HongWeiChen
header-img: img/blog-banner-dark.jpg
catalog: true
tags:
    - React-Native
---

# 参考

- [React-Native运行原理](https://idmrchan.com/2019/10/12/react-native-principle-01/)

>这个 JSI 是由 C++ 实现的，可以认为 JSI 是一个简单版的 JS 引擎接口，同时连接 JS 和 Native，可以让 JS 保存对 c++ Host Object 的引用，也就是说不用将传递的消息序列化到 JSON，实现 JS 和 Native 的同步通信。

也就是说JSI是一个连接JS和Native的引擎接口，`可以让 JS 保存对 c++ Host Object 的引用`就是可以保存C++的对象实例数据，所以实际上是JS和Objective-C++的通信。

>所以整个 RN 的执行流程就是：
>>初始化：View/Bridge -> 加载/执行 JSBundle -> Bridge(JSI) -> Native

>>运行时：(e.g) 触摸屏幕 -> Native EventListen -> Bridge(JSI) -> JS xxxx -> Bridge(JSI) ……
