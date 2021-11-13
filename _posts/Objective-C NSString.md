# NSString

这几天一直和同事讨论NSString相关的一些话题，便想写一篇文记录一下。

在NSString中

```Objective-C
@"Hello" == @"Hello"
```

是等于true的

又或者说

```Objective-C
NSString *str = @"Hello"
NSString *str2 = @"Hello"
```

str和str2也是相等的。

原因就因为str和str2使用的都是一个常量字符串@"Hello"
