---
layout:     post
title:      swift中struct和class的区别
subtitle:   笔记
date:       2021/12/22
author:     HongWeiChen
header-img: img/blog-banner-dark.jpg
catalog: true
tags:
    - Swift
---

# struct和class的区别

类型不同，class是引用类型，struct是值类型，值类型和引用类型区别在于，值类型在赋值和传递时将进行复制，而引用类型则只会使引用对象的一个"指向"。

### class的优势

- 可以继承
- 类型转换可以在runtime的时候检查和解释一个实例的类型
- 可以用deinit来释放资源
- 一个类可以多次被引用

### struct的优势

- 结构较小，适用于复制操作，相比于一个class的实例被多次引用更加安全
- 无须担心内存Memory leak或者多线程冲突问题
