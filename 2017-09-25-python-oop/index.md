# Python核心编程笔记3 面向对象编程


python有很好的OOP特性, 自由度也非常大

<!--more-->

![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/64495434_p0.jpg)

## 命名规则
1. 小驼峰命名法(camel case), 适用于变量名: `varFirst`, `verSecond`
2. 大驼峰命名法, 适用于类名: `PersonFirst`, `PersonSecond`
2. 下划线命名法, 适用于函数/方法名: `var_first`, `var_second`

## 类属性(class attribute)
### 类的数据属性
相当于static修饰的变量
```
class C(object):
    foo = 100
>>> print(C.foo)
>>> 100
```
### 方法与绑定: 静态方法和类方法
没有实例, 方法是不能被调用的. 

但有两个特殊例子: 类方法(classmethod)和静态方法(staticmethod)
有关如何使用这两个方法, 参考[stackoverflow上的这篇文章](https://stackoverflow.com/questions/12179271/), 在这里总结一下:

1. 类方法可以被子类继承, 静态方法不可以.
2. 由于python不支持重载, 类方法的一大用处是作为工厂函数, 但静态方法就不可以, 因为如果是派生类, 静态方法会构造出基类, 而类方法可以构造出派生类.
3. 类方法和实例方法一样, 需要将类作为第一个隐式参数传入方法:

```python
@classmethod
def foo(cls, data):
    return cls(data)
```

此处类作为了第一个参数, 它由解释器传给方法, 类不需要特别地命名, 多数人使用`cls`作为变量名字

> @classmethod means: when this method is called, we pass the class as the first argument instead of the instance of that class (as we normally do with methods). This means you can use the class and its properties inside that method rather than a particular instance.

---

> @staticmethod means: when this method is called, we don't pass an instance of the class to it (as we normally do with methods). This means you can put a function inside a class but you can't access the instance of that class (this is useful when your method does not use the instance).

### 特殊类属性

|特殊属性|说明|
|-|-|
|`__class__`|(实例的)类|
|`__name__`|类名|
|`__doc__`|文档字符串|
|`__bases__`|所有父类构成的元组|
|`__dict__`|类的属性|

## 实例(instance)

### 初始化
1. `__init__`方法用于初始化实例, 返回`None`
2. `__new__`用于对内建类型进行派生
3. `__del__`是解构函数, 一般不要去手动实现

### 实例属性 vs 类属性
python能够在运行时直接创建实例属性, 但要小心会被类属性覆盖(Override)

```python
>>> class Foo(object):
...     x = 1.5  # 类属性
>>> foo = Foo()  # 创建实例
>>> foo.x        # 因为不存在实例属性, 此处调用类属性
1.5
>>> foo.x = 1.7  # 在运行时直接创建实例属性
>>> foo.x
1.7
>>> Foo.x        # 类属性没变
1.5
>>> del foo.x    # 删除实例属性
>>> foo.x        # 类属性重见天日
1.5
```
此外, 如果类属性是可变对象, 通过实例属性修改类属性是很危险的

```python
>>> class Foo(object):
...     x = {2003: 'poe2'}  # 类属性
>>> foo = Foo()
>>> foo.x
{2003: 'poe2'}
>>> foo.x[2004] = 'valid patch'  # 添加一个key
>>> foo.x
{2003: 'poe2', 2004: 'valid patch'}  # 生效
>>> Foo.x
{2003: 'poe2', 2004: 'valid patch'}  # 类属性也变了
>>> del foo.x
AttributeError: x  # 没有覆盖所以无法删除
```

## 子类和派生
### 继承
派生类会继承基类的所有属性, 无论是数据属性还是方法属性. 如果出现了同名属性, 会发生覆盖:
```python
class P(object):
    def foo(self):
        print('foo from P')

class C(P):
    def foo(self):
        print('foo from C')

>>> c = C()
>>> c.foo()  # 调用派生类的foo
foo from C
```
在类的继承中, 子类需要调用父类被覆盖的同名方法怎么办呢? 我们不希望复制父类中的代码, 可以用子类的实例启动父类的方法, 需要传入`self`参数:
```
>>> P.foo(c) # 通过传入派生类实例来调用基类的foo
```


对于`__init__`而言,  派生类方法会覆盖基类方法的构造函数, 因此需要在派生类中手动调用. 
```python
class Inherited(Base):
    def __init__(self, inhData, bseData):
        Base.__init__(self, bseData)  # 派生类调用基类方法
        self. inhData = inhData
```
此时如果继承关系复杂, 还可以使用`super()`帮你处理好父类
```python
class Inherited(Base):
    def __init__(self, inhData, bseData):
        super(Inherited, self).__init__(bseData)
        self. inhData = inhData
```

### built-in类型的派生
待补充

### 多重继承的MRO问题
现在python的MRO(Method Resolution Order)搜索采用广度优先

## 类和实例的内建函数
### 基本方法
|方法|说明|
|-|-|
|`issubclass(sub, sup)`|sub是sup的子类:True|
|`isinstance(obj, cls)`|obj是cls或者其子类的一个实例|

### 用魔法方法定制类
有关Magic Method的列表, 自行谷歌吧

### 私有化
python没有c++中的private关键字, 使用双下划线法'混淆'属性:
```python
class C(object):
    def __init__(self):
        self.__num = 0  # 解释器重命名为self._C__num
```
这么做的主要原因是保护变量不与父类的名空间冲突, 子类中的代码就可以安全使用

## 高级特性
### `__slot__`类属性
`__dict__`跟踪了所有属性, `inst.foo`和`inst.__dict__['foo']`等价. 如果某一个类的实例太多, 那么`__dict__`会占据太多空间, 可以用`__slot__`代替. 其为一可迭代对象, 用户不再可以任意动态增加实例属性
### 描述符
待补充

### 元类和`__metaclass__`
基于python中万物皆为类的概念, 元类也是一个类, 而它的实例是其他的类:
```python
class C(object):
    pass

class CC:
    pass

>>> type(C)
<type 'type'>
>>> type(CC)
<type 'classobj'>

>>> import types
>>> type(CC) is types.ClassType  # CC是元类ClassType
True
```

在执行类定义的时候, 解释器必须要知道这个类的正确的元类, 首先寻找类属性的`__metaclass__`, 如果此属性存在, 就把这个属性赋值给此类作为它的元类. 如果没有定义, 会继续向上查找.

元类一般用于创建类. 元类初始化时通常传递三个参数: **类名**, **从基类继承数据的元组**和**类的属性字典**

以下是一个元类应用, 它在创建类时显示时间标签.
```python
from time import ctime


class MetaC(type):
    def __init__(cls, name, bases, attrd):
        super(MetaC, cls).__init__(name, bases, attrd)
        print('%s created at %s' % (name, ctime()))

class Foo(object):
    __metaclass__ = MetaC
    def __init__(self):
        print('instantiated class %s at %s' % (self.__class__.__name__, ctime()))

>>> f = Foo()
```
脚本输出
```
Foo created at Mon Sep 25 16:48:37 2017
instantiated class Foo at Mon Sep 25 16:48:37 2017
```


