---
title: redis 初步探索
date: 2018-11-24 17:40:14
tags: tools
---
> 在本周我尝试了 redis 的一些操作，感觉还是很有意思的，所以写本文记下来


# 简介
Redis是一个开源的、基于内存的数据结构存储器，可以用作数据库、缓存和消息中间件。
Redis 是一款采取 CS 架构的单线程服务器，基于事件循环处理客户端请求。尽管 redis 是基于内存进行存储的，但是它可以通过快照的形式（包括全量和增量）来进行数据持久化。


# 使用
redis 的数据库结构非常简单，每个数据库采取一个数字来标识，默认数据库是 0.通过 select n 就可以切换数据库。

redis 提供了 5 种数据结构来进行数据存储，分别是 字符串，数组，set，hashMdp。

## 字符串
这是最常见的数据类型，
```
set users:leto '{"name": leto, "planet": dune, "likes": ["spice"]}' //写
get users:leto  //读
```
另外 redis 对字符串有一些常见的操作
```
> strlen users:leto     // 获取字符串长度 
(integer) 50    

> getrange users:leto 31 48     // 获取特定范围的字符
"\"likes\": [\"spice\"]"

> append users:leto " OVER 9000!!"  // 追加内容
(integer) 62
```

另外如果字符串是整型也会很有意思
```
> incr stats:page:about     // 加一
(integer) 1
> incr stats:page:about
(integer) 2

> incrby ratings:video:12333 5  // 增加指定大小
(integer) 5
> incrby ratings:video:12333 3
(integer) 8
```

## 哈希结构
和普通字符串而言是多了一层数据:
```
hset users:goku powerlevel 9000     // 设置hashset
hget users:goku powerlevel          // 获取hashset
hmset users:goku race saiyan age 737    // 一次设置多个
hmget users:goku race powerlevel        // 一次获取多个
hgetall users:goku      // 获取全部值
hkeys users:goku        // 获取全部key
hdel users:goku age     // 删除key
```

## 列表
列表结构可以让你，为指定的 key 保存和处理数组形式的 value 。
```
lpush newusers goku     // push 内容
ltrim newusers 0 49     // 保留最新的五十个
```

## 集合结构
```
sadd friends:leto ghanima paul chani jessica    // 向 set 内添加内容
sismember friends:leto jessica      // 检验是否在内部
```

## 有序集合
和普通集合的唯一区别在于存在权重
```
zadd friends:duncan 70 ghanima 95 paul 95 chani 75 jessica 1 vladimir // 添加内容时加权重
zcount friends:duncan 90 100    // 统计 90-100 间的个数
zrank friends:duncan chani      // 由低到高进行排序
zrevrank friends:duncan chani   // 由高到低进行排序
```

## 其它操作
### 原子操作
Redis 所有的命令都是原子性的，包括那些一次可以执行多项操作的命令也一样。此外，在使用多命令的时候，Redis 支持事务，例如
```
    redis.watch('powerlevel')
    current = redis.get('powerlevel')
    redis.multi()
    redis.set('powerlevel', current + 1)
    redis.exec()
```

### 超时
```
expire pages:about 30               // 超时时间 30 s
expireat pages:about 1356933600     // 设置 unix 时间戳
ttl pages:about                     // 还剩多久
persist pages:about                 // 删除过期数据
setex pages:about 30 '<h1>about us</h1>....'    // 在放入时就设置过期时间
```

### 发布订阅
同一个发布事件可以被多个窗口监听，一个窗口也可以监听多个事件
```
subscribe warnings          // a 窗口发布消息
publish warnings "it's over 9000!"  // b 窗口接受消息
```

### Monitor and Slow Log
这是两个非常好的调试工具,但不应该被用于生产环境中

### 排序
sort 是Redis 最强力命令之一。它允许你对列表，集合，有序集合中的值进行排序 (有序集是依照权重排序的，而不是集合中的成员)

```
    rpush users:leto:guesses 5 9 10 2 4 10 19 2
    sort users:leto:guesses         // 由低到高（默认）
    sort friends:ghanima limit 0 3 desc alpha   //  分页、逆序，按照字典序

```

排序可以更复杂
```
    hset bug:12339 severity 3
    hset bug:12339 priority 1
    hset bug:12339 details '{"id": 12339, ....}'

    sort watch:leto by bug:*->priority get bug:*->details   // 根据优先级排序，获取细节
```

另外，store 可以缓存排序结果

## 扫描
keys 命令不应该被用于生产环境，scan 则是另一回事了
一次调用 scan 无需返回所有匹配结果。没什么奇怪的，就是一个被分页的结果;但是, scan 返回的结果条数不定，它不能被精确的控制。你可以用 count 选项，默认是 10，不过它完全有可能拿到比指定的 count 更多或更少的结果。
也就是说，一个空结果集并不意味着没有其他的结果存在。只有一个 0 游标，才意味着没有更多的结果。

```
    scan 0 match bugs:* count 2
    > 1) "3"
    > 2) 1) "bugs:125"
    scan 3 match bugs:* count 2
    > 1) "0"
    > 2) 1) "bugs:124"
    >    2) "bugs:123"
```

*这有两点需要牢记。首先，scan 可以多次返回相同的 key 。你需要自己处理(比如说保存一个已有值集合)。其次，scan 只保证在迭代的整个持续过程中的存在值会被返回。如果在迭代中有值被添加或者被删除，新值可能被返回，旧值可能被忽略。再强调一次，这就是 scan 所谓的无状态; 它不会对存在值做快照(就像你在许多数据库中看到的那样，提供了强一致性保证)，仅仅是遍历同一块内存空间，不管空间有没有发生变更。*


# 集群
简单集群和负载均衡其实是很危险的——当某一台机器出现问题时上面的数据会全部丢失，因此，我们应该采取主从库的结构，而 master-slave 模式可以有以下两个优势：
- 数据高可用：Master负责接收客户端的写入请求，将数据写到Master后，同步给Slave，实现数据备份。一旦Master挂了，可以将Slave提拔为Master；
- 提高查询效率：一旦Master发现自己忙不过来了，可以把一些查询请求，转发给Slave去处理，也就是Master负责读写或者只负责写，Slave负责读；尽管这样可能导致读到一些过期的数据，不过还算是可以接受的折中。

配置主从服务可以通过修改配置文件的 slaveof 标签或者用 slaveof 命令来实现。不过目前 redis 还不支持自动将 slave 提升为 master,因此依旧需要用心跳监控和脚本来手动处理

# 备份
备份 Redis 只需要简单的将 Redis 的快照拷贝到你想要的地方(S3, FTP, ...)。默认的，Redis 把它的快照保存在名为 dump.rdb 的文件中。随时，你都可以 scp, ftp 或者 cp (或别的什么) 操作这个文件。

在主服务上禁用快照或者禁用增量文件(append-only file (aof))，转而让从服务去维护，这种做法并不少见。这有助于降低主服务器上的负载，并且允许在从服务上使用更积极的备份操作，而不会影响整个系统的响应速度。


# 参考资料
- [redis 简介](https://github.com/ilivebox/the-little-redis-book/blob/master/zh-cn/redis.md)
- [Redis简明教程](https://zhuanlan.zhihu.com/p/37055648)
