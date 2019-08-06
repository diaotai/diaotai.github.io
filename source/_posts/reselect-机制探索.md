---
title: reselect 机制探索
date: 2019-04-25 15:10:52
tags: source-code
---
# 背景
在学习 redux 的过程中，我看到了一个优化功能非常明显的包 —— reselect.该包的核心作用就是根据输入值进行缓存，若本次输入与上一次输入相同，则不再进行重复计算。
本包最常用的地方在于 react-redux 的 mapStateToProps 里面，由于该函数每次 state 更新都会重复计算，无论是否涉及到它。在这种情况下，缓存就显得非常重要了。
虽然我个人对 reselect 的猜测是一个闭包，但是还是决定拆开看一看。

# 示例
```
import { createSelector } from 'reselect'

const shopItemsSelector = state => state.shop.items
const taxPercentSelector = state => state.shop.taxPercent

const subtotalSelector = createSelector(
  shopItemsSelector,
  items => items.reduce((acc, item) => acc + item.value, 0)
)

const taxSelector = createSelector(
  subtotalSelector,
  taxPercentSelector,
  (subtotal, taxPercent) => subtotal * (taxPercent / 100)
)

export const totalSelector = createSelector(
  subtotalSelector,
  taxSelector,
  (subtotal, tax) => ({ total: subtotal + tax })
)

let exampleState = {
  shop: {
    taxPercent: 8,
    items: [
      { name: 'apple', value: 1.20 },
      { name: 'orange', value: 0.95 },
    ]
  }
}

console.log(subtotalSelector(exampleState)) // 2.15
console.log(taxSelector(exampleState))      // 0.172
console.log(totalSelector(exampleState))    // { total: 2.322 }
```

# 源码
## defaultMemoize
defaultMemoize 是缓存的关键，可以很清楚地看到，就是通过闭包来缓存上一次的输入和结果，然后决定是否重新计算。
```
export function defaultMemoize(func, equalityCheck = defaultEqualityCheck) {
  let lastArgs = null
  let lastResult = null
  // we reference arguments instead of spreading them for performance reasons
  return function () {
    if (!areArgumentsShallowlyEqual(equalityCheck, lastArgs, arguments)) {
      // apply arguments instead of spreading for performance.
      lastResult = func.apply(null, arguments)
    }

    lastArgs = arguments
    return lastResult
  }
}
```

## createSelectorCreator
createSelectorCreator 则是另一个关键，可以看到，这里的缓存是进行了两层，第一层是 selector 的总输入——一般是全局的 state，第二层则是计算出来的子参数，当两层结果都有不同时，才会触发重新计算。
另外，判断是否相等的默认标准是严格相等。
```
export function createSelectorCreator(memoize, ...memoizeOptions) {
  return (...funcs) => {
    let recomputations = 0
    const resultFunc = funcs.pop()
    const dependencies = getDependencies(funcs)

    const memoizedResultFunc = memoize(
      function () {
        recomputations++
        // apply arguments instead of spreading for performance.
        return resultFunc.apply(null, arguments)
      },
      ...memoizeOptions
    )

    // If a selector is called with the exact same arguments we don't need to traverse our dependencies again.
    const selector = memoize(function () {
      const params = []
      const length = dependencies.length

      for (let i = 0; i < length; i++) {
        // apply arguments instead of spreading and mutate a local list of params for performance.
        params.push(dependencies[i].apply(null, arguments))
      }

      // apply arguments instead of spreading for performance.
      return memoizedResultFunc.apply(null, params)
    })

    selector.resultFunc = resultFunc
    selector.dependencies = dependencies
    selector.recomputations = () => recomputations
    selector.resetRecomputations = () => recomputations = 0
    return selector
  }
}
```

# 总结
其实东西很有限，但是我还是想记下来，避免以后想不起来。

# 参考资料
* [reselect 仓库地址](https://github.com/reduxjs/reselect)