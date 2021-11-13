# Currying 柯里化

前阵子在翻看Redux源码的时候，突然看到createStore中有个代码是
```
function createStore(preloadState, state, enhancer) {
  ...忽略一大段代码
  enhancer(createStore)(preloadState, state)
  ...忽略一大段代码
}
```
对JavaScript的了解层度不深，瞬间有点疑惑这行代码是干嘛的。

问了下Web前端开发的同事，才知道这个叫柯里化（一开始听成颗粒化了），一听就是高大上了。

后面又了解到，其实这个就是函数式编程。

其实在Objective-C的Masonry框架中，基本运用的就是函数式编程，就是柯里化，只是换了个名称。

# 柯里化
```
维基百科：是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术
```

示例：
```JavaScript
function sum(x, y) {
  return x + y
}

let ret = sum(3, 4)
console.log(ret)
// 7

// 转换成Currying ->

function sum(x) {
  return function(y) {
    return x + y
  }
}

let ret = sum(3)(4)
console.log(ret)
// 7
```

# Currying的作用

- [参数复用](#参数复用)
- [提前确认](#提前确认)
- [延迟运行](#延迟运行)

## 参数复用
这种写法也经常出现在面试题中，起码我就见过。
```JavaScript
function sum() {
  return function(x) {
    return x + 4
  }
}

let sumFunc = sum()
// 复用sumFunc
sumFunc(1) // 5
sumFunc(2) // 6
sumFunc(3) // 7
```

## 提前确认
提前确认使用的方法
```JavaScript
function calculate(isAdd) {
  if (isAdd == true) {
    return function(x, y) {
      return x + y
    }
  } else {
    return function(x, y) {
      return x - y
    }
  }
}

var add = calculate(true)
add(3, 4) // 7
var sub = calculate(false)
sub(4, 3) // 1
```

## 延迟运行
这个就没有好示例的，上面的方法基本已经做了延迟运行的事情。也就是可以先声明好对应的方法，以及编写好对应的逻辑，有需要的时候再去使用。

# 性能方面

- 存取arguments对象通常要比存取命名参数要慢一点
- 一些老版本的浏览器在arguments.length上的实现上是相当慢的
- 使用fn.call(...)与fn.bind(...)通常比fn(...)要慢一些
- 创建大量嵌套作用域和闭包函数会带来花销，无论是内存还是速度上
