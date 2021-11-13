# Redux原理
因为之前在公司开发React-Native项目中，为了解决数据同步相应的问题，使用了Redux框架，今天就来写一写Redux原理，加深一下自己对Redux的理解。

参考来自：https://www.jianshu.com/p/e984206553c2

# Redux设计理念
Redux是将整个应用状态存储到一个地方上称之为store，里面保存着一个状态树store tree，组件可以派发(dispatch)行为(action)给store，而不是直接通知其他组件，组件内部通过订阅store中的状态state来刷新自己的视图
![](https://upload-images.jianshu.io/upload_images/6548744-df461a22f59ef7da.png?imageMogr2/auto-orient/strip|imageView2/2/w/800)

# Redux三大原则
- 唯一数据源
- 保持只读状态
- 数据改变只能通过纯函数执行

## 唯一数据源
整个应用的state都被存储到一个状态树里面，并且这个状态树，只存在于唯一的store中。

## 保持只读性
state是只读的，唯一可以改变state的方式只能通过action，action是一个用于描述以发生时间的普通对象。

## 数据改变只能通过纯函数执行
使用纯函数来执行修改，为了描述action是如何改变state的，只需要编写reducer

# Redux概念解析

## Store
- store就是保存数据的地方，整个应用只能有一个store
- redux提供createStore来生成store
```JavaScript
import {createStore} from 'redux'
const store=createStore(fn);
```
## State
state就是store里面的数据，store里可以有多个state，Redux规定一个state对应一个View，只要View相同，state就相同，反过来也一样，可以通过store.getState()来获取state
```JavaScript
import {createStore} from 'redux'
const store=createStore(fn);
const state=store.getState()
```

## Action
state的改变会导致View的变化，但是在redux中不能直接操作state，也就是说不能使用this.setState来操作，用户只能操作到View。Redux提供了一个对象来告诉Store需要改变state。Action是一个对象其中type属性是必须的，表示Action名称，其他可以根据需求自由设置。
```JavaScript
const action={
  type:'ADD_TODO',
  payload:'redux原理'
}
```

## store.dispatch()

store.dispatch是唯一View发出Action的方法
```JavaScript
store.dispatch({
  type:'ADD_TODO',
  payload:'redux原理'
})
```
store接收一个action作为参数，将它发送给store通知store改变state。

# Reducer

Store收到Action后，必须给出新的state，这样View才会变化。这种state的计算过程就叫做Reducer。

reducer是一个纯函数，也就是说函数的返回结果必须由参数state和action决定，而且不能产生任何副作用也不能修改state和aciton对象。

```JavaScript
const reducer =(state,action)=>{
  switch(action.type){
    case ADD_TODO:
        return newstate;
    default return state
  }
}
```

这部分内容来自于参考文章。

在文章下方有对源码的解析，但是个人对redux源码一翻查看后，发现这份解析是偏老了，有兴趣的可以去看下。有空的话我也会去对redux新的源码进行解析，未完待续。
