---
title: redux-undo 机制探索
date: 2019-04-25 16:41:49
tags: source-code
---
# 背景
在看到 redux 的示例的时候，看到了一个很有意思的项目 —— redux-undo。该项目的作用在于时间旅行，可以很好的进行撤销、重做等操作。我比较好奇它的实现，所以就看了一下代码。代码很简单，但是还是决定要记录一下，一面日后遗忘。

# 原理
redux-undo 本质上是一个 redux 的 enhancer，它用两个数组和一个 present 来记录之前的状态和当前的状态，当触发 undo、 redo、 jump 等操作时，就去调用相应的 state，然后直接将其返回。这样就可以跳转到当时的状态。若这个 action 类型是没有见过的，那么直接调用原 reducer。
以下是不完整的代码，大致是这个思路：

```
export default function undoable (reducer, rawConfig = {}) {

  let initialState = config.history
  return (state = initialState, action = {}, ...slices) => {
    debug.start(action, state)

    let history = state

    const skipReducer = (res) => config.neverSkipReducer
      ? {
        ...res,
        present: reducer(res.present, action, ...slices)
      } : res

    let res
    switch (action.type) {
      case undefined:
        return history

      case config.undoType:
        res = jump(history, -1)
        debug.log('perform undo')
        debug.end(res)
        return skipReducer(res)

      case config.redoType:
        res = jump(history, 1)
        debug.log('perform redo')
        debug.end(res)
        return skipReducer(res)

      case config.jumpToPastType:
        res = jumpToPast(history, action.index)
        debug.log(`perform jumpToPast to ${action.index}`)
        debug.end(res)
        return skipReducer(res)

      case config.jumpToFutureType:
        res = jumpToFuture(history, action.index)
        debug.log(`perform jumpToFuture to ${action.index}`)
        debug.end(res)
        return skipReducer(res)

      case config.jumpType:
        res = jump(history, action.index)
        debug.log(`perform jump to ${action.index}`)
        debug.end(res)
        return skipReducer(res)

      case actionTypeAmongClearHistoryType(action.type, config.clearHistoryType):
        res = createHistory(history.present, config.ignoreInitialState)
        debug.log('perform clearHistory')
        debug.end(res)
        return skipReducer(res)

      default:
        res = reducer(
          history.present,
          action,
          ...slices
        )
        return res
    

        
    }
  }
}

```

# 参考资料
* [redux-undo 仓库](https://github.com/omnidan/redux-undo)