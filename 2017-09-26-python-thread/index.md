# Python核心编程笔记4 多线程


即使有GIL的存在使得python的多线程显得鸡肋, 但在重I/O应用中还是很实用, 并且multiprecessing是基于threading的, 有必要理解

<!--more-->

![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/57080648_p0.jpg)

## GIL
全局解释器锁(Global Interpreter Lock, GIL)是Python虚拟机的一个特性, 保证了任意时刻只有一个线程在解释器中运行. Python虚拟机的执行方式如下:

1. 设置GIL
2. 切换到一个线程运行
3. 运行线程
    1. 执行指定数量的字节码
    2. 线程主动让出控制(如调用time.sleep(0))
4. 把线程设置为睡眠状态
5. 解锁GIL
6. 重复1~5

在调用外部代码(如c/c++扩展函数)时, GIL将锁定到这个函数返回为止. 编写扩展的程序员可以主动解锁GIL.
例如对面向I/O(调用内建C代码)的程序, GIL会在I/O调用之前被解锁, 以允许其他线程在等待I/O时运行.

## thread模块
不建议使用thread模块. thread不支持守护线程, 只要主线程退出, 所有其他线程没有被清除就退出了, 这是不安全的. 有关守护进程, 见Threading模块的相应部分.

<center>thread模块的方法</center>

|函数|说明|
|-|-|
|`start_new_thread(func,args,kw=None)`|产生一个新线程调用func|
|`allocate_lock()`|分配一个LockType类的锁对象|
|`exit()`|退出线程|

<center>LockType类型锁对象的方法</center>

|方法|说明|
|-|-|
|`acquire(wait=None)`|尝试获取锁对象|
|`locked()`|是否获取锁对象|
|`release()`|释放锁|


运行两个io进程的例子. (在python3.x中thread已被抛弃, 改名为_thread)
```python
import _thread
from time import sleep, ctime

ioSec = [4, 2]

# 一个I/O函数, 利用sleep模拟I/O操作
# ioName为函数序号, nsec:耗时, locks:锁序列, i:锁序号
def i_o(ioName, nsec, lock):
    print("I/O线程 %d 开始于 %s" % (ioName, ctime()))
    sleep(nsec)
    print("I/O线程 %d 结束于 %s" % (ioName, ctime()))
    lock.release()  # 解锁


def main():
    locks = []
    nloops = range(len(ioSec))
    # 先给所有子线程加锁
    for i in nloops:
        lock = thread.allocate_lock()
        lock.acquire()
        locks.append(lock)
    # 执行子线程
    for i, sec in enumerate(ioSec):
        _thread.start_new_thread(i_o, (i, ioSec[i], locks[i]))
    # 自旋锁
    for lock in locks:
        while lock.locked():
            pass


if __name__ == '__main__':
    main()
```

程序输出
```bash
I/O线程 0 开始于 Tue Sep 26 16:04:54 2017
I/O线程 1 开始于 Tue Sep 26 16:04:54 2017
I/O线程 1 结束于 Tue Sep 26 16:04:56 2017
I/O线程 0 结束于 Tue Sep 26 16:04:58 2017
```
虽然两个I/O线程只运行了4秒, 是最长的线程的运行时间. 但是自旋锁会顺序从第一个锁开始检查, 所以如果后面的锁如果提前释放了, 可能会浪费等待时间.

---

(可以跳过, 这段代码尝试维护一个locks列表来替代LockType对象)
```python
import _thread
from time import sleep, ctime

ioSec = [4, 2]

# 一个I/O函数, 利用sleep模拟I/O操作
# ioName为函数序号, nsec:耗时, locks:锁序列, i:锁序号
def i_o(ioName, nsec, locks, i):
    print("I/O线程 %d 开始于 %s" % (ioName, ctime()))
    sleep(nsec)
    print("I/O线程 %d 结束于 %s" % (ioName, ctime()))
    locks[i] = False  # 解锁


def main():
    # 先给所有子线程加锁
    locks = [True]*len(ioSec)
    # 执行子线程
    for i, sec in enumerate(ioSec):
        _thread.start_new_thread(i_o, (i, ioSec[i], locks, i))
    # 自旋锁
    for i in range(len(locks)):
        while locks[i]:
            pass


if __name__ == '__main__':
    main()
```

(一个小bug)
如果自旋锁实现还是和上一节一样:
```python
for lock in locks:
    while lock:
        pass
```
则在debug模式下发现`lock`始终为`True`, 即使`locks`中所有元素已经变成了`False`. 目前还不知道为何会发生这种情况. 可能是`lock`为原数组元素的引用复制, 因此一直没有改变, 所以最好还是使用序号遍历来避免这种潜在的bug. 当然使用LockType对象最为安全.

---

## threading模块
threading模块最重要的是Thread对象, 简单介绍一下:

### Thread类
```python
class threading.Thread(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)
```

 - `group`: 应为 `None`; 是为`ThreadGroup`类预留的参数

 - `target`: 被 `run()` 调用的函数(或者其他可调用对象), 默认为`None`

 - `name`: 线程名, 默认为“Thread-N”, 其中N是个小自然数

 - `args`: trarget的可变长参数. 默认为空元组

 - `kwargs`: 关键字变量参数, 默认为空字典

 - `daemon`: 设置守护进程属性. 如果是`None`, 那么该线程是否为守护线程继承于当前线程

几个注意点:
> This constructor should always be called with keyword arguments. 
> 该构造函数必须用关键字方式初始化
> If the subclass overrides the constructor, it must make sure to invoke the base class constructor (Thread.__init__()) before doing anything else to the thread.
> 如果子类覆盖了构造函数, 必须首先调用基类的构造函数

守护线程的解释:
如果某个线程被设置为主线程的守护进程(通过`Thread.setDaemon(True)`), 那么主线程退出时不用等待这些子线程完成.

### Thread类的其他方法:

|方法|说明|
|-|-|
|`start()`|开始执行线程|
|`run()`|定义线程功能, 一般被子类覆盖|
|`join(timeout=None)`|程序挂起直到线程结束或者timeout|
|`getName(),setName(name)`|获取/设置线程名|
|`isAlive()`|线程是否还在运行中|
|`isDaemon()`|是否为守护进程|
|`setDaemon()`|设置守护进程属性, 一定要在`start()`前调用|

### 使用Thread类

1. 创建一个Thread实例, 传给它一个函数
2. 创建一个Thread实例, 传给它一个可调用的类对象. 对象必须实现`__init__`和`__call__`方法
3. 从Thread派生出一个子类, 创建一个这个子类的实例, 覆盖基类的`run()`

此处只提供一下最后一种方法的实现:

```python
import threading
from time import ctime, sleep


class MyThread(threading.Thread):
    def __init__(self, func, args, name=''):
        threading.Thread.__init__(self)
        self.func = func
        self.args = args
        self.name = name

    def run(self):
        print("线程 %s 开始于 %s" % (self.name, ctime()))
        self.res = self.func(*self.args)
        print("线程 %s 结束于 %s" % (self.name, ctime()))

    def getResult(self):
        return self.res


def i_o(x):
    sleep(x)
    return x


def main():
    fib_num = [[3], [2], [4]]
    threads = []

    for i, num in enumerate(fib_num):
        threads.append(MyThread(i_o, num, i))

    for item in threads:
        item.start()

    for item in threads:
        item.join()
        print("%s: %s" % (item.name, item.getResult()))


if __name__ == '__main__':
    main()
```
输出结果:
```bash
线程 0 开始于 Tue Sep 26 21:41:57 2017
线程 1 开始于 Tue Sep 26 21:41:57 2017
线程 2 开始于 Tue Sep 26 21:41:57 2017
线程 1 结束于 Tue Sep 26 21:41:59 2017
线程 0 结束于 Tue Sep 26 21:42:00 2017
0: 3
1: 2
线程 2 结束于 Tue Sep 26 21:42:01 2017
2: 4
[Finished in 4.2s]
```

1. 其中由于`apply()`已经被弃用, 使用`func(*args)`替代, 所以`args`需要是可迭代对象, 就用了数组.
2. 这里有一个有趣的地方, 在最后一个循环中, `print("%s: %s" % (item.name, item.getResult()))`是写在循环内部的. 这意味着主线程必须按顺序等待子线程结束之后才会join后续线程. 整个过程可以简单表示如下:

```bash
t 0   2   3   4
m +***+***+***+
  |   |   |   |
0 +-------+   |
  |   |       |
1 +---+       |
  |           |
2 +-----------+

+ 开始/结束
* 等待
- 运行 
```
 1. 主线程同时启动三个子线程
 2. join子线程0, 等待0结束
 2. 子线程1结束
 2. 子线程0结束, 输出`0:3`
 3. join子线程1, 发现已经结束了, 输出`1:2`
 3. join子线程2, 等待2结束
 4. 子线程2结束, 输出`2:4`
 
 假如有100个线程同时开始, 最长子线程时间为3秒, 按这样的循环, 会导致时间浪费吗? 答案是不会.
 有关守护线程的机制, 参考[这篇问答](https://stackoverflow.com/questions/15085348/what-is-the-use-of-join-in-python-threading)
