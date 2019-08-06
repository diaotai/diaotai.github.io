---
title: pudb 使用指南
date: 2019-08-03 22:48:46
tags: tool
---
> 简而言之，pudb 就是 python 的一个调试工具，在特殊环境下会非常的好用

* 使用: import pudb;pu.db; 这时候会在下一行打一个断点
* ctrl-x: 切换到命令行，命令行里可以直接输出结果
* b: 打断点或取消断点
* s: step in，但是面对语言自带函数是进不去的
* r: return, 会直接到函数结尾的地方
* c: continue, 会执行到下一个断点
* n: next, 下一行
* q: quit, 退出 pudb

