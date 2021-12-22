---
layout:     post
title:      MRC下retain所影响retainCount的变化
subtitle:   剖析retain、strong、retainCount
date:       2021/11/15
author:     HongWeiChen
header-img: img/blog-banner-dark.jpg
catalog: true
tags:
    - iOS
---

# MRC下的一些测试

```
Foo *f = [[Foo alloc] init];
NSLog(@"%d", [f retainCount]);
```

> 输出结果是1

```
@property (retain, nonatomic) Foo *retainFoo;

self.retainFoo = [[Foo alloc] init];
NSLog(@"%d", [self.retainFoo retainCount]);
```

> 输出结果是2

```
@property (strong, nonatomic) Foo *strongFoo;

self.strongFoo = [[Foo alloc] init];
NSLog(@"%d", [self.strongFoo retainCount]);
```

> 输出结果是2

由此可见，在MRC模式下，strong与retain并没有什么太大区别，而在ARC模式下，retain和strong使用起来也不会存在什么问题。

# 换一种方式进行测试

```
@property (retain, nonatomic) Foo *retainFoo;

_retainFoo = [[Foo alloc] init];
NSLog(@"%d", [self.retainFoo retainCount]);
```

> 输出结果是1

```
@property (strong, nonatomic) Foo *strongFoo;

_strongFoo = [[Foo alloc] init];
NSLog(@"%d", [self.strongFoo retainCount]);
```

> 输出结果是1

# 为什么retainCount会是2？换成下划线又会是1？

self本身会进行一次retain引用计数自增，而[[Foo alloc] init]这行代码也会产生一次引用计数自增的动作，所以retainCount会是2，而当我们使用下划线来赋值时，不会调用内存修饰符retain的行为。

# 内部是如何处理的？

retain的内部是将旧值release，重新赋值，并且retain，相当于是
```
[foo release];
foo = newObj;
[foo retain];
```
执行了这样的一个行为，而newObj本身在传递进来的时候，retainCount已经是1了，所以在retain的时候就变成了2
