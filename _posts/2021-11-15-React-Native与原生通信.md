---
layout:     post
title:      React-Native与原生通信
subtitle:   React-Native与原生通信
date:       2021/11/15
author:     HongWeiChen
header-img: img/blog-banner-dark.jpg
catalog: true
tags:
    - React-Native
---

# 前言

延续上一章React-Native运行原理，这里依旧是参考大佬文章来做详解，加深自己理解和印象，也不能死记硬背。

# 参考

- [Native和RN通信](https://idmrchan.com/2019/10/14/react-native-principle-02/)

# RCT_EXPORT_MODULE/RCT_EXPORT_METHOD

因为我个人从事主要还是原生端的偏多，我先从原生端的这两个比较重要的方法开始讲起

# RCT_EXPORT_MODULE

```
#define RCT_EXPORT_MODULE(js_name) \
RCT_EXTERN void RCTRegisterModule(Class); \
+ (NSString *)moduleName { return @#js_name; } \
+ (void)load { RCTRegisterModule(self); }
```

RCT_EXPORT_MODULE代表了两个方法

- moduleName
- load

# moduleName

用于获取类名称

# load

```Objective-C
void RCTRegisterModule(Class moduleClass)
{
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    RCTModuleClasses = [NSMutableArray new];
    RCTModuleClassesSyncQueue = dispatch_queue_create("com.facebook.react.ModuleClassesSyncQueue", DISPATCH_QUEUE_CONCURRENT);
  });

  RCTAssert([moduleClass conformsToProtocol:@protocol(RCTBridgeModule)],
            @"%@ does not conform to the RCTBridgeModule protocol",
            moduleClass);

  // Register module
  dispatch_barrier_async(RCTModuleClassesSyncQueue, ^{
    [RCTModuleClasses addObject:moduleClass];
  });
}
```

RCTRegisterModule则是将模块的Class进行存储到一个数组中。

**补充一点：load是系统方法，是在pre-main的时候运行的，在对RCTBridge进行初始化时，就已经将所有Module类名进行存储**


# RCT_EXPORT_METHOD

```Objective-C
#define _RCT_EXTERN_REMAP_METHOD(js_name, method, is_blocking_synchronous_method) \
  + (const RCTMethodInfo *)RCT_CONCAT(__rct_export__, RCT_CONCAT(js_name, RCT_CONCAT(__LINE__, __COUNTER__))) { \
    static RCTMethodInfo config = {#js_name, #method, is_blocking_synchronous_method}; \
    return &config; \
  }
```

RCT_EXPORT_METHOD是一个嵌套宏，最终执行的宏函数是_RCT_EXTERN_REMAP_METHOD，其实就是返回一个RCTMethodInfo的结构体，返回结构体的地址。

>RN 提供了`RCT_EXPORT_MODULE`和`RCT_EXPORT_METHOD`两个宏，用于暴露模块方法，模块名默认是方法类名。

[官方文档](https://reactnative.dev/docs/native-modules-ios)

# 实现原理

**JS方面部分内容摘自参考文章**

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g7xskerihoj30sx0moq3n.jpg)

```js
// index.bundle.js
var NativeModules = _$$_REQUIRE(_dependencyMap[0], "../../BatchedBridge/NativeModules");

// BatchedBridge/NativeModules.js
let NativeModules: {[moduleName: string]: Object} = {};
if (global.nativeModuleProxy) {
  NativeModules = global.nativeModuleProxy;
} else if (!global.nativeExtensions) {
  // ...从其他人写的文章来看，之前的版本和 Debug JS Remote 走这里
}
```

>之前说到 JS 调用 Native，实际调用的是 NativeModules。这个 global.nativeModuleProxy 是 Native 注册的方法，可以翻回 ReactNative运行原理执行 JS 这段。

[React-Native运行原理](https://hongweichen.github.io/2021/11/10/React-Native运行原理/)

有个疑问：所以其实NativeModules等价于global.nativeModuleProxy?直接用global.nativeModuleProxy调用等价于NativeModules调用？

```C++
// JSINativeModules.cpp
folly::Optional<Object> JSINativeModules::createModule(
    Runtime& rt,
    const std::string& name) {
  if (!m_genNativeModuleJS) {
    // 调用 global.__fbGenNativeModule，该方法在 NativeModule.js 实现
    m_genNativeModuleJS =
        rt.global().getPropertyAsFunction(rt, "__fbGenNativeModule");
  }
  auto result = m_moduleRegistry->getConfig(name);
  if (!result.hasValue()) {
    return folly::none;
  }
  Value moduleInfo = m_genNativeModuleJS->call(
      rt,
      valueFromDynamic(rt, result->config),
      static_cast<double>(result->index));
  CHECK(!moduleInfo.isNull()) << "Module returned from genNativeModule is null";
  folly::Optional<Object> module(
      moduleInfo.asObject(rt).getPropertyAsObject(rt, "module"));
  return module;
}
```

```js
// NativeModule.js

// export this method as a global so we can call it from native
global.__fbGenNativeModule = genModule;

function genModule(config, moduleID) {
  const [moduleName, constants, methods, promiseMethods, syncMethods] = config;
  const module = {};
  methods &&
    methods.forEach((methodName, methodID) => {
     	// ...
      module[methodName] = genMethod(moduleID, methodID, methodType);
    });
  Object.assign(module, constants);

  // ...
  return {name: moduleName, module};
}
```

```js
function genMethod(moduleID: number, methodID: number, type: MethodType) {
  fn = function(...args: Array<any>) {
    return new Promise((resolve, reject) => {
      BatchedBridge.enqueueNativeCall(
        moduleID,
        methodID,
        args,
        data => resolve(data),
        errorData => reject(createErrorFromErrorData(errorData)),
      );
    });
  };
  fn.type = type;
  return fn;
}

function enqueueNativeCall(moduleID, methodID, params, onFail, onSucc) {
  this.processCallbacks(moduleID, methodID, params, onFail, onSucc);

  this._queue[MODULE_IDS].push(moduleID);
  this._queue[METHOD_IDS].push(methodID);
  this._queue[PARAMS].push(params);
	// ...
  const now = Date.now();
  if (
    global.nativeFlushQueueImmediate &&
    now - this._lastFlush >= MIN_TIME_BETWEEN_FLUSHES_MS	// MIN_TIME_BETWEEN_FLUSHES_MS = 5
  ) {
    const queue = this._queue;
    this._queue = [[], [], [], this._callID];
    this._lastFlush = now;
    global.nativeFlushQueueImmediate(queue);
  }
  // ...
}
```

```C++
// JSIExecutor.cpp
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

// NativeToJsBridge.cpp
void callNativeModules( __unused JSExecutor& executor, folly::dynamic&& calls, bool isEndOfBatch) override {
  for (auto& call : parseMethodCalls(std::move(calls))) {
    m_registry->callNativeMethod(call.moduleId, call.methodId, std::move(call.arguments), call.callId);
  }
}

// ModuleRegistry.cpp
void ModuleRegistry::callNativeMethod(unsigned int moduleId, unsigned int methodId, folly::dynamic&& params, int callId) {
  if (moduleId >= modules_.size()) {
    throw std::runtime_error(
      folly::to<std::string>("moduleId ", moduleId, " out of range [0..", modules_.size(), ")"));
  }
  modules_[moduleId]->invoke(methodId, std::move(params), callId);
}
```

这边代码很绕，基本就是C++和js代码反复横条，这边来总结一下。

1. `rt.global().getPropertyAsFunction(rt, "__fbGenNativeModule")`C++调用js`global.__fbGenNativeModule`
2. `__fbGenNativeModule`在js中是个全局函数，实际上调的是genModule
3. genModule调用genMethod，genMethod内部执行的`BatchedBridge.enqueueNativeCall`，这边就是js调用native的方法
4. `global.nativeFlushQueueImmediate(queue)`js调用C++函数
5. nativeFlushQueueImmediate调用callNativeModules，callNativeModules调用callNativeMethod，最终走到`modules_[moduleId]->invoke(methodId, std::move(params), callId);`函数中。
6. modules_根据moduleId取到对应的类，调用方法并且传递参数，至此，js调用RN方法完成。

modules_是RCTBridge初始化的方法列表，而方法列表是通过最开始RCT_EXPORT_MODULE来注册的。在文章开头也写了RCT_EXPORT_MODULE代码详解，这里就不重复写了。
