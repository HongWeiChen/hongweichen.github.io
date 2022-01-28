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

![](https://formidable.com/uploads/new-5.png)

>这个 JSI 是由 C++ 实现的，可以认为 JSI 是一个简单版的 JS 引擎接口，同时连接 JS 和 Native，可以让 JS 保存对 c++ Host Object 的引用，也就是说不用将传递的消息序列化到 JSON，实现 JS 和 Native 的同步通信。

也就是说JSI是一个连接JS和Native的引擎接口，`可以让 JS 保存对 c++ Host Object 的引用`就是可以保存C++的对象实例数据，所以实际上是JS和Objective-C++的通信。

>所以整个 RN 的执行流程就是：
- 初始化：View/Bridge -> 加载/执行 JSBundle -> Bridge(JSI) -> Native
- 运行时：(e.g) 触摸屏幕 -> Native EventListen -> Bridge(JSI) -> JS xxxx -> Bridge(JSI) ……

下面内容部分摘抄自参考文章里，主要从代码层面详解了初始化、运行时都做了什么事情，略过一些比较不重要的内容，主要核心提出个人对文章内容的理解以及重心。

# 初始化

```Objective-C
// AppDelegate.m
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  RCTBridge *bridge = [[RCTBridge alloc] initWithDelegate:self launchOptions:launchOptions];
  RCTRootView *rootView = [[RCTRootView alloc] initWithBridge:bridge
                                                   moduleName:@"demo"
                                            initialProperties:nil];

  rootView.backgroundColor = [[UIColor alloc] initWithRed:1.0f green:1.0f blue:1.0f alpha:1];

  self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
  UIViewController *rootViewController = [UIViewController new];
  rootViewController.view = rootView;
  self.window.rootViewController = rootViewController;
  [self.window makeKeyAndVisible];
  return YES;
}
```

这部分初始化也是iOS AppDelegate的入口，RCTBridge的初始化以及RCTRootView的初始化，其他没有什么好说的。

# RCTBridge

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7vh5z731jj30q80ggaah.jpg)

# Setup

```C
- (void)setUp
{
  ...
  self.batchedBridge = [[[RCTCxxBridge class] alloc] initWithParentBridge:self];
  [self.batchedBridge start];
	...
}
```

# Start

```Objective-C
// RCTCxxBridge.mm
- (void)start
{
  // 通知
  [[NSNotificationCenter defaultCenter]
    postNotificationName:RCTJavaScriptWillStartLoadingNotification
    object:_parentBridge userInfo:@{@"bridge": self}];

  // Set up the JS thread early
  // 开一个线程给 JS 使用
  _jsThread = [[NSThread alloc] initWithTarget:[self class]
                                      selector:@selector(runRunLoop)
                                        object:nil];
  _jsThread.name = @"com.facebook.react.JavaScript";
  // 线程最高优先级，用于用户交互事件
  _jsThread.qualityOfService = NSQualityOfServiceUserInteractive;
  [_jsThread start];

  dispatch_group_t prepareBridge = dispatch_group_create();

  [_performanceLogger markStartForTag:RCTPLNativeModuleInit];

  [self registerExtraModules];
  // Initialize all native modules that cannot be loaded lazily
  // 加载 JS 调用 Native 模块
  (void)[self _initializeModules:RCTGetModuleClasses() withDispatchGroup:prepareBridge lazilyDiscovered:NO];
  [self registerExtraLazyModules];

  __weak RCTCxxBridge *weakSelf = self;

  // Dispatch the instance initialization as soon as the initial module metadata has
  // been collected (see initModules)
  // 确定是否在 JS 线程，如果不是，指定在 JS 线程操作
  dispatch_group_enter(prepareBridge);
  [self ensureOnJavaScriptThread:^{
    [weakSelf _initializeBridge:executorFactory];
    dispatch_group_leave(prepareBridge);
  }];

  // Load the source asynchronously, then store it for later execution.
  // 加载资源
  dispatch_group_enter(prepareBridge);
  __block NSData *sourceCode;
  [self loadSource:^(NSError *error, RCTSource *source) {
    if (error) {
      [weakSelf handleError:error];
    }
    sourceCode = source.data;
    dispatch_group_leave(prepareBridge);
  } onProgress:^(RCTLoadingProgress *progressData) {}];

  // Wait for both the modules and source code to have finished loading
  // 加载完线程和 JS 模块，执行 JS
  dispatch_group_notify(prepareBridge, dispatch_get_global_queue(QOS_CLASS_USER_INTERACTIVE, 0), ^{
    RCTCxxBridge *strongSelf = weakSelf;
    if (sourceCode && strongSelf.loading) {
      [strongSelf executeSourceCode:sourceCode sync:NO];
    }
  });
}
```

文章中已经对代码增加了注释，这里在做一下总结：
1. 发出将要加载bridge事件的通知
2. 使用NSThread创建一个优先级最高的线程，提供给JS使用。
3. registerExtraModules可以直接看下注释来理解，这个不是很重要，因为大部分情况下没有用到
4. _initializeModules初始化JS调用的Native模块
5. registerExtraLazyModules(Debug下调用，先忽略)
6. ensureOnJavaScriptThread确认是否在JavaScript线程执行，如果不是就调到JavaScirpt的线程执行
7. 加载js
8. 所有都加载完毕后执行js

**ensureOnJavaScriptThread与loadSource都会有一个dispatch_group_level的动作，当这两个都执行完dispatch_group_level之后就会执行dispatch_group_notify中的dispatch_block_t里面的函数**

### registerExtraModules

>/**
 * The bridge initializes any registered RCTBridgeModules automatically, however
 * if you wish to instantiate your own module instances, you can return them
 * from this method.
 *
 * Note: You should always return a new instance for each call, rather than
 * returning the same instance each time the bridge is reloaded. Module instances
 * should not be shared between bridges, and this may cause unexpected behavior.
 *
 * It is also possible to override standard modules with your own implementations
 * by returning a class with the same `moduleName` from this method, but this is
 * not recommended in most cases - if the module methods and behavior do not
 * match exactly, it may lead to bugs or crashes.
 */

# 执行js

```C++
// JSIExecutor.cpp
void JSIExecutor::loadApplicationScript(
    std::unique_ptr<const JSBigString> script,
    std::string sourceURL) {
  SystraceSection s("JSIExecutor::loadApplicationScript");

  runtime_->global().setProperty(
      *runtime_,
      "nativeModuleProxy",
      Object::createFromHostObject(
          *runtime_, std::make_shared<NativeModuleProxy>(*this)));

  runtime_->global().setProperty(
      *runtime_,
      "nativeFlushQueueImmediate",
      Function::createFromHostFunction(
          *runtime_,
          PropNameID::forAscii(*runtime_, "nativeFlushQueueImmediate"),
          1,
          [this](
              jsi::Runtime &,
              const jsi::Value &,
              const jsi::Value *args,
              size_t count) {
            if (count != 1) {
              throw std::invalid_argument(
                  "nativeFlushQueueImmediate arg count must be 1");
            }
            callNativeModules(args[0], false);
            return Value::undefined();
          }));

  runtime_->global().setProperty(
      *runtime_,
      "nativeCallSyncHook",
      Function::createFromHostFunction(
          *runtime_,
          PropNameID::forAscii(*runtime_, "nativeCallSyncHook"),
          1,
          [this](
              jsi::Runtime &,
              const jsi::Value &,
              const jsi::Value *args,
              size_t count) { return nativeCallSyncHook(args, count); }));

  if (runtimeInstaller_) {
    runtimeInstaller_(*runtime_);
  }
  runtime_->evaluateJavaScript(
      std::make_unique<BigStringBuffer>(std::move(script)), sourceURL);
  flush();
}
```

>
- runtime_->global().setProperty 是在 JS 全局对象 global 上注册 Function 或 Object

>
- runtime_->evaluateJavaScript 则是调用了 iOS 内置的 JavaScript 引擎，解释执行 JS 代码。

>
- 而 runtime_ 是 JSI 的一个方法(?)，这里就和之前说的 执行 JS -> Bridge(JSI) 连上了。
