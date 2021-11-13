# State

[Apple Developer]([State](https://developer.apple.com/cn/documentation/swiftui/managing-user-interface-state/))

# Xcode

```Swift
/// A property wrapper type that can read and write a value managed by SwiftUI.
///
/// SwiftUI manages the storage of any property you declare as a state. When the
/// state value changes, the view invalidates its appearance and recomputes the
/// body. Use the state as the single source of truth for a given view.
///
/// A ``State`` instance isn't the value itself; it's a means of reading and
/// writing the value. To access a state's underlying value, use its variable
/// name, which returns the ``State/wrappedValue`` property value.
///
/// You should only access a state property from inside the view's body, or from
/// methods called by it. For this reason, declare your state properties as
/// private, to prevent clients of your view from accessing them. It is safe to
/// mutate state properties from any thread.
///
/// To pass a state property to another view in the view hierarchy, use the
/// variable name with the `$` prefix operator. This retrieves a binding of the
/// state property from its ``State/projectedValue`` property. For example, in
/// the following code example `PlayerView` passes its state property
/// `isPlaying` to `PlayButton` using `$isPlaying`:
///
///     struct PlayerView: View {
///         var episode: Episode
///         @State private var isPlaying: Bool = false
///
///         var body: some View {
///             VStack {
///                 Text(episode.title)
///                 Text(episode.showTitle)
///                 PlayButton(isPlaying: $isPlaying)
///             }
///         }
///     }
```

一种属性包装器类型，可以读取和写入 SwiftUI 管理的值。

SwiftUI会管理任何使用@State声明的属性，当State值发生变化，SwiftUI会使关联的UI失效并重新计算。

State实例不是值本身，只是一种阅读方式，State提供了读取和写入Value值。要访问状态的基础值，请使用变量name，他返回State/wrappedValue的属性值

- State/wrappedValue 与 @propertyWrapper是息息相关的

要将状态属性传递给视图层次结构中的另一个视图，请使用带有$前缀运算符的变量名。这将检索状态属性来自于State/projectedValue属性

- 如果没有带有$就在SwiftUI中使用的话，编译器会报错，并且提示'insert $'

```Swift
@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
@frozen @propertyWrapper public struct State<Value> : DynamicProperty {

    /// Creates the state with an initial wrapped value.
    ///
    /// You don't call this initializer directly. Instead, declare a property
    /// with the `@State` attribute, and provide an initial value; for example,
    /// `@State private var isPlaying: Bool = false`.
    ///
    /// - Parameter wrappedValue: An initial wrappedValue for a state.
    public init(wrappedValue value: Value)

    /// Creates the state with an initial value.
    ///
    /// - Parameter value: An initial value of the state.
    public init(initialValue value: Value)

    /// The underlying value referenced by the state variable.
    ///
    /// This property provides primary access to the value's data. However, you
    /// don't access `wrappedValue` directly. Instead, you use the property
    /// variable created with the `@State` attribute. For example, in the
    /// following code example the button's actions toggles the value of
    /// `showingProfile`, which toggles `wrappedValue`:
    ///
    ///     @State private var showingProfile = false
    ///
    ///     var profileButton: some View {
    ///         Button(action: { self.showingProfile.toggle() }) {
    ///             Image(systemName: "person.crop.circle")
    ///                 .imageScale(.large)
    ///                 .accessibilityLabel(Text("User Profile"))
    ///                 .padding()
    ///         }
    ///     }
    ///
    /// When a mutable binding value changes, the new value is immediately
    /// available. However, updates to a view displaying the value happens
    /// asynchronously, so the view may not show the change immediately.
    public var wrappedValue: Value { get nonmutating set }

    /// A binding to the state value.
    ///
    /// Use the projected value to pass a binding value down a view hierarchy.
    /// To get the `projectedValue`, prefix the property variable with `$`. For
    /// example, in the following code example `PlayerView` projects a binding
    /// of the state property `isPlaying` to the `PlayButton` view using
    /// `$isPlaying`.
    ///
    ///     struct PlayerView: View {
    ///         var episode: Episode
    ///         @State private var isPlaying: Bool = false
    ///
    ///         var body: some View {
    ///             VStack {
    ///                 Text(episode.title)
    ///                 Text(episode.showTitle)
    ///                 PlayButton(isPlaying: $isPlaying)
    ///             }
    ///         }
    ///     }
    public var projectedValue: Binding<Value> { get }
}
```

State是一个结构体，实现了DynamicProperty，有两个注解

- @frozen
- @propertyWrapper

DynamicProperty是一个Protocol，也就是State内部，实现了update的方法

通过网上一些文章可以了解到，State是通过struct值变化通知SwiftUI，使当前视图失效并且重新计算。

- Why @state only works with structs

  - [知乎](https://zhuanlan.zhihu.com/p/111033422)
  - [hackingwithswift](https://www.hackingwithswift.com/books/ios-swiftui/why-state-only-works-with-structs)

文章中解释了为什么@state只可以在struct中使用，大概意思是，结构体本身是不可变的，我们无法修改的它的属性，Swift需要销毁并且重建整个结构以完成属性的的改动。类并不需要，因为哪怕类本身是一个常量，Swift仍然可以直接修改它的变量属性。

- 重点就在于 class是引用类型，struct是值类型。

当然如果我们要实现数据共享，可以使用ObservedObject，文章中也有讲解到。

还有几个疑问点：
- $符号是如何实现的，我能否自己实现一个类似于State的注解
- State底层是如何进行销毁后又重新创建再通知到SwiftUI的

翻阅了很多文档都没看到相关的资料，在Apple上也没有相关的文档，若能翻阅到相关文档后续再更新吧。
