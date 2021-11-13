# NSObject

我们经常使用`[[NSObject alloc] init]`这类的方法

今天就来研究一下NSObject初始化底层的实现

首先看下alloc这个方法都做了什么，在很多网站都看多类似于对alloc的说明，就是开辟了一块内存空间，那具体是怎么调用？

- `NSObject.mm`

```Objective-C
+ (id)alloc {
    return _objc_rootAlloc(self);
}

// Base class implementation of +alloc. cls is not nil.
// Calls [cls allocWithZone:nil].
id
_objc_rootAlloc(Class cls)
{
    return callAlloc(cls, false/*checkNil*/, true/*allocWithZone*/);
}

// Call [cls alloc] or [cls allocWithZone:nil], with appropriate
// shortcutting optimizations.
static ALWAYS_INLINE id
callAlloc(Class cls, bool checkNil, bool allocWithZone=false)
{
#if __OBJC2__
    if (slowpath(checkNil && !cls)) return nil;
    if (fastpath(!cls->ISA()->hasCustomAWZ())) {
        return _objc_rootAllocWithZone(cls, nil);
    }
#endif

    // No shortcuts available.
    if (allocWithZone) {
        return ((id(*)(id, SEL, struct _NSZone *))objc_msgSend)(cls, @selector(allocWithZone:), nil);
    }
    return ((id(*)(id, SEL))objc_msgSend)(cls, @selector(alloc));
}
```

## 调用过程
  `alloc -> _objc_rootAlloc -> callAlloc`

在callAlloc中，有slowpath和fastpath两个判断的方法，slowpath和fastpath是封装的两个宏定义。

在objc-os文件中
```Objective-C
#define fastpath(x) (__builtin_expect(bool(x), 1))
#define slowpath(x) (__builtin_expect(bool(x), 0))
```

这个指令是gcc引入的，作用是允许程序员将最有可能执行的分支告诉编译器。这个指令的写法为：__builtin_expect(EXP, N)
意思是：EXP==N的概率很大

一般使用方法是将__builtin_expect指令封装为liekly和unlikely宏。

在llvm-DenseMap文件中，也可以看到封装的两个宏定义
```Objective-C
#define LLVM_UNLIKELY slowpath
#define LLVM_LIKELY fastpath
```

- fastpath(x)表示x值为真的可能性更大
- slowpath(x)表示x值为假的可能性更大

通过这种方式，编译器在编译过程中，会将可能性更大的代码紧跟着前面的代码，从而减少指令跳转带来的性能上的下降。

## 例

```Objective-C
int x, y;
if (fastpath(x > 0)) {
  y = 1
} else {
  y = -1
}
```

gcc编译的指令会预先读取y=1这条指令，一般这么写适合x值大于0的概率比较高的情况。如果x值<0的概率比较高，则使用slowpath(x)。这样编译器指令就会预先读取y = -1。
这样系统就可以在运行时减少重新取值了。

待续
