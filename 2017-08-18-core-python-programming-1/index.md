# Python核心编程笔记1 基础知识

![](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/080_S.jpg)
这本书翻译稀烂, 然而我还是忍着看完了第一部分

<!--more-->

## Python基础
### 基本风格指南
1. 跨行代码可以使用反斜杠`\`
2. 变量赋值通过**引用传递**
3. python不支持`++`, `--`, 请使用`+=`和`-=`
4. 多元赋值: 实际上是元组赋值: `a, b = 1, 2`等价于`(a, b) = (1, 2)`. 在python中, 没有括号的分组**默认是元组**, 而不是列表
5. 下划线标识符:
 - `_xxx`: 此类属性不会被`from module import *`导入. 另外python不提倡使用此语句导入包, 容易造成名空间混乱
 - `__xxx__`: 类的特殊方法
 - `_xxx`: 类的私有变量名, 在子类中建议使用, 可以避免和父类冲突
6. `__name__`: 如果module被导入, 变量值为module名, 如果被执行, 为`__main__`

### 内存管理
python的垃圾收集器实际上是引用计数器+循环辣鸡收集器

## Python对象
### Built-in 类型
1. 内建类型略
2. 切片对象的复制问题
3. 对象值的比较可以用`id()`, 类似于c中的取地址操作`&`, `a is b`可以比较a与b的地址
4. python不支持函数或者方法重载
5. **类型工厂函数/内建函数**: 看上去是函数, 实际上是生成了该类型的一个实例:
 - `int`, `float`, `complex`, `str`, `list`, `turple`, `dict`, `type`, `bool`, `set`, `super`等
6. **可变类型**: 列表和字典. **不可变类型**: 数字, 字符串, 元组

### 数字
1. 数字的隐式类型转换和c++相似
2. 几个取整的常用工厂函数:
 - `round(3.141, 2) = 3.15`: 按四舍五入取整
 - `int(3.14) = 3`: 直接截取整数部分, 返回int
 - `floor(-0.2) = -1.0`: 取最接近但小于原数的整数, 返回float

## 序列
### 字符串

1. 切片索引的一个小技巧: 用`None`作为索引值, 可以使用一个变量作为索引从第一个遍历到最后一个, 即`s[:None] = s`

```python
s = 'abcde'
for i in [None] + range(-1, -len(s), -1):
    print s[:i]  
# 输出
abcde
abcd
abc
ab
a
```

2. 字符串比较是按ASCII值比较的, 因此大写字母一定比小写字母小. 即`cmp('Z', 'a' ) == -1`
3. `string`库的常用方法和属性:
 - 替换`'abcde'.replace('abc', 'xyz') = 'xyzde'`
 - 分割`'a/bxxx/c'.split('/') = ['a', 'bxxx', 'c']`
 - 删除头尾空格`'  abc  '.strip() = 'abc'`

### 列表
1. 删除元素时, 知道元素值用`del(element)`, 不知道元素只知道序号用`remove(index)`
2. 列表的`sort()`和`reversed()`都是inplace操作, 返回None, 而`sorted()`和`reversed()`是工厂函数, 可以作为值返回

### 元组

1. 单元素元组的坑: 只有一个元素的元组需要加逗号, 否则容易和工厂函数混淆:

```python
>>> tuple('abc')
('a', 'b', 'c')
>>> ('abc', )
('abc',)
```

2. 所有多对象, 用逗号分隔的, 没有明确符号定义的集合默认为元组

### 浅拷贝, 深拷贝和可变/不可变对象的例子

```python
>>> person = ['name', ['savings', 100.00]]
>>> hubby = person[:] # 切片复制
>>> wifey = list(person) # 工厂函数复制
>>> [id(x) for x in person, hubby, wifey] # 查看地址
[1, 2, 3] # 为了方便使用了虚构的地址, 可见三个变量不同

>>> hubby[0] = 'joe'
>>> wifey[0] = 'jane'
>>> hubby, wifey
(['joe', ['savings', 100.0]], ['jane', ['savings', 100.0]])

>>> hubby[1][1] = 50
>>> hubby, wifey
(['joe', ['savings', 50.0]], ['jane', ['savings', 50.0]])
```
 - hubby, wifey这两个对象是新的, 但他们的内容不一定
 - 序列类型对象的默认拷贝是浅拷贝, 可以通过**切片**, **工厂函数**, **`copy()`**实现
 - **不可变对象**(例子中是字符串)的拷贝, 实际上是删除变量引用, 创建新的对象, 然后转移引用到新变量
 - **可变对象**(例子中是列表)的拷贝只是复制了引用
 - 想要深拷贝请使用`copy.deepcopy()`
 - 对原子元素组成的元组进行深拷贝无效

## 映像和集合类型
### 字典
1. 只需要字典名就可以遍历字典key, 但key是无序的
2. 检查key是否存在使用`in`和`not in`, `has_key()`即将被弃用
3. key必须是可hash的, 也就是不可变对象. 可以使用`hash()`检验
4. 字典的常用方法有
 - `iteritems()`返回一个迭代器, 而不是数组
 - `values()`返回一个包含字典所有值的列表

### 集合
无序, 只能用工厂函数构造

## 条件和循环
### 循环语句
1. 三元组表达式: `A if EXP else B`, 等价于`exp? A:B`
2. `else`可以放在循环之后, 只要循环是正常结束的, 会执行`else`中的语句

### 迭代器

1. 定义: Iterator类是实现了`__iter__()`和`next()`的类.[例子出处](https://stackoverflow.com/questions/2776829/difference-between-pythons-generators-and-iterators)
```python
class Yes(collections.Iterator):
    def __init__(self, stop):
        self.x = 0
        self.stop = stop
    def __iter__(self):
        return self
    def next(self):
        if self.x < self.stop:
            self.x += 1
            return 'yes'
        else:
            # Iterators must raise when done, else considered broken
            raise StopIteration 
    __next__ = next # Python 3 compatibility
```
在`for`循环中迭代该类, 会依次调用迭代器的`next()`方法, 直到`StopIteration`抛出, `for`循环会捕捉处理.
2. 将其他序列转化成迭代器可以直接使用`iter()`工厂函数. 如果传递两个参数给`iter()`, 他会重复调用`func`, 直到迭代器的下个值等于`sentinel`

### 列表解析式
只说一点, 列表解析式可以套双层循环:
```python
>>> [(x+1, y+1) for x in range(3) for y in range(5) if x != y]
>>> [(1, 2), (1, 3), (1, 4), (1, 5), (2, 1), (2, 3), (2, 4), (2, 5), (3, 1), (3, 2), (3, 4), (3, 5)]
```
scala也有类似的语法, 也许是从函数式语言中引进的

### 生成器表达式
和生成器类似, 有关生成器的内容见后文, 这里只给出和列表解析式的区别:

```python
# 列表解析式
[expr for iter_var in iterable if cond_expr] 
# 生成器表达式
(expr for iter_var in iterable if cond_expr)
```

可见只是将方括号换成了圆括号. 对生成器表达式迭代会引发和生成器类似的效果, 即延迟计算(lazy evaluation), 也可以理解为懒惰的列表解析, 是对内存更友好的结构.

## 文件和输入输出
### 内建函数open()和file()
常见访问模式
 - `r`: 只读
 - `w`: 写入,若文件存在则首先清空
 - `a`: 写入, 若文件存在则追加
 - `w+`/`a+`: 读写模式,参考前两个模式
### 内建方法
1. `readline()`: 读取打开文件的第一行, **不会删除行结束符**
2. `readlines()`: 读取所有行并把他们作为一个字符串列表返回
3. `write()`: 写入, **但不会自动加入换行符**
4. `writelines()`: 将列表写入, **但不会自动加入换行符**
5. 文件对象可以直接作为文件迭代器:

```python
 f = open(`abc.txt`, 'r')
 for eachLine in f:
     print eachLine
 f.close()
```
## 错误和异常
有些行为, 比如用户退出或者系统退出并不应该被捕获, 需要提前处理:
```python
try:
    pass
except (KeyboardInterrupt, SystemExit):
    # 系统或者用户退出
    raise
except Exception:
    # 处理真正的错误
```
2.5版本之后, KeyboardInterrupt和SystemExit被从Exception里移出和Exception平级, 因此不必单独处理这两种情况:
```python
try:
    pass
except Exception, e:
    # 处理错误
```
最后, Exception的父类是BaseException

### 上下文管理
```python
with open('abc.txt', 'r') as f:
    pass
```

### 断言
一个用try语句捕捉断言产生的错误的例子:
```python
try:
    assert 1 == 0, "That's reall silly"
except AssertionError, args:
    print ('%s: %s' % args.__class__.__name__, args)
```
