# React-Native底层原理

- 参考
  - https://zhuanlan.zhihu.com/p/41920417

## React-Native框架构成

React-Native框架内部已经提供了很多内置组件，如图所示。
如View、Text等基本组件，用于一些功能布局的Button、Picker等，用于列表展示的各种List组件和对应和对应iOS平台还有Android平台的特定组件、API等。
同时也提供了供编写与原生太平交互的接口。
![](https://pic2.zhimg.com/80/v2-2bf84ece7deae66b28515b442240cd7d_720w.jpg)

## React-Native工作原理

![](https://pic4.zhimg.com/80/v2-990aa3a1c34a8e1b956baaa00b4ca9db_720w.jpg)
React-Native渲染在React框架中，JSX源码通过React框架最终渲染到了浏览器的真实DOM中，而在React-Native框架中，JSX源码通过React-Native框架编译后，通过对应平台的Bridge实现了与原生框架的通信。如果我们在程序中调用了React-Native提供的API，那么React-Native框架通过Bridge调用原生框架中的方法。因为React-Native的底层为React框架，所以如果是UI层的变更，那么就映射为虚拟DOM后进行的diff算法，diff算法计算出变动后的JSON映射文件，最终由Native层将此JSON文件映射渲染到原生APP的页面元素上，最终实现了在项目中只需控制state以及props的变更来引起iOS与Android平台的UI变更。编写React-Native最终会打包成一个main.bundle.js文件供App加载，此文件可以在App设备本地，也可以存放于服务器上供App更新（热更新）。

## React-Native与原生平台进行通信

在与原生框架通信中，React-Native采用了JavaScriptCore作为JS VM，中间的JSON文件与Bridge进行通信。而如果在使用Chrome浏览器进行调试时，那么所有的JavaScript代码都将运行在Chrome的V8引擎中，与原生代码通过WebSocket进行通信。
![](https://pic4.zhimg.com/80/v2-60eb566b812a49fa945e802abe8dd453_720w.jpg)
* JS VM是一个面向JavaScript开发领域的基础框架。

# 组件间通信

React-Native开发最基本的元素就是组件，React-Native与React一样，也会涉及到组件之间的通信，用于数据在组件间传递。

## 父子组件的通信

在React-Native中，父组件向子组件传递值，可以通过props的形式。在下例中，父组件通过调用子组件并赋值子组件的name为React，子组件通过this.props.name获取父组件传递的name的字符串值React。
```JavaScript
/**
* 章节: 03-04
* 父子组件通信，在父组件中调用子组件
* FilePath: /03-04/parent-2-child.js
* @Parry
*/

<ChildComponent name='React'/>

/**
* 章节: 03-04
* 子组件实现，通过 props 获取父页面传递的值
* FilePath: /03-04/parent-2-child.js
* @Parry
*/

class ChildComponent extends Component {
    render() {
        return (
          <Text>Hello {this.props.name}!</Text>
        );
    }
}
```

## 子父组件的通信

在开发过程中，不仅有父子之间的通信，有时还会有子组件像父组件通信传值的需求，比如当子组件某个值变更后，需要通知到父组件做相应的变更与响应，那么就会需要父子之间的通信。示例代码如下，在父组件的定义中，在调用子组件时，同样向子组件传递了一个参数，不过这个参数是一个函数，此函数用于接收后续子组件像父组件传递过来的数据，与之间父组件向子组件传递数据不太一样。
```JavaScript
/**
* 章节: 03-04
* 子父组件通信，父组件的实现
* FilePath: /03-04/child-2-parent.js
* @Parry
*/
import React, {Component} from 'react';
import ChildComponent from './ChildComponent'

class App extends Component {
  constructor(props) {
    super(props)
    this.state = {
      name: 'React'
    }
  }

  //传递到子组件的参数，不过参数是一个函数。
  handleChangeName(nickName) {
    this.setState({name: nickName})
  }

  render() {
    return (
      <div>
        <p>父组件的 name：{this.state.name}</p>
        <ChildComponent
          onChange={(val) => {
          this.handleChangeName(val)
        }}/>
      </div>
    );
  }
}

export default App;
```
下面为子组件的定义，子组件在页面中定义了一个按钮，点击此按钮后，调用自身的一个函数handleChange，修改了自身state中的值name为nickName定义的值Parry，那么此子组件的页面上的字符串将由之前的Hello React变为Hello Parry！，同时使用了this.props.changeName，也就是父组件调用时传递过来的函数，向父组件传递了nickName的值Parry。父组件在接收到了子组件的调用后，调用了父组件自身的函数handleChangeName修改了自身state中的name为Parry，也就是子组件传递过来的Parry，所以同时，父组件页面上的值也由Hello React变成了Hello Parry。
```JavaScript
/**
* 章节: 03-04
* 子父组件通信，子组件的实现
* FilePath: /03-04/child-2-parent.js
* @Parry
*/

import React, {Component} from 'react'

export default class ChildComponent extends Component {
  constructor(props) {
    super(props)

    this.state = {
      name: 'React'
    }
  }

  handleChange() {
    const nickName = 'Parry';
    this.setState({name: nickName})
    //调用父组件传递过来的函数参数，传递值到父组件去。
    this
      .props
      .changeName(nickName)
  }

  render() {
    const {name} = this.state;
    return (
      <div>
        <p>Hello {name}!</p>
        <Button
          onPress={this
          .handleChange
          .bind(this)}
          title="修改一下 name 为 Parry"/>
      </div>
    )
  }
}
```

## 多级组件之间的通信

如果组件之间的父子层级非常多，需要进行组件之间的传递，这时候当然可以通过上面介绍的方法一层层的传递，但是当这种组件层级很深的时候，这样的传递不是一个太好的方法。解决方法是首先在设计App时，需要注意不能让组件之间的层级关系太深，一是为了避免组件之间的通信的冗长，还有一个原因是太深的嵌套逻辑，用户体验上也不太好，可以想象一下用户从底层一层层操作返回到最顶层时的体验。第二就是可以使用如Context对象或global等方式进行多级组件之间的通信，但是这种方式不太推荐。

## 无直接关系之间的通信

前面提到的都是有层级关系的组件通信方式，而如果组件之间没有层级的话，可以通过如AsyncStorage或JSON文件等方式进行无直接关系组件间的通信。当然，还可以使用EventEmitter/EventTarget/EventDispatch继承或实现接口的方式、Signals模式或Publish/Subscribe的广播形式，都可以达到无直接关系组件间的通信。这些组件间的通信方式使得组件之间的数据可以传递起来。
