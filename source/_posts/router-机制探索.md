---
title: router 机制探索
date: 2019-04-24 17:31:02
tags: front-end
---
# 背景
路由这个概念最先是后端出现的。在以前用模板引擎开发页面时，经常会看到一些 .html 或者 .aspx 结尾的路径，这些就是传统的后端路由, 路由的作用就是根据不同的路径来返回不同的资源（这个资源很多时候是页面）。
传统的路由分为以下四步:
1. 浏览器发出请求
2. 服务器监听到80端口（或443）有请求过来，并解析url路径
3. 根据服务器的路由配置，返回相应信息（可以是 html 字串，也可以是 json 数据，图片等）
4. 浏览器根据数据包的`Content-Type`来决定如何解析数据

但是后来，前端路由出现了.
就算在我接触 SPA 的两年后的今天，我都清楚的记得自己第一次接触前端路由时的震撼和赞叹。将所有的路由跳转都交由前端来负责，确实是里程碑意义的突破，而且，这里面对我来说黑科技的成分实在太重了，所以我决定花一些时间，将里面的东西拆开看一看，了解一下内在的机制。

# 原理
所谓前端路由，其实只做了两件事：
1. 保证当路径发生变化的时候不会再次向服务器请求数据，也就是避免页面刷新
2. 在路径变化的时候渲染对应的组件

甚至第二点都不必要保证——只需要将回调函数和路径对应起来，在路径变为某值时执行对应的回调函数即可。
如果要想实现 history 的操作，可以直接使用 html5 的 history，也可以在内部建立一个数组（这样一刷新肯定就没了，但是如果在非 DOM 环境下会很有用）

前端路由常见的实现思路有两种 —— hash 模式和 history 模式。

## hash 模式
hash 模式指的是利用 location.hash 来指定路径，由于 location.hash 的改变不会引起页面的跳转，所以可以很好的完成前端路由的第一点。此外，该模式不需要服务器的配合——向服务器请求的路径只会是根路径，后面的 hash 值是不影响资源的，这样的话如果用户刷新的话，也可以直接跳转到之前的页面。

要注意的是， hash 值的改变是会被直接记录在 history 对象中的，而当我们手动进行前进、后退或手动输入地址的时候只会触发 hashChange 事件(对当前地址的手动输入是不会触发的)，刷新时会触发 load 事件，所以我们只需要监控这两点就可以了。

```
    window.addEventListener('load', this.refresh, false);
    window.addEventListener('hashchange', this.refresh, false);
```

![hash 模式流程图](https://pic1.zhimg.com/80/v2-73fd628a36c848bce7771c395b9ce6a2_hd.jpg)

总结以下 hash 模式的优点和缺点：
优点：
* 兼容性良好
* 不需要服务器的配合

缺点：
* 会有一个丑陋的 # 标志
* 可能会影响滚动的代码编写

## history 模式
在 html5 中， history.pushState 和 history.replaceState，通过这两个 API 可以改变 url 地址且不会发送请求——这样带来了新的可能。

就像在 hash 模式下我们需要关注 hashchange 事件，在 history 我们需要关注 popstate 事件。

window.onpopstate 是 popstate 事件在 window 对象上的事件处理程序.
每当处于激活状态的历史记录条目发生变化时, popstate 事件就会在对应 window 对象上触发. 如果当前处于激活状态的历史记录条目是由 history.pushState() 方法创建,或者由 history.replaceState() 方法修改过的, 则 popstate 事件对象的 state 属性包含了这个历史记录条目的 state 对象的一个拷贝.
调用 history.pushState() 或者 history.replaceState() 不会触发 popstate 事件. popstate 事件只会在浏览器某些行为下触发, 比如点击后退、前进按钮(或者在 JavaScript 中调用 history.back()、history.forward()、history.go() 方法).

由于我们的 pushState 和 replaceState 方法是不会触发 popstate 事件的，所以我们需要在路由改变的时候手动触发一下。当然，两个事件还是一定要关注的。

当用户手动刷新时，浏览器会向服务器请求资源，而且那个路径在服务器上应当是没有资源的，这种时候服务器应该返回跟路径所对应的资源，然后由前端路由跳到指定页面。

![history 模式流程图](https://pic2.zhimg.com/80/v2-ba9732885bb188f8cde94c24ce5b6757_hd.jpg)

优点：
* 没有 # 符号，更为流畅

缺点：
* 兼容性问题
* 需要服务器有所配合

# 总结
综上，前端路由本质上就是在不刷新页面的情况下改变 url，并对应的有所行动。至于 react-router 或 vue-router 这些，就是要显示在当前路径上的组件了,还是很有意思的。

# 参考资料
* [ReactRouter-V4 构建之道与源码分析](https://zhuanlan.zhihu.com/p/25696969)
* [前端路由简介](https://www.zhihu.com/question/53064386)
* [面试官：你了解前端路由吗](https://juejin.im/post/5ac61da66fb9a028c71eae1b)
* [history与hash路由的区别](https://juejin.im/post/5b31a4f76fb9a00e90018cee)
* [简易前端路由实现](https://github.com/SME-FE/sme-router)
* [react-router 源码解析](https://github.com/fi3ework/blog/issues/21)