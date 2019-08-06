---
title: 'nginx初步探索（基本介绍）'
date: 2018-11-17 23:31:54
tags: tools
---
> nginx 这个东西我听说很久了，实际上用页用过几次，但是实际上我一直没搞明白它是一个什么东西，以及它到底有什么用处，另外到底是什么原理。在这个周末，我看了一些文字，自己做了一些实验，简单的记一下。

# nginx是什么？
nginx 是一个开源的服务器，主要是用于http服务器，反向代理以及邮件服务器。理论上它和Apache是一类东西。而它和 Apache 的区别在于 Apache 是多进程的，当一个请求到来的时候 Apache 都会启动一个进程或线程对其进行处理。这样当并发量上去的时候对CPU和内存的需求简直爆炸。。。而 Nginx 是异步、单进程、事件驱动和非阻塞的，非常适合处理高并发。
此外，nginx 可以提供静态资源服务、缓存服务、修改 HTTP 头或内容、负载均衡、流量控制和权限控制等功能，可以说非常的全面且可靠。
当 nginx 提供负载均衡时存在 内置策略 和 自定义策略，而内置策略又分为三种：轮询，ip Hash，加权轮询三种。

# nginx 基本原理介绍
## nginx 基本组成
nginx 从架构上讲由一个核心和多个模块组成，核心部分的工作是维持一个事件队列，在合适的时候调用worker处理事件，并在相应的时候调用对应模块。而功能模块负责的东西更多，包括网络内容的读写，对内容的处理，与上游服务器交互，应用选择器进行筛选（在返回给client之前）。

## nginx 基本架构

![nginx基础架构](http://www.aosabook.org/images/nginx/architecture.png)

而具体到实现思路上是由一个 master 进程 和 N 个 worker 组成的（另外还有一个缓存加载进程和一个缓存管理进程，其实完全可以把它们视为 worker 进程）。

master 进程负责的任务包括：
- 读取并校验配置文件
- 创建、绑定或删除 与worker 进行通信的socket
- 开始、终止或维持 根据配置文件所设定的多个 worker
- 在不中断服务的基础上重新加载配置文件
- 处理日志输出
- 编译 Perl 脚本

worker 负责的任务包括：
- 读取并处理来自客户端的请求
- 反向代理
- 对返回内容进行筛选

而每个 worker 都可以处理 N 个请求，而正如之前所说，Nginx 不会为每个请求都启动新的进程或线程，而是会用已有的 worker 进行处理。这样就不会存在创建、销毁进程和切换上下文所产生的消耗，这是 Nginx 高效的原因之一，也是为什么 Apache 的消耗会随着并发量的上涨线性上涨，而 Nginx 是非线性上涨。另外 worker 和 Nginx 之间的通信是通过共享内存来实现的。

每个请求都完全由一个 worker 进程来进行处理，这样做的好处包括：
- 节省锁带来的开销
- 独立进程、避免风险
- 一次请求无需进行上下文切换

最后是缓存加载器和缓存管理器，前者负责缓存的读取和书写，后者负责缓存过期时间管理以及内容校验。

# nginx 最大连接数
- worker_processes：worker 进程的个数
- worker_connections： 每个 worker 可以处理的请求的个数
- Nginx作为http服务器的时候：max_clients = worker_processes * worker_connections
- Nginx作为反向代理服务器的时候：max_clients = worker_processes * worker_connections/4 （/4原因：因为浏览器默认会开启2个连接到nginx server，而且nginx还会为每个连接使用fds（file descriptor）从连接池建立connection到upstream后端。）

# nginx 常用命令
- nginx 配置文件所在：  /usr/local/etc/nginx 或 /etc/nginx
- 启动: nginx
- 缓和终止(完成现有任务再停止): nginx -s quit
- 迅速终止（直接退出）:  nginx -s stop
- 重启服务（通常因为修改配置）: nginx -s reload
- 重新打开日志文件: nginx -s reopen
- 指定配置文件: nginx -c filename
- 校验配置文件（不执行）: nginx -t filename
- 显示 nginx 版本: nginx -v
- 显示 nginx 版本及配置参数: nginx -V

# 参考资料
[nginx简介](http://www.aosabook.org/en/nginx.html)
[nginx快速入门之基本原理篇](https://zhuanlan.zhihu.com/p/31196264)
