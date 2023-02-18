# Python数据分析笔记1 numpy


用了这么久numpy居然没有总结一下

<!--more-->

![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/56167711_p0.jpg)

```python
import numpy as np
```

## ndarray属性


```python
# 初始化
a = np.array([[1, 2], [3, 4]])
```


```python
# 第一维的维数
a.ndim
```




    2




```python
# 所有维的维数
a.shape
```




    (2, 2)




```python
# 数据类型
a.dtype
```




    dtype('int64')




```python
# 改变数据类型
a.astype(np.float64)
```




    array([[ 1.,  2.],
           [ 3.,  4.]])



## ndarray内置初始化方法


```python
np.ones((2, 3))
```




    array([[ 1.,  1.,  1.],
           [ 1.,  1.,  1.]])




```python
np.zeros((2, 3))
```




    array([[ 0.,  0.,  0.],
           [ 0.,  0.,  0.]])




```python
np.arange(5)
```




    array([0, 1, 2, 3, 4])



## 数组和标量的运算


```python
# 相同尺寸的数组运算全部是element-wise的
a = np.ones((2, 3))
b = np.ones((2, 3)) / 2
a + b
```




    array([[ 1.5,  1.5,  1.5],
           [ 1.5,  1.5,  1.5]])




```python
# 数组和标量的运算全是broadcasting
a * 3
```




    array([[ 3.,  3.,  3.],
           [ 3.,  3.,  3.]])



## 索引和切片


```python
a = np.array([[1, 2, 3],     # 0
              [4, 5, 6],     # 1
              [7, 8, 9],     # 2
              [10, 11, 12]]) # 3
              #0  #1  #2
```


```python
# 两种索引方式等价
a[1, 2] == a[1][2] == 6
```




    True




```python
# 花式索引: 传入一个数组
a[[0, 2]]
```




    array([[1, 2, 3],
           [7, 8, 9]])




```python
# 花式索引: 传入两个数组, 等价于取出了[1, 1], [3, 2], [2, 1]
a[[1, 3, 2], 
  [1, 2, 1]]
```




    array([ 5, 12,  8])




```python
# 切片
a[1:2]
```




    array([[4, 5, 6]])




```python
# 多维切片. 注: 切片都是引用, 不是拷贝
a[1:3, 1:3]
```




    array([[5, 6],
           [8, 9]])



## 布尔索引


```python
# masking, mask条件可以使用 |, & 等合并
a = np.array([1,2,3,1,2,3,1,2])
mask = (a == 1) | (a == 2)
mask
```




    array([ True,  True, False,  True,  True, False,  True,  True], dtype=bool)




```python
# mask的长度必须和数组的(第一个)维度一致
a[mask]
```




    array([1, 2, 1, 2, 1, 2])



## element-wise 函数

这里只列举一些常用的.
### 一元函数

|函数|简介|
|-|-|
|sqrt|平方根|
|exp|指数|
|log, log1p|log(x), log(x+1)|
|sign|符号|
|isinf|无穷大的mask|

### 二元函数

|函数|简介|
|-|-|
|add(A, B)|求和|
|subtract(A, B)|element-wise的A减去B|
|multiply(A, B)|element-wise乘法|
|power(A, B)|element-wise的A的B次方|
|isinf|无穷大的mask|
|greater(A, B)|A>B的element-wise的mask|

## where指令


```python
x = np.array([0, 1, 2, 3, 4])
y = np.array([5, 6, 7, 8, 9])
flag = np.array([True, False, True, False, True])
```


```python
np.where(flag, x, y)
```




    array([0, 6, 2, 8, 4])



1. flag是一个mask数组, 值为True的赋予x中的值, 值为False的赋予y中的值
2. x和y可以是两个数, 也可以是两个数组, 其大小甚至可以不相等

## 统计学函数

只介绍一下sum, 其他函数的用法类似.
其他常用函数还有mean, (arg)min, (arg)max, var, std等, 见中文版P104


```python
a = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9],
              [10, 11, 12]])
```


```python
# 在行的维度上求和
a.sum(axis=0)
```




    array([22, 26, 30])




```python
# 在列的维度上求和. 注意此时变成了形状为(1, 4)的数组
a.sum(axis=1)
```




    array([ 6, 15, 24, 33])




```python
# 在列的维度上求和时, 同时保持原数组的形状(4, 1)
a.sum(axis=1, keepdims=True)
```




    array([[ 6],
           [15],
           [24],
           [33]])



## 线性代数库
懒得介绍了, 翻书吧, 中文版P109

## 随机数
所有函数都在np.random模块下.

|函数|说明|
|-|-|
|shuffle(a)|in-place随机打乱序列|
|rand(d1, d2..dn)|按(d1,d2..dn)产生(0,1)均匀分布|
|randint(low, high, size=(d1, d2..dn))|从(floor, ceil)内随机取整数, 产生(d1,d2..dn)分布|


```python

```

