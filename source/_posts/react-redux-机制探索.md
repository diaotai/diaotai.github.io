---
title: react-redux 机制探索
date: 2019-04-18 17:46:06
tags: front-end
---
# 背景
react-redux 本身的作用很小，就是为了让 redux 配合 react。其中，暴露出来的接口只有两个—— Provider 组件和 Connect 函数，这两个东西的作用就是将 store 内的数据传入对应组件，传入触发 action 的方法以及在 store 更新时进行更新，要注意的是，react-redux 对于 react 做了一些性能上的优化。

# API 简介
Connect 的两个参数:
## mapStateToProps
```
function mapStateToProps(state, ownProps?)
```
第二个参数是可选的，指的是那些从 Connect 组件上传下来的 prop，没有必要最好别用。
该函数的返回值应该是一个对象，其 key 为 this.props.xxx 的名称， value 则是具体值

### 使用指南
* 重塑数据形式（可以改变数据的 key，可以考虑将多个分支的数据合在一起）
* 使用 selector 来选择数据，本身就是一种很好的避免重复计算的方式（此外， 如果返回值的每一个子对象都没有变化的话，是不会重新渲染的）
* 运行的逻辑应当尽可能少，因为只要有 dispatch 导致 store 发生变化, 就会触发所有 Connect 包裹组件的重新计算（不管最后是否重新渲染）
* 应当是同步且不包含副作用，因为这个运算必然会进行很多次，所以任何副作用都可能导致奇怪的错误
* 该方法使用严格相等(===)来判断子对象是否相等，因此，要注意避免返回 map，concat，... 等方式返回的数据（这样会因为每次引用都不一样导致重新渲染），可以通过 selector 来解决
* 可以通过将数据运算放在 render 或 reducer 里来减少不必要的运算
* 该方法在 store 整体无变化的时候不会被执行（因为 redux 里面本身有是否有变化的判断）

## mapDispatchToProps
### 空传
在该值为 null 的时候，组件内部会有一个 dispatch 的属性。而当该值不为空的时候，就不会把 dispatch 传到组件中

### 函数形式
一共有 dispatch,ownProps (optional) 两个参数，返回值应当是一个对象，每个 key 都是函数名

### 对象形式
key 为函数名， value 为 actionCreator 


## Provider
Provider 是将 store 传入子组件用的，具体的原理就是靠 react 的 context 机制，Provider 会默认创建一个 Context，我们也可以使用自定义的 Context。如果使用自定义的 Context，那么每个 Connect 需要手动传一下。另外是可以存在多个 Context。

# 注意
* 每个 Connect 包裹的组件都是订阅了 store 的

# 参考资料
* [redux 官方文档](https://react-redux.js.org/using-react-redux/accessing-store)
* [redux 历史和实现](https://blog.isquaredsoftware.com/2018/11/react-redux-history-implementation/)