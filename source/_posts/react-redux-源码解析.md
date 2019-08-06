---
title: react-redux 源码解析
date: 2019-04-22 17:46:11
tags: source-code
---
# 整体结构
本次解析的代码是 react-redux 的 v5.0 版本，并不是最新的，不过也很好了。

代码的核心部分是 Provider 和 connectAdvanced 部分，前者是包含全部组件的地方，后者则是传递搞自组件的地方。

# Provider
```
export default class Provider extends Component {
  getChildContext() {
    return { store: this.store }
  }

  constructor(props, context) {
    super(props, context)
    this.store = props.store
  }

  render() {
    return Children.only(this.props.children)
  }
}

if (process.env.NODE_ENV !== 'production') {
  Provider.prototype.componentWillReceiveProps = function (nextProps) {
    const { store } = this
    const { store: nextStore } = nextProps

    if (store !== nextStore) {
      warnAboutReceivingStore()
    }
  }
}
```
可以看到 Provider 基本没做什么事情，就是把 store 放进了 context 里面，以便内部的组件可以直接通过 context 拿到数据。另外比较有意思的地方在于 process.env，这个变量在前端本该是不存在的，但是 webpack 在打包时通过插件将当时的变量注入了进去，也就是说该值是与打包环境有关，而不是与执行环境有关。（可以参考 webpack.config.js）

## connnectAdvanced
本方法会传入一大堆参数，但是这些都属于无关紧要的细节，真正的重点在于它返回的高阶组件，在该组件中，会从 context 中取出 store，此外还会试图取出 parentSub,在最外面的该组件时，就没有这个东西，但是它会把自己放在 context 中，从而让自己内部的 connect 生成的组件可以拿到它。

而在组件的 constructor 中，会执行 initSelector 方法，该方法的作用就是根据 mapStateToProps 和 mapDispatchToProps 来算出新的结果，如果结果和之前不一样，就会将 selector 的 shouldComponentUpdate 字段置为 true。 而用来计算和判断新属性的方法为 run，该方法会在 Connect 组件的 componentDidMount 和 componentWillReceiveProps 以及 subscription.onStateChange 中执行。此外，selector.shouldComponentUpdate 属性会在每次 render 中置为 false。

组件的 constructor 中，还会执行另一个方法 initSubscription，在该方法中，会进行 subscription.onStateChange 的实现（模版设计模式），然后若 selector.shouldComponentUpdate 为 false，则执行子订阅的通知，若为 true， 则更新自己的 state —— 这样会触发整体的更新。

在 Subscription 的实现中，会分为存在 parentSub（Connect 内部中的 Connect）和不存在 parentSub 两种情况。前者会在其父 Subscription 上实现订阅，后者则直接使用 store 的订阅。这样，可以保证子组件的 onStateChange 必然在父组件的之后调用。


# 总结
代码的核心思路就是这样了，没什么太了不起的地方，但是整体思路还是很巧妙的（个人觉得最巧妙的地方在于优化逻辑以及模版模式上）。
