# Comebine

- [概念](#概念)
  - [函数式响应编程](#函数式响应编程)
- [特性](#特性)
- [使用场景](#使用场景)
- [核心概念](#核心概念)

# 概念
## 函数式响应编程
函数式响应编程（Functional reactive programming）也称为数据流编程，基于函数式编程的概念上。函数式编程应用于元素列表（元素的转变加工），而函数相应应用于元素流（元素的转变加工、流的分割合并；

一个随着时间处理数据的声明式的SwiftAPI

# 特性

将这些概念应用到Swift中只是Combine所做的一部分。Combine还可以通过加入控流(back-pressure)的概念扩展了函数响应式编程。控流是指控制接收和处理的信息的数量，这使得数据可控可取消的扩展概念得到更有效的实践。

# 使用场景
  - 你可以对一个按钮进行设置，只有当输入值是有效，才可以使按钮可用
  - 执行一些异步操作（比如检查网络服务），并且利用返回值来更新内容
  - 用来对用户在文本框中的动态输入做出回应，并且根据用户输入的内容更新视图内容

# 核心概念
  - Publisher(发布者)
  - Subscriber(订阅者)
  - Operators(操作者)
  - Subjects(对象)


# Operators

## receive
```
/// Specifies the scheduler on which to receive elements from the publisher.
///
/// You use the ``Publisher/receive(on:options:)`` operator to receive results and completion on a specific scheduler, such as performing UI work on the main run loop. In contrast with ``Publisher/subscribe(on:options:)``, which affects upstream messages, ``Publisher/receive(on:options:)`` changes the execution context of downstream messages.
///
/// In the following example, the ``Publisher/subscribe(on:options:)`` operator causes `jsonPublisher` to receive requests on `backgroundQueue`, while the
/// ``Publisher/receive(on:options:)`` causes `labelUpdater` to receive elements and completion on `RunLoop.main`.
///
///     let jsonPublisher = MyJSONLoaderPublisher() // Some publisher.
///     let labelUpdater = MyLabelUpdateSubscriber() // Some subscriber that updates the UI.
///
///     jsonPublisher
///         .subscribe(on: backgroundQueue)
///         .receive(on: RunLoop.main)
///         .subscribe(labelUpdater)
///
///
/// Prefer ``Publisher/receive(on:options:)`` over explicit use of dispatch queues when performing work in subscribers. For example, instead of the following pattern:
///
///     pub.sink {
///         DispatchQueue.main.async {
///             // Do something.
///         }
///     }
///
/// Use this pattern instead:
///
///     pub.receive(on: DispatchQueue.main).sink {
///         // Do something.
///     }
///
///  > Note: ``Publisher/receive(on:options:)`` doesn’t affect the scheduler used to call the subscriber’s ``Subscriber/receive(subscription:)`` method.
///
/// - Parameters:
///   - scheduler: The scheduler the publisher uses for element delivery.
///   - options: Scheduler options used to customize element delivery.
/// - Returns: A publisher that delivers elements using the specified scheduler.
```

使用receive可以更改下游消息执行的线程调度

例如：
receive(on: DispatchQueue.global())
下游操作符就会在全局子线程执行
receive(on: DispatchQueue.main)
下游操作符就会在主子线程执行

* 在调用assign时，记得调用receive(on: RunLoop.main)，保证在MainThread执行UI操作。否则会报以下错误。
```
[SwiftUI] Publishing changes from background threads is not allowed; make sure to publish values from the main thread (via operators like receive(on:)) on model updates.
```
