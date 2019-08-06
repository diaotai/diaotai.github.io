---
title: python 学习笔记(基础)
date: 2019-07-13 16:00:22
tags: python, web
---
> 本笔记是我第 n 次重新上手 python 的笔记，该笔记并不全面，但是记录了很多关于 python 的某些的需要注意的点。只要注意到这些点，对我来说就可以迅速重新上手这门语言。

# 变量
* python 中变量不需要声明
* python 并没有常量的概念，虽然惯例上使用大写字母作为常量名，但是这只是惯例上的约束，实际上依旧可以改变
* 整数相除必然除出浮点数，除非用 // 地板除
* int 数字等于对应的浮点数
* 只要 x 上非零数字、非空字符串、非空 list 等，就判断为 True，否则为 False

# 数据结构
## list
* 不可越界，否则报错
* 可以存放不同的类型
* insert(index, val) 方法用于插入, pop(index?) 可以用来删除指定位置的值

## turple
* 形式类似于 (1, 3, 4),其内容是不可变的，剩下基本等同于 list

## dic
* 可以用 in 来判断某个 key 是否存在于对象
* get 方法可以用来安全的获取值，并可以在第二个参数设置默认值
* pop 可以用来删除特定 key

## set
* 添加可以使用 add 方法
* 删除的方法为 remove 而不是 pop

# 函数
* python 的函数可以返回多个值
* python 的默认参数一定要指向不变的对象
* 参数星号代表多参数，参数会以 list 的形式插入
* 两个星号则代表以命名参数的行事形成一个 dic
* 注意，python 同样存在闭包问题，因此要考虑变量在延迟执行时的情况

# 迭代器及函数式编程
* 能被 for 使用的对象就是 iterable 对象
* 能调用 next 的才是 Iterator 迭代器，可以理解为一个超长的数据流
* map, resuce, filter 的第一个参数是函数，第二个才是迭代器
* sorted 的第一个参数是 list，第二个是 key=function，第三个是布尔值 reverse
* functools.partial 可以以默认值的方式从右向左固定一些值

# 装饰器
```
def log(func):
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper
```
* func 即为被装饰器所包装的函数，如果需要给装饰器添加参数，则需要再包一层，参数放在最外边

# 模块
* python 使用包和文件作为模块的机制，包下必须要有__init__.py,否则会被认为是普通的目录
* python 没有限制导出的策略，因此习惯上正常变量名随意导出、__xxx__属于有特殊用途的情况，而_xxx属于私有变量，不应该被使用

# 类
```
class Name(object)
```
* 类名通常大写，括号内则是继承的类名
* \_\_init\_\_代表构造函数，第一个参数为 self 实例本身，后面则是类的参数
* 类内的 __xxx  在类外不可访问
* python 在类型上属于鸭子类型语言，十分类似于 go
* isinstance 可以判断子类及本身，而 type 则可以判断 class 类型
* 不加 self 即为类属性，实例属性会覆盖类属性，找不到实例属性就会去找类属性(直接在类中书写普通变量即可)
* \_\_slots\_\_属性可以限制允许挂载到对象上的属性的
* @property 用来将方法变成了 getter, @xxx.settter 把方法变成了赋值的 setter
* 结合 hasattr, getattr, setattr 可以很好的处理对象的属性
* \_\_str\_\_代表返回给用户的字符串，\_\_repr\_\_代表返回给开发者的字符串
* 要想实现一个迭代器，则必须实现\_\_iter\_\_方法返回一个迭代对象，并实现一个\_\_next\_\_方法不断的调用，直到碰到 StopIteration Error 
* \_\_getItem\_\_可以帮助实现一个类似于 list 的结构
* \_\_getattr\_\_用于处理没有定义的属性
* \_\_call\_\_让一个对象可以直接作为方法来被调用

# 测试
* 测试分为单元测试和文档测试
* class TestDict(unittest.TestCase) 可以用来测试
* setUp() 和 tearDown() 可以在测试开始前和结束后执行

# Collections
* namedtuple 可以实现一个类似于类的小玩意
* dqeue 用于实现队列和栈，存在 append, pop, appendleft, popleft 方法
* OrderedDict 会按插入的顺序进行排布
* ChainMap 可以合并多个 dic，最后以从前向后的顺序查找值
* struct 提供了二进制到类型的转换
* @contextmanager 可以提供上下文，让事物在开始前结束后做一些什么东西，可用于迭代器， yield 代表了返回的值


