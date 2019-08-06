---
title: redux 机制探索
date: 2019-04-16 19:53:23
tags: front-end
---
# 背景
近日研究了一下 redux 的文档和源码，深感自己原来根本没有理解对 redux 的用法和原理。在花了三天时间把这一块搞明白之后，写下这篇博客作为记录。

# 什么是 redux
我一直以为 redux 是一个 react 的专属的全局状态管理工具，但实际上， redux 是一个单向数据流的观察者模式，完全可以用在后端或者其它前端框架里使用。
redux 将整个数据流分为以下几个部分:

## store
store 是 redux 的核心部分，它负责全局状态的管理（包括创建、更新以及对外暴露）、观察者的管理（包括注册、删除及触发对应的更新）。
而全局状态 state 对外是只读的（这也是为什么能保证单向数据流的原因），理论上讲，只要我们有全局 state 的数据，就获得一个 web 状态的复刻。

在 createStore 实现中比较有意思的点包括:
* 通过闭包维持一个变量，保证了不可能在 reducer 中再次触发 action
* 对 action 的类型判断（原生的 redux 只允许有对象类型的 action）
* 有一点很搞笑 —— 每次 dispatch 触发会使得所有的 listener 都触发，并没有针对的部分

## action
action 通常而言是对象（当使用 redux-thunk 的时候可以是特定类型的函数）, 这个对象会包含 type 和 data 两部分， 标明了类型和数据。
触发 action 是唯一变动 state 的方式, 这也是维持单向数据流的方式

## reducer
reducer 可以说是 redux 的核心部分，每一个 reducer 都是一个纯函数（最好是，否则时间旅行就是空谈）。只要我们知道前一个状态和触发的 action，那么必然会算出同一个结果。
reducer 的纯函数理论也让 redux 的测试变得非常的容易。

## combineReducer
redux 要求全局只有一个状态，但是这会让我们的开发有很大的麻烦，比如数据更新时的嵌套问题，也不方便 reducer 的测试。
而 combinReducer 提供了一个解决方法，它可以把多个 reducer 合并成同一个（返回一个新的 reducer 函数），状态也是如此。
不过它的更新机制有点搞笑——它会把所有子 reducer 和传入的 action 都算一遍，由于 reducer 默认返回原状态，所以也没什么大不了的，如果能更新，则必然会更新。

## Middleware
中间件机制类似于 express 中的中间件，指的是当 dispatch 触发前后进行一些操作，本质上中间件机制就是通过对原 dispatch 进行层层的嵌套调用。


# redux 中间件
## redux-thunk
redux-thunk 的本意是用来解决异步问题的，但是实际上可以用来使一个特定类型的函数作为 action。
该函数会接受 (dispatch, getState) 作为参数，而在函数中应当触发新的 action，否则父 action 会终结
通过这种写法可以把请求数据、更新数据汇成一体，只需要写一个这种类型的函数即可

## 手写
中间件只是一个三阶函数， next 可以理解为该中间件内部的中间件们包裹的 dispatch，下面是我手写的一个尝试性的中间件，大致就是这样
```
export function createHelloworld () {
    return ({dispatch, getState}) => next => action => {
        console.log('hello world!!!!!', action);
        debugger;
        next(action);

    }
}

const helloWorld = createHelloworld();

export {
    helloWorld
}
```

# 进阶
## state 组织
比较常见的玩法主要是以下几点：
* 将数据像数据库一样组织，准确的说是 selector 机制，这样在一些容易修改数据的web app 中会可靠很多
* 可以通过 reselect 可以完成对数据很好的缓存（一般写在 mapState 中）, 这样可以避免重复的计算


# 参考资料
* [redux官网](https://redux.js.org)
* [Idiomatic Redux: Using Reselect Selectors for Encapsulation and Performance](https://blog.isquaredsoftware.com/2017/12/idiomatic-redux-using-reselect-selectors/)