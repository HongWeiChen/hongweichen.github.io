# ViewBuilder

ViewBuilder 视图构建

Apple官方有一个解释：

- Allowing those closure to provide multiple child views

允许闭包中提供多个子视图

如果不使用@ViewBuilder的时候只可以传递一个View在闭包里，使用@ViewBuilder可以传递多个View到闭包中。

- 示例

```Swift
struct ExampleView<Content: View>: View {

    @ViewBuilder var content: () -> Content

    var body: some View {
        content()
    }

}

@main
struct MainApp: App {

  var body: some Scene {
      WindowGroup {
          ExampleView {
              Text("Hello world")
          }
      }
  }
}
```

![](http://m.qpic.cn/psc?/V53N7OG413cvz52OliN03DvKaZ42EFAJ/TmEUgtj9EK6.7V8ajmQrEJA**rCz*ljiFBpgg0HY5fmkaAskrF3r*WS.8KGer4kPGNr7*TkQz3.XCxo2DfuNdKVwux1JWBJ.sGcgpHeDk1U!/b&bo=PgGyAgAAAAADF70!&rf=viewer_4)

- 在UIKit中UIView是一个Class，而在SwiftUI中，View是一个Protocol

# 再看看ViewBuilder

```Swift
@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
@resultBuilder public struct ViewBuilder {

    /// Builds an empty view from a block containing no statements.
    public static func buildBlock() -> EmptyView

    /// Passes a single view written as a child view through unmodified.
    ///
    /// An example of a single view written as a child view is
    /// `{ Text("Hello") }`.
    public static func buildBlock<Content>(_ content: Content) -> Content where Content : View
}
```
可以看出ViewBuilder是一个struct

# ResultBuilder

- @resultBuilder是一个自定义的类型（之前是叫@_functionBuilder），添加了相关语法，用以自然地、声明的方式来创建嵌套的数据，比如链表和树。使用resultBuilder的代码可以包含原始的Swift语法，比如if和for以方便掌控条件数据或者重复的数据。

- 一个类、结构体使用@resultBuilder注解时，必须至少包含一个buildBlock方法，并且这个方法是static静态的。这个方法可以接收0个或者多个参数，在函数内部就确定了参数组成形式。

- 示例

```Swift
@resultBuilder
struct StringBuilder {
    static func buildBlock(_ s1: String, _ s2: String) -> String {
            s1 + s2
    }
}

func welcome(@StringBuilder showText: () -> String) -> String {
    showText()
}

@main
struct MainApp: App {

    var body: some Scene {
        WindowGroup {
            ExampleView {
                Text(welcome {
                    "hello "
                    "world"
                })
            }
        }
    }
}
```

![](http://m.qpic.cn/psc?/V53N7OG413cvz52OliN03DvKaZ42EFAJ/TmEUgtj9EK6.7V8ajmQrEJA**rCz*ljiFBpgg0HY5fmkaAskrF3r*WS.8KGer4kPGNr7*TkQz3.XCxo2DfuNdKVwux1JWBJ.sGcgpHeDk1U!/b&bo=PgGyAgAAAAADF70!&rf=viewer_4)

# 再回到ViewBuilder

在ViewBuilder中，Extension中含有许多方法

```Swift
/// Provides support for “if” statements in multi-statement closures,
/// producing an optional view that is visible only when the condition
/// evaluates to `true`.
public static func buildIf<Content>(_ content: Content?) -> Content? where Content : View

/// Provides support for "if" statements in multi-statement closures,
/// producing conditional content for the "then" branch.
public static func buildEither<TrueContent, FalseContent>(first: TrueContent) -> _ConditionalContent<TrueContent, FalseContent> where TrueContent : View, FalseContent : View

/// Provides support for "if-else" statements in multi-statement closures,
/// producing conditional content for the "else" branch.
public static func buildEither<TrueContent, FalseContent>(second: FalseContent) -> _ConditionalContent<TrueContent, FalseContent> where TrueContent : View, FalseContent : View

/// Provides support for "if" statements with `#available()` clauses in
/// multi-statement closures, producing conditional content for the "then"
/// branch, i.e. the conditionally-available branch.
public static func buildLimitedAvailability<Content>(_ content: Content) -> AnyView where Content : View
```

比较值得关心的是

```Swift
public static func buildBlock<C0, C1>(_ c0: C0, _ c1: C1) -> TupleView<(C0, C1)> where C0 : View, C1 : View
```

C0...Cn一系列方法

ViewBuilder结构体有11个名为buildBlock的函数，分别从0-10个View的类型参数，因此再SwiftUI中一个接收@ViewBuilder类型参数的视图容器最多能接收10个子视图，如果不能满足需求可以通过拆分来增加子视图的个数

- A custom parameter attribute that constructs views from closures.

这是Apple官方文档对ViewBuilder的定义，简单来说ViewBuilder就是一个包含多个视图的闭包。
