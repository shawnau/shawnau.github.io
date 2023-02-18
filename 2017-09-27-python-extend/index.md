# Python核心编程笔记5 扩展Python


C++ 和 Python 混合编程可以兼顾性能与效率

<!--more-->

![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/46228691_p0.jpg)

## Python扩展
[官方文档](https://docs.python.org/3/extending/extending.html)

> python在设计之初就考虑到让模块的导入机制足够抽象, 抽象到让使用者无法了解到模块的具体实现细节, 甚至哪种编译语言写的都分辨不出来

为何要扩展python?

1. 添加额外的功能: 有些功能python没有实现, 或者其他语言有现成的实现
2. 性能瓶颈的效率提升: 现做一个简单的代码性能测试, 找到瓶颈之后在扩展中实现
3. 保持专有代码的私密: 编译后的目标文件不容易被逆向工程

## 创建Python扩展
### 编写C++代码
这里实现了两个简单的阶乘和逆置字符串的函数
```c++
#include <iostream>
using namespace std;

/* 阶乘 */
int fac(int n)
{
    if (n < 2) {return(1);}
    return n*fac(n - 1);
}
/* 逆置字符串 */
char* rev(const char *c)
{
    string s(c);
    reverse(s.begin(), s.end());
    char *ret = strdup(s.c_str());
    return ret;
}
/* 主函数用于测试 */
int test() {
    cout << "4! == " << fac(4) << endl;
    cout << "reverse abc: " << rev("abc") << endl;
    return 0;
}
```
此处主函数被改名成了test, 主要是为了导入Python之后可以调用, 如果直接用main是比较危险的, 因此改名为test

### 用样板包装C++代码
主要需要四个步骤

1. 包含Python的头文件`Python.h`. 注意**必须放在所有include声明的最前面**
2. 为模块的每个函数增加一个形如`PyObject * Module_func()`的包装函数
3. 为每个模块增加一个形如`PyMethodDef ModuleMethods[]`的数组, 并构造`PyModuleDef`结构体
4. 增加模块初始化函数`PyInit_Module`, 向`PyModule_Create`传入`PyModuleDef`结构体

```c++
#include <Python.h>
/* 包装fac */
static PyObject * Extest_fac(PyObject *self, PyObject *args)
{
    int num;
    if (PyArg_ParseTuple(args, "i", &num))
    {
        return Py_BuildValue("i", fac(num));
    }
    return NULL;
}
/* 包装rev */
static PyObject * Extest_rev(PyObject *self, PyObject *args)
{
    char *orig_str;
    char *dupl_str;
    PyObject * retval;

    if (PyArg_ParseTuple(args, "s", &orig_str))
    {

        retval = Py_BuildValue("ss", orig_str, dupl_str=rev(strdup(orig_str)));
        free(dupl_str);
        return retval;
    }
    return NULL;
}
/* 包装test */
static PyObject * Extest_test(PyObject *self, PyObject *args)
{
    test();
    return Py_BuildValue("");
}
```
这段代码为每个被Python访问的函数包裹了一个静态函数, 函数名遵循`模块名_函数名()`的规则, 即 `Module_func()`, 主要功能如下:

1. 接收`PyObject *`类型的数据
2. 将其转换成C++数据类型, 涉及`PyArg_Parse`系列函数
3. 调用相关函数处理
4. 将输出值转换成`PyObject *`类型并返回, 涉及`Py_BuildValue`函数

简单介绍一下以上概念
Q: 什么是`PyObject`? 为什么要用`PyObject *`作为返回值?
A: `PyObject`代表了Python中任一种对象. 得益于Python处理对象的机制几乎完全一致, 在C/C++中只用一种类型来表示似乎很合理. 几乎所有Python对象都生存在堆区, 因此不可以声明auto或者静态`PyObject`, 只能用指针作为返回值.

接下来介绍一下代码中用到的两个主要函数, `PyArg_ParseTuple`和`Py_BuildValue`

### 包装函数简介
介绍一下常用的包装函数. 有关parse的详细内容, 参考官方文档: [Parsing arguments and building values](https://docs.python.org/3/c-api/arg.html)
```c++
PyArg_ParseTuple(PyObject *args, const char *format, …)
```
 - `args`: 被解析的参数变量
 - `format`: 一个字符串告诉我们如何去解析元组中每一个元素。字符串的第n个字母正是代表着元组中第n个参数的类型。例如"i"代表整形"s"代表字符串类型, "O"则代表一个Python对象. 有关含义参考文档
 - `...`: 接下来的参数都是你想要通过`PyArg_ParseTuple()`函数解析并保存的元素

这样做之后, 参数的数量和模块中函数期待得到的参数数量保持一致并保证了位置的完整性。例如我们想传入一个字符串一个整数和一个Python列表可以这样写:
```c++
int n;
char *s;
PyObject* list;
PyArg_ParseTuple(args, "siO", &n, &s, &list);
```

另外, `PyArg_ParseTuple`只能解析传入固定位置参数(positional parameters)的函数, 对于键值对参数的函数, 有`PyArg_ParseTupleAndKeywords`等其他一系列函数处理

---

```c++
PyObject* Py_BuildValue(const char *format, …)
```
 - `format`: 和`PyArg_ParseTuple`一样, 控制参数类型
 - `...`: 需要返回的对象

### 添加函数与模块
写好包装函数之后, 我们需要把它们列在某个地方, 让python解释器能够导入并调用它们. 这就是我们的`ExtestMethods`要做的事情
```c++
/* 注册每个函数 */
static PyMethodDef ExtestMethods[] = {
        {"fac", Extest_fac, METH_VARARGS},
        {"rev", Extest_rev, METH_VARARGS},
        {"test", Extest_test, METH_VARARGS},
        {NULL, NULL}
};

/* 创建模块定义 */
static struct PyModuleDef extestDemo =
        {
            PyModuleDef_HEAD_INIT,
            "Extest", /* name of module */
            "cpp extent test",          /* module documentation, may be NULL */
            -1,          /* size of per-interpreter state of the module, or -1 if the module keeps state in global variables. */
            ExtestMethods
        };

/* 初始化模块 */
PyMODINIT_FUNC PyInit_Extest(void)
{
    return PyModule_Create(&extestDemo);
}
```
注意在python 3中, 和2.x不一样的地方: 参考[这个问题](https://stackoverflow.com/questions/28305731). 这里使用的是python3

### 安装模块
在同目录下创建一个`setup.py`, 使用`distutils`来将c++代码编译成系统可用的目标文件

```python
from distutils.core import setup, Extension

MOD = 'Extest'
setup(name=MOD, ext_modules=[
    Extension('Extest', sources=['Extest.cpp'])])
```
 - `MOD`是包名
 - `Extension`的第一个参数是模块名, 如果是包下的模块的话可以使用`A.B`. `ext_modules`是个列表意味着可以同时注册包下的多个模块.

在目录下运行
```bash
$ python setup.py build
$ python setup.py install
```
然后就可以在python里调用这个包了
```python
>>> import Extest
>>> Extest.test()
4! == 24
reverse abc: cba
```

### 一些坑
在clion里编译需要在cMakeLists里加入以下几行
```
find_package(PythonLibs REQUIRED)
include_directories(${PYTHON_INCLUDE_DIRS})
```
[参考问题](https://stackoverflow.com/questions/11041299/python-h-no-such-file-or-directory)

想要卸载已经安装的包, 要记下安装路径手动删除:
```
$ python setup.py install --record files.txt
$ cat files.txt | xargs rm -rf
```

此外, 还有计数引用的问题以及GIL的问题, 待续
