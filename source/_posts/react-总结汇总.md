---
title: react 总结汇总
date: 2019-04-30 15:11:41
tags: front-end
---
# react 生命周期
![react 生命周期图](https://upload-images.jianshu.io/upload_images/4345168-fd3988bc7dcfee69.png)

我们可以将 react 生命周期分为两种：当组件在挂载或卸载时和当组件产生实质性更新时。

DOM 真正被添加到 HTML 中的生命周期方法时 componentDidMount 和 componentDidUpdate 中，因此，在这两个方法中去取得 DOM 元素才是有意义的。而取得 DOM 元素的方法有两种，其一是 findDOMNode（只对已挂载组件有作用），其二则是 render 方法中的回调。

在 componentWillReceiveProps 中调用 setState 是很正常的操作，但是经测试，就算不调用，props 的改变也会导致整个子组件的更新。

# react 事件机制
* react 并不会把时间处理函数直接绑定到真实的节点上，而是把所有时间绑定到结构的最外层，使用一个统一的事件监听器。
* 在某些情况下我们可能需要使用原生的事件，通常是在 componentDidMount 中挂载， 在 componentWillUnmount 中清除。
* 最好不要将 react 事件和原生的事件共用，react 是无力阻止原生事件的冒泡的。而在原生事件里阻止冒泡，是可以阻止 react 事件的冒泡的。

# react 组件抽象
* react 的高阶组件思路有两种——属性代理和反向继承。前者是进行一次包裹、后者是继承然后调用目标组件的 render。
* 两种方法都可以很好的处理 state、props 或者进行渲染劫持。
* 不过生命周期顺序有不同：前者是 didMount->HOC did mount -> HOC will unmount -> unmount, 后者是 did mount -> HOC did mount -> will unmount -> HOC will unmout

# react 性能优化
* 尽量使用纯函数组件
* 使用 Immutable，该库使用 tire 进行设计，非常的高效和好用

# react 源码注意点
* react 对于 text、DOM 和 Component 有不同的处理
* 在处理差异时，基本就是先干掉没有的旧属性，然后加上或修改新属性（包括样式、监听器、属性、DOM 属性）
* 生命周期的实现是使用模版设计模式，且大量使用的 事务（可嵌套，前后有操作，中间才是任务）
* setState 使用事务来进行标记，在同步状态下会将几个更新放入队列中合并、而异步时不会进入事务
* diff 算法分为 tree diff 部分、component diff 和 element diff
* patch 部分即为将更新映射到 DOM 上

# MVC 和 MVVM
## MVC
![MVC 架构图](https://crxdoc-zh.appspot.com/static/images/mvc.png)
* Model 负责保存数据以及和后端交互数据或校验数据，Model 的改变会应用到 View 上
* View 是 Model 的可视化表示
* Controller 负责连接 View 和 Model，View 的操作会通过 Controller 应用到 Model 上

## MVVM
![MVVM 架构图](https://ask.qcloudimg.com/http-save/yehe-2192986/uptje5tcky.png?imageView2/2/w/1620)
* 区别就在于用 ViewModel 代替了 Controller，ViewModel 是双向绑定的

# redux
## 三大原则：
* 单一数据源
* 状态是只读的
* 状态修改由纯函数完成

## 其它 
* 中间件中使用 dispatch 的原因一般是因为希望不到达原生的分发，通常是在异步流里
* webpack.DefinePlugin 可以让我们定义任意字符串，并在代码中把字符串替换成该值（可以用来检测是否为环境变了）
* 将 router 和 redux 也是很重要的部分

## SSR
现在的渲染方式有以下几种——前端渲染、后端渲染和混合渲染。
### 前端渲染
SPA 最正常的渲染方式，也就是绝大多数人所使用的。当完成开发后打包就好。

### 后端渲染
纯粹的后端渲染其实很像模版了——都是路由对应后端的某个路径，当请求该路径的时候会把 jsx 当作类似于模版的东西，将获得的数据传入其中，然后在后端完成代码的执行，得到生成的 html 后再发送至浏览器。
其实可以将后端渲染理解为一个运行在服务器的简易版浏览器。

### 混合渲染
前后端渲染都有所涉及，比较常见的方式是只对主页进行服务端渲染，剩下的部分依旧使用前端渲染（比较正常的做法是对不同的路由组件分开进行打包和懒加载，可以很好的提升性能）。

另一种做法被称为 universal app 或者 isomophic app，这种做法就是对同一个路径前后端都进行支持，当用户以正常的点击进入时，走的是前端路由，而如果在该页面刷新的话，使用的就是后端的服务器端渲染。

要注意的是，当一个组件想要支持服务器端渲染时，需要在获取数据的那部分做一点兼容（若传入数据，则直接使用，否则用 ajax 请求来获取数据）

### 参考资料
* [React + Koa 实现服务端渲染(SSR)](https://juejin.im/post/5b0269c2518825428b3916f9#comment)
* [React + Koa 实现服务端渲染(SSR) Part II](https://juejin.im/post/5c73ad30f265da2db66de010)
* [服务端渲染与 Universal React App](https://juejin.im/post/5a0536346fb9a044fe45d33a)
* [react 服务端(ssr) 框架next.js开发个人网站分享](http://www.liuweibo.cn/p/206)

