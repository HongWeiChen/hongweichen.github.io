# Copy on write

## 写时复制

在Swift中，结构体有着相当重要的地位，在Swift标准库中，大约有90%的公开类型都是结构体，包括我们常用的Array、Dictionary、String。结构体相比类，一个最重要的特征就是它的值类型，而类似引用类型。值类型是通过复制来赋值的，而不是引用同一个地址，这就不存在数据共享的问题，能防止意外的数据改变，并且它是线程安全的。

举一个简单的例子，在objc中，数据是类，是引用类型，在Swift中，是结构体，是值类型。

```Objective-C
NSMutableArray *array = @["zhangsan", "lisi"].mutableCopy;
NSMutableArray *array2 = array;
[array addObject:@"wangwu"];
```

array和array2都变成了["zhangsan", "list", "wangwu"]也就是array的改变会导致array2的改变，因为他们都是引用类型，并且都引用了相同的一块内存地址。

而在Swift中，就不会存在这样的问题
```Swift
var array = ["zhangsan", "lisi"]
array2 = array
array.append("wangwu")
```
在Swift中，array是["zhangsan", "lisi", "wangwu"]，而array2还是["zhangsan", "lisi"]，这就是结构体和类的最大区别。


那么，是不是每次将struct赋值给到其他变量或者传递函数时都会发生复制呢。答案是否定的，在Swift中Array、Dictionary、String这些类型中，尽管他们都是值类型，但在Swift的具体实现中做了优化，可避免不必要的复制。在《The Swift Programming Language(Swift2.2)》一书中的“Classes and Structures”一章末尾写到：`The description above refers to the “copying” of strings, arrays, and dictionaries. The behavior you see in your code will always be as if a copy took place. However, Swift only performs an actual copy behind the scenes when it is absolutely necessary to do so. Swift manages all value copying to ensure optimal performance, and you should not avoid assignment to try to preempt this optimization.`

在Swift中采用的优化方式叫写时复制，简单的说就是，只有当一个结构体发生了写入的动作时才会有复制的行为。具体的做法就是，在结构体内部用一个引用类型来存储实际的数据，在不进行写入操作的普通传递中，都是将内部的reference引用计数+1，在进行写入操作时，对内部的reference做一次copy操作用来存储新的数据，防止和之前的reference产生意外的数据共享。

在Swift中有一个方法：`isUniquelyRefrencedNonObjC`(Swift 2.2)，在Swift3中这个函数变成了：`isKnownUniquelyReferenced`，他能检查一个类的实例是不是唯一的引用，如果是，我们就不需要对结构体的实例进行复制，如果不是，就说明被不同的结构体共享，这时对他进行修改就需要进行复制。

这个函数只对Swift对象有效，如果要用在Objective-C对象上，可以对Objective对象用Swift进行一次封装。

下面是《Advanced Swift》书中的一个写时复制技术的代码实例

```Swift
class Box<A> {
  var unbox: A
  init(_ value: A) {
    unbox = value
  }
}

struct GaussianBlur {
  private var boxedFilter: Box<CIFilter> = {
    var filter = CIFilter(name: "CIGaussianBlur", withInputParameters: [:])!
    filter.setDefaults()
    return Box(filter)
  }()

  fileprivate var filter: CIFilter {
    get { return boxedFilter.unbox }
    set { boxedFilter = Box(newValue) }
  }

  private var filterForWriting: CIFilter {
    mutating get {
      if !isKnownUniquelyReferenced(&boxedFilter) {
        filter = filter.copy() as! CIFilter
      }
      return filter
    }
  }

  var inputImage: CIImage {
    get { return filter.value(forKey: kCIInputImageKey) as! CIImage }
    set { filterForWriting.setValue(newValue, forKey: kCIInputImageKey) }
  }

  var radius: Double {
    get { return filter.value(forKey: kCIInputRadiusKey) as! Double }
    set { filterForWriting.setValue(newValue, forKey: kCIInputRadiusKey) }
  }
}

extension GaussianBlur {

  var outputImage: CIImage? {
    return filter.outputImage
  }

}
```
