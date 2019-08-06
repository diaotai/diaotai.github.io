---
title: js 模块机制探索
date: 2019-04-16 15:03:07
tags: JavaScript
---
# 背景
近日出现了在写代码的时候碰到了一次循环引用的问题，最有趣的是竟然没有出问题，这引起了我的兴趣，所以决定来写一下自己的探索过程。
下文将分为 CommonJS 和 ES6 两种模块机制来写

# CommonJS
CommonJS模块的重要特性是加载时执行，即脚本代码在require的时候，就会全部执行。其中，获取的文件是已缓存的形式存在的，第二次获取不再执行，引用文件中的改变也不会体现出来。
而当出现循环加载时、CommonJS的做法是就只输出已经执行的部分，还未执行的部分不会输出。
举个例子:
```
// a.js
exports.done = false;
console.log('a开始执行');
var b = require('./b.js');
console.log('在 a.js 之中，b.done = %j', b.done);
exports.done = true;
console.log('a.js 执行完毕');
```

```
// b.js
exports.done = false;
console.log('b 开始执行');
var a = require('./a.js');
console.log('在 b.js 之中，a.done = %j', a.done);
exports.done = true;
console.log('b.js 执行完毕');
```

```
// main.js
var a = require('./a');
var b = require('./b');
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done);
```

实际的输出结果为:
```
a开始执行
b 开始执行
在 b.js 之中，a.done = false
b.js 执行完毕
在 a.js 之中，b.done = true
a.js 执行完毕
在 main.js 之中, a.done=true, b.done=true
```

1. 因为在 main.js 先引用 a.js， 因此 a.js 率先执行，打印出第一行输出。
2. 在 a.js 第三行，引用了 b.js, 因此 b.js 开始执行，打印出输出第二行。
3. 而在 b.js 第三行它再次引用了 a.js, 由于已经存在 a.js，所以只会获得 a.js 中已执行部分（且由于已经执行过了，所以不会再次执行）,体现在 b.js 第四行，就是 a.done 依旧是 false（因为已执行部分还不是 true）.
4. 随后就有了 输出的 第三行和第四行。再之后就是 a.js 执行完成。
5. 在 main.js 中，第二行引用 b.js 由于 b.js 已经被导入了，所以不会再引起执行。最后的输出则体现了 a.js 和 b.js 两个文件都完成的被导入了。

综上，在 CommonJS 中文件的导入是以缓存的形式存在的，且在第一次导入时即执行。

# ES6
ES6模块的运行机制与CommonJS不一样，它遇到模块加载命令import时，不会去执行模块，而是只生成一个引用。等到真的需要用到时，再到模块里面去取值（如果有什么打印，就会在第一次完成打印,以后不会打印）。
因此，ES6模块是动态引用，不存在缓存值的问题，而且模块里面的变量，绑定其所在的模块。可以理解为不同的模块只是放在同一个文件里不同对象里的值或函数，如果同一个文件下的两个函数循环执行不会爆栈，那么放在两个模块也不会。

```
// c.js
import { testD } from './d';

let count = 0;
function testC() {
    console.log('count from c', count++);
    if (count < 10) {
        testD();
    }
}

console.log('console from c');

export {
    testC
}
```

```
// d.js
import { testC } from './c';

let count = 0;
function testD() {
    console.log('count from d', count++);
    if (count < 10) {
        testC();
    }
}

console.log('console from d');

export {
    testD
}

```

```
// main.js
import { testC } from './c';
import { testD } from './d';

testC();
testD();
```

输出结果为：
```
console from d
console from c
count from c 0
count from d 0
count from c 1
count from d 1
count from c 2
count from d 2
count from c 3
count from d 3
count from c 4
count from d 4
count from c 5
count from d 5
count from c 6
count from d 6
count from c 7
count from d 7
count from c 8
count from d 8
count from c 9
count from d 9
```

分析以下以上输出结果：
* 开局执行 testC, c.js 文件中代码被执行，而由于 testC 中有函数 testD，因此正式启动导入 d.js 的过程。
* 所以就有了第一行输出，完成导入后 c.js 继续执行，完成了第二行输出。 
* 然后就是执行 testC 了，除了最后一行输出以外剩下的都是递归执行的结果。最后一行是执行 testD 的结果。

# 对比
CommonJS 被认为是缓存，而 ES6 被认为是指针，个人更喜欢 ES6 一些，只要注意就算递归引用也不会出错。

# 参考资料
[JavaScript 模块的循环加载](http://www.ruanyifeng.com/blog/2015/11/circular-dependency.html)