---
title: go debug 方法
date: 2019-08-02 22:28:58
tags: go
---
# 使用 delve 来进行调试
* break: 断点，可以使用 a.b 进行方法断点，或者 a.go:line 进行行断点
* continue(c): 执行到下一个断点
* list(l): 展示代码
* next(n): 执行到下一行
* step(s): 进入下一层
* stepout: 离开这一层
* print: 打印值

# 参考资料
* [delve 文档](https://github.com/go-delve/delve/tree/master/Documentation/cli)