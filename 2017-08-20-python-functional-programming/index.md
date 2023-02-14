# Python核心编程笔记2 函数式编程

![](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/gamersky_028small_056_201785187DEA.jpg)
不学好函数式编程, 总觉得人生有遗憾
<!--more-->

## 什么是函数
1.  python函数可以返回一个值或对象, 但返回一个容器对象的时候看起来像是返回了多个值: `return 'abc', 'def'`实际上是返回了一个有两个元素的元组而已
2. 函数有标准调用和关键字调用两种形式. 有关可变长度参数调用参考后文

## 创建函数
```pyhon
def foo():
    print('in foo()')
    bar()
def bar():
    print('in bar()')

>>> foo()
>>> in foo()
>>> in bar()
```
python和c++不太一样的地方: 不存在前向引用的问题. 在`foo()`中对`bar()`进行的调用出现在`bar()`之前, 但`foo()`本身不是在`bar()`声明之前被调用的, 在调用`foo()`时`bar()`已经存在了, 因此调用成功

## 函数属性(attribute)
1. 函数也拥有自己的名空间, 使用函数名+`.`标识
```python
def foo():
    'foo -- property created doc string'
def bar():
    pass
bar.__doc__ = 'Oops, forgot the doc string'
bar.version = 0.1 # 利用名空间赋值
>>> print(bar.version)
0.1
```

1. 内嵌函数

```python
def foo():
    def bar():
        print 'bar() called'
    print 'foo() called'
    bar()
>>> foo()
foo() called # 调用foo输出的
bar() called # foo中调用bar输出的
>>> bar()    # 直接调用bar出错
...          
NameError: name 'bar' is not defined
```
注意内嵌函数的整个函数体都在外部函数的名空间内, 因此如果没有任何对`bar()`的外部引用, 那么除了函数体内, 任何地方都不能对其进行调用

## 装饰器(decorator)
### 装饰器的产生背景
2.2版本时类方法被引入python, 实现很笨拙:
```python
class MyClass(object):
    def staticFoo(): # 静态方法省略了self关键字
        pass
    # 使用内建staticmethod将其转化为静态方法
    staticFoo = staticmethod(staticFoo)
```
使用装饰器之后, 可以用以下语法替换掉上面的
```python
class MyClass(object):
    @staticmethod
    def staticFoo():
        pass
```
甚至还可以将装饰器和函数调用一样堆叠起来:
```python
@deco2
@deco1
def func():
    pass
# 等价于
func = deco2(deco1(func))
```
### 带参数的装饰器
```python
@deco1(deco_arg)
@deco2
def func():
    pass
# 等价于
func = deco1(deco_arg)(deco2(func))
```
### 装饰器举例
装饰器及闭包接受一个函数, 返回一个修改后的函数对象, 将其重新赋予原来的标识符, 并永久失去对原始函数的访问.

1. 以下例子对函数增加了输出调用时间的功能
```python
from time import ctime

def time_func(func):
    def wrapper(*arg, **kw):  # 不知道func的参数
        print('[%s] %s called' %\
              (ctime(), func.__name__))
        return func(*arg, **kw)  # 传入相应参数并调用func
    return wrapper

@time_func
def foo():
    print("it's foo() running")

>>> foo()
[Fri Aug 18 16:49:11 2017] foo called
it's foo() running
```
1. 以下例子增加了传入参数并输出的功能
```python
from time import ctime, sleep

def time_log(text):
    def time_func(func):
        def wrapper(*arg, **kw):  # 不知道func的参数
            print('[%s] %s %s' % (ctime(), func.__name__, text))
            return func(*arg, **kw)  # 传入相应参数并调用func
        return wrapper
    return time_func

@time_log('called')
def foo():
    print("it's foo() running")

>>> foo()
[Fri Aug 18 17:05:08 2017] foo called
it's foo() running
```
这里体现了装饰器的一个好处: 装饰器内的函数在装饰器的名空间内, 因此可以直接调用传入的`text`而不需要写在参数列表里

## 可变长度参数
函数接受参数的顺序:
1. 必须参数
2. 默认参数
3. 可变长度参数: 用`*args`表示, 当然名字任意, 但星号必须
4. 关键字变量参数(字典): 用`**kw`表示, 规则同上

一个小例子:
```python
def foo(arg1, arg2=0.0, *args, **kw):
    'display regular args and all variable args'
    print(arg1, arg2)
    for _ in args:
        print('additional non-keyword arg: ', _)
    for _ in kw:
        print('additional non-keyword arg: %s: %s' % \
              (_, kw[_]))
>>> foo('arg1', 1.0, 2.0, 3.0, kw1=4, kw2=5, kw3=6)
('arg1', 1.0)
('additional non-keyword arg: ', 2.0)
('additional non-keyword arg: ', 3.0)
additional non-keyword arg: kw1: 4
additional non-keyword arg: kw3: 6
additional non-keyword arg: kw2: 5
```
注意字典是无序的

## 函数式编程(functional programming)
### lambda表达式
```
def add(x, y=2):
    return x + y
# 等价于
lambda x, y=2: x+y
```
### 内建函数
```python
# 将func作用于每个seq元素, 返回所有令func返回True的元素
filter(func, seq)
# 将func作用于每个seq元素ele, 返回func(ele)组成的列表
map(func, seq)
# 将含有两个参数的函数func作用于seq的前两个元素, 然后将返回值和seq的下一个元素配对, 再赋予func, 以此往复
reduce(func, seq)
```
例子就懒得举了...自行搜索

### 偏函数
偏函数(partial function) 将任意数量(顺序)参数的函数转化成另一个带剩余参数的函数对象. 下面是一个带关键字参数的FPA:
```python
from functools import partial

baseTwo = partial(int, base=2) # 这里int为工厂函数int()
baseTwo('10010') # 等价于int('10010', base=2)
>>> 18
```

### 变量作用域/闭包
参考[stackoverflow上的文章](https://stackoverflow.com/questions/4020419/why-arent-python-nested-functions-called-closures)

> A closure occurs when a function has access to a local variable from an enclosing scope that has finished its execution.

一个闭包的例子:
```python
def make_counter():
    i = 0
    def counter(): # counter() is a closure
        nonlocal i
        i += 1
        return i
    return counter

>>> c1 = make_counter()
>>> c2 = make_counter()
>>> print (c1(), c1(), c2(), c2())
>>> 1 2 1 2
```

1. 在调用`make_counter()`之后, 一个函数栈帧被创造, 其中`counter()`和`i`都在函数的名空间之内. 
2. 接着函数返回`counter()`. 然而因为`counter()`refer了`i`变量, 因而在`make_counter()`返回之后, `i`依然保持, 没有随着函数栈帧的返回而被销毁.
3. 这种携带了整个enclosing函数的名空间的函数就是一个闭包.

---

闭包的后期绑定(late binding):
```python
def foo():
    return [lambda x: i*x for i in range(4)]

>>> for func in foo():
>>> ...  print(func(2))
6
6
6
6
```
为何返回的不是`0 2 4 6`而是`6 6 6 6`? 原因是定义一个函数，函数内的变量并不是立刻就把值绑定了，而是等调用的时候再查找这个变量. 此谓后期绑定. 
因为调用`func(2)`的时候lambda表达式的名空间里只有`x`而没有`i`, 在`foo()`返回之后`i`的值是3, 因此全部返回4.
根据闭包的性质, 如果lambda表达式内部的名空间拥有`i`, 则可以返回`0 2 4 6`:
```python
def foo():
    return [lambda x, i=i: i*x for i in range(4)]
>>> for func in foo():
>>> ...  print(func(2))
0
2
4
6
```
此时在创建lambda匿名函数的时候就要获取默认参数的值，放到 lambda 的名空间中

[参考文章](https://www.zhihu.com/question/29483144)

### 生成器(Generator)

从语法上讲, 生成器是一个带`yield`语句的函数, 当生成器的`next()或__next__()`(python2或3)被调用时, 他会准确地从离开的地方继续.

以下为生成器的几个主要方法:

1. `generator.__next__()`

 > Starts the execution of a generator function or resumes it at the last executed yield expression. 

 > When a generator function is resumed with a __next__() method, the current yield expression always evaluates to None. 

 > The execution then continues to the next yield expression, where the generator is suspended again, and the value of the expression_list is returned to __next__()’s caller. 

 > If the generator exits without yielding another value, a StopIteration exception is raised.

 > This method is normally called implicitly, e.g. by a for loop, or by the built-in next() function.

2. `generator.send(value)`

 > Resumes the execution and “sends” a value into the generator function. The value argument becomes the result of the current yield expression. 

 > The send() method returns the next value yielded by the generator, or raises StopIteration if the generator exits without yielding another value. 
 
 > When send() is called to start the generator, it must be called with None as the argument, because there is no yield expression that could receive the value.

对于生成器, 需要注意以下几点:
1. 除了一开始, 对generator调用`next()`时, 当前`yield`语句返回`None`
2. 接着运行直到遇见下一次`yield`, 将该表达式的值作为函数返回值
3. `send()`改变的是恢复之后的`yield`语句的返回值, 返回的是generator生成的下一个值

两个例子比较:
```python
def counter(start=0):
    count = start
    while True:
        val = (yield count)
        if val is not None:
            count = val
        else:
            count += 1
>>> count = counter(5)
>>> count.next()
5
>>> count.next()
6
>>> count.send(9)
9
>>> count.next()
10
```
看一看调用`count.send(9)`之后发生了什么:
1. `val`返回了9, `if`语句成立, `count = 9`
2. 循环继续, 遇到`yield`并返回9

```python
def adder():
    x = 1
    while True:
        y = (yield x)
        x += y
>>> g = adder()
>>> g.next()
1
>>> g.send(3)
4
>>> g.send(10)
14
```
为什么此处返回4呢? 看一看发生了什么:
1. 调用`next()`之后, 函数返回1, 此时`x = 1`
2. 调用`send(3)`之后, `yield`语句返回3, 即`y = 3`
3. `x += 3`之后为4, 接下来返回4, 注意此时`x = 4`
4. 调用`send(10)`之后, `yield`语句返回10, 即`y = 10`
5. `x += 10`之后为14, 最后返回14

此处，生成器函数 `adder` 用 `yield` 表达式，将处理好的 `x` 发送给生成器的调用者；与此同时，生成器的调用者通过 `send` 函数，将外部信息作为生成器函数内部的 `yield` 表达式的值，保存在 `y` 当中，并参与后续的处理。

这一特性是使用 `yield` 在 Python 中使用协程的基础。

参考文章:
1. [官方文档](https://docs.python.org/3/reference/expressions.html?highlight=send#generator.send)
2. [Python 中的黑暗角落（一）：理解 yield 关键字](https://liam0205.me/2017/06/30/understanding-yield-in-python/)
