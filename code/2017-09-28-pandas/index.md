# Python数据分析笔记2 pandas基础


![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/65138347_p0.png)
全面翻新的pandas介绍, 篇幅较长, 请善用右下角目录!

<!--more-->



```python
import numpy as np
import pandas as pd
```

## 数据结构
### Series

```python
Series(data=None, index=None, dtype=None, name=None, copy=False, , fastpath=False)
```
 - `data`: 可遍历序列数据
 - `index`: 索引, hsahable即可. 默认为从0开始的range
 - `dtype`: `numpy.dtype`数据类型
 - `copy`: 是否为data的副本(默认为视图, 对它的改动会直接反映到data上)


```python
# 数组初始化
s = pd.Series([4, 7, -5, 3], index=['d', 'b', 'a', 'c'])
# 字典初始化
s = pd.Series({'d': 4, 'b':7, 'a':-5, 'c':3})

```




    a   -5
    b    7
    c    3
    d    4
    dtype: int64




```python
# 转换为numpy数组
s.values
```




    array([-5,  7,  3,  4])




```python
# 取出索引
s.index
```




    Index(['a', 'b', 'c', 'd'], dtype='object')




```python
# name属性: Series的name会自动继承于DataFrame的列名, Index的name会自动继承于Series的索引名
s.name = 'population'
s.index.name = 'state'
```

### DataFrame

```python
DataFrame(data=None, index=None, columns=None, dtype=None, copy=False)
```
 - `data`: 初始化方式和Series相似, 可以传入二维数组或者字典(key作为列名, value是等长数组)
 - `columns`: 列名. 字符串列表

其他参数和Series相似


```python
# 初始化
df = pd.DataFrame(np.arange(16).reshape((4,4)), 
                  index=['ohio','colorado', 'utah', 'new york'],
                 columns=['one', 'two', 'three', 'four'])
df
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>one</th><th>two</th><th>three</th><th>four</th></tr></thead><tbody><tr><th>ohio</th><td>0</td><td>1</td><td>2</td><td>3</td></tr><tr><th>colorado</th><td>4</td><td>5</td><td>6</td><td>7</td></tr><tr><th>utah</th><td>8</td><td>9</td><td>10</td><td>11</td></tr><tr><th>new york</th><td>12</td><td>13</td><td>14</td><td>15</td></tr></tbody></table></div>



### 索引对象
1. 索引对象是不可改变的
2. 索引的方法类似于集合操作: 包括diff, intersection, union, isin, drop, unique等

## 基本功能

### reindex()
```python
DataFrame.reindex(index=None, columns=None, **kwargs)
```
 - `index`: 重排的索引顺序
 - `columns`: 重排的列顺序(DataFrame独有)
 - `method`: 插值选项
 - `fill_value`: 插值时用于填充的值
 - `limit`: 最大填充量
 - `level`: 层次索引的层级
 - `copy`: 拷贝副本

### drop()
```python
DataFrame.drop(labels, axis=0, level=None, inplace=False, errors='raise')
```
 - `labels`: index列表或者column列表, 取决于axis是'index'还是'columns'
 - `axis`: 按行/列删除
 - `level`: 层次索引的层次
 - `inplace`: 就地删除

### 索引
#### Series的索引


```python
# 取元素
s['a']
```




    -5




```python
# 切片, 注意切片是闭区间, 和数组不同
s['a':'c']
```




    state
    a   -5
    b    7
    c    3
    Name: population, dtype: int64




```python
# 花式索引
s[['c', 'a', 'b']]
```




    state
    c    3
    a   -5
    b    7
    Name: population, dtype: int64



#### DataFrame的索引


```python
# 取列
df['two']
```




    ohio         1
    colorado     5
    utah         9
    new york    13
    Name: two, dtype: int64




```python
# 取多列
df[['three', 'one']]
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>three</th><th>one</th></tr></thead><tbody><tr><th>ohio</th><td>2</td><td>0</td></tr><tr><th>colorado</th><td>6</td><td>4</td></tr><tr><th>utah</th><td>10</td><td>8</td></tr><tr><th>new york</th><td>14</td><td>12</td></tr></tbody></table></div>




```python
# 按行切片
df[:2]
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>one</th><th>two</th><th>three</th><th>four</th></tr></thead><tbody><tr><th>ohio</th><td>0</td><td>1</td><td>2</td><td>3</td></tr><tr><th>colorado</th><td>4</td><td>5</td><td>6</td><td>7</td></tr></tbody></table></div>




```python
# 掩码
df[df['three'] > 5]
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>one</th><th>two</th><th>three</th><th>four</th></tr></thead><tbody><tr><th>colorado</th><td>4</td><td>5</td><td>6</td><td>7</td></tr><tr><th>utah</th><td>8</td><td>9</td><td>10</td><td>11</td></tr><tr><th>new york</th><td>12</td><td>13</td><td>14</td><td>15</td></tr></tbody></table></div>




```python
# 索引切片
df.loc['colorado', ['three', 'one']]
```




    three    6
    one      4
    Name: colorado, dtype: int64




```python
# 数值索引切片
df.iloc[0:2, 1:3]
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>two</th><th>three</th></tr></thead><tbody><tr><th>ohio</th><td>1</td><td>2</td></tr><tr><th>colorado</th><td>5</td><td>6</td></tr></tbody></table></div>



### 算数计算和数据对齐: add(), sub(), etc.

#### Series之间和DataFrame之间的运算


```python
# 初始化
s1 = pd.Series([7.3, -2.5, 3.4, 1.5], 
               index=['a', 'c', 'd', 'e'])
s2 = pd.Series([-2.1, 3.6, -1.5, 4, 3.1],
               index=['a', 'c', 'e', 'f', 'g'])
```


```python
print(s1)
print(s2)
```

    a    7.3
    c   -2.5
    d    3.4
    e    1.5
    dtype: float64
    a   -2.1
    c    3.6
    e   -1.5
    f    4.0
    g    3.1
    dtype: float64



```python
# 加法会自动对齐索引, 不重叠的索引会引入nan
s1 + s2
```




    a    5.2
    c    1.1
    d    NaN
    e    0.0
    f    NaN
    g    NaN
    dtype: float64




```python
# 使用fill_value参数可以指定不重叠部分的默认填充值
s1.add(s2, fill_value=0)
```




    a    5.2
    c    1.1
    d    3.4
    e    0.0
    f    4.0
    g    3.1
    dtype: float64




```python
s1.add(s2, fill_value=1)
```




    a    5.2
    c    1.1
    d    4.4
    e    0.0
    f    5.0
    g    4.1
    dtype: float64



#### DataFrame和Series的运算

DataFrame和Series做加减法时, 会将Series的索引匹配到DataFrame的列, 然后沿着行一直向下广播


```python
df = pd.DataFrame(np.arange(12.).reshape((4, 3)),
                     columns=list('bde'),
                     index=['Utah', 'Ohio', 'Texas', 'Oregon'])
s1 = df.loc['Utah']  # 一行
s2 = df.loc[:, 'd']  # 一列
print(df)
print(s1)
print(s2)
```

              b     d     e
    Utah    0.0   1.0   2.0
    Ohio    3.0   4.0   5.0
    Texas   6.0   7.0   8.0
    Oregon  9.0  10.0  11.0
    b    0.0
    d    1.0
    e    2.0
    Name: Utah, dtype: float64
    Utah       1.0
    Ohio       4.0
    Texas      7.0
    Oregon    10.0
    Name: d, dtype: float64



```python
# 匹配某一列, 然后沿着每一列广播
df.sub(s2, axis='index')
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>b</th><th>d</th><th>e</th></tr></thead><tbody><tr><th>Utah</th><td>-1.0</td><td>0.0</td><td>1.0</td></tr><tr><th>Ohio</th><td>-1.0</td><td>0.0</td><td>1.0</td></tr><tr><th>Texas</th><td>-1.0</td><td>0.0</td><td>1.0</td></tr><tr><th>Oregon</th><td>-1.0</td><td>0.0</td><td>1.0</td></tr></tbody></table></div>




```python
# 匹配某一行, 然后沿着每一行传播
df.sub(s1, axis='columns')
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>b</th><th>d</th><th>e</th></tr></thead><tbody><tr><th>Utah</th><td>0.0</td><td>0.0</td><td>0.0</td></tr><tr><th>Ohio</th><td>3.0</td><td>3.0</td><td>3.0</td></tr><tr><th>Texas</th><td>6.0</td><td>6.0</td><td>6.0</td></tr><tr><th>Oregon</th><td>9.0</td><td>9.0</td><td>9.0</td></tr></tbody></table></div>



### 函数和映射: apply()

apply可以作用到每个元素, 或者应用到各行各列形成的一维数组上


```python
df = pd.DataFrame(np.random.randn(4, 3), columns=list('bde'),
                     index=['Utah', 'Ohio', 'Texas', 'Oregon'])
df
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>b</th><th>d</th><th>e</th></tr></thead><tbody><tr><th>Utah</th><td>-0.183165</td><td>0.160413</td><td>-0.774075</td></tr><tr><th>Ohio</th><td>2.241114</td><td>0.928337</td><td>0.035367</td></tr><tr><th>Texas</th><td>0.811800</td><td>-1.106693</td><td>-0.355058</td></tr><tr><th>Oregon</th><td>0.394904</td><td>1.499819</td><td>-0.235244</td></tr></tbody></table></div>




```python
f = lambda x: x.max() - x.min()
# 作用到一列形成的一维数组(默认)
df.apply(f, axis='index')
```




    b    2.424278
    d    2.606512
    e    0.809443
    dtype: float64




```python
# 作用到一行形成的一维数组
df.apply(f, axis='columns')
```




    Utah      0.934488
    Ohio      2.205746
    Texas     1.918493
    Oregon    1.735063
    dtype: float64




```python
# 作用到每个元素. map是属于Series的
f = lambda x: "%.2f" % (x)
df.applymap(f)
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>b</th><th>d</th><th>e</th></tr></thead><tbody><tr><th>Utah</th><td>-0.18</td><td>0.16</td><td>-0.77</td></tr><tr><th>Ohio</th><td>2.24</td><td>0.93</td><td>0.04</td></tr><tr><th>Texas</th><td>0.81</td><td>-1.11</td><td>-0.36</td></tr><tr><th>Oregon</th><td>0.39</td><td>1.50</td><td>-0.24</td></tr></tbody></table></div>




```python
# 形成多个统计值
def f(x):
    return pd.Series([x.min(), x.max()], index=['min', 'max'])
# 沿着索引统计每一列
df.apply(f, axis='index')
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>b</th><th>d</th><th>e</th></tr></thead><tbody><tr><th>min</th><td>-0.183165</td><td>-1.106693</td><td>-0.774075</td></tr><tr><th>max</th><td>2.241114</td><td>1.499819</td><td>0.035367</td></tr></tbody></table></div>




```python
# 沿着列统计每一行
df.apply(f, axis='columns')
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>min</th><th>max</th></tr></thead><tbody><tr><th>Utah</th><td>-0.774075</td><td>0.160413</td></tr><tr><th>Ohio</th><td>0.035367</td><td>2.241114</td></tr><tr><th>Texas</th><td>-1.106693</td><td>0.811800</td></tr><tr><th>Oregon</th><td>-0.235244</td><td>1.499819</td></tr></tbody></table></div>



### 排序

#### 按索引排序: sort_index()

默认是按升序排列, 参数ascending可以改为降序


```python
# Series按索引排序
s = pd.Series(range(4), index=['d', 'a', 'b', 'c'])
s.sort_index()
```




    a    1
    b    2
    c    3
    d    0
    dtype: int64




```python
# DataFrame按索引和列排序. 通过axis参数选择按索引还是列排序
df = pd.DataFrame(np.arange(8).reshape((2, 4)),
                     index=['three', 'one'],
                     columns=['d', 'a', 'b', 'c'])
# 按索引排序(默认)
df.sort_index(axis='index')
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>d</th><th>a</th><th>b</th><th>c</th></tr></thead><tbody><tr><th>one</th><td>4</td><td>5</td><td>6</td><td>7</td></tr><tr><th>three</th><td>0</td><td>1</td><td>2</td><td>3</td></tr></tbody></table></div>




```python
# 按列排序
df.sort_index(axis='columns')
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>a</th><th>b</th><th>c</th><th>d</th></tr></thead><tbody><tr><th>three</th><td>1</td><td>2</td><td>3</td><td>0</td></tr><tr><th>one</th><td>5</td><td>6</td><td>7</td><td>4</td></tr></tbody></table></div>



#### 按值排序: sort_values()

1. 通过by选择某一列
2. NaN会排在最后


```python
# Series按值排序
s = pd.Series([4, np.nan, -3, 2])
s.sort_values()
```




    2   -3.0
    3    2.0
    0    4.0
    1    NaN
    dtype: float64




```python
# DataFrame按值排序, 通过by参数选择列
df = pd.DataFrame({'b': [4, 7, -3, 2], 'a': [0, 1, 0, 1]})
df.sort_values(by=['a', 'b'])
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>a</th><th>b</th></tr></thead><tbody><tr><th>2</th><td>0</td><td>-3</td></tr><tr><th>0</th><td>0</td><td>4</td></tr><tr><th>3</th><td>1</td><td>2</td></tr><tr><th>1</th><td>1</td><td>7</td></tr></tbody></table></div>



#### 排名: rank()

1. 通过axis参数选择按索引还是列排名
2. 通过method选择排名方法


```python
# DataFrame按值rank, 通过axis参数选择按索引还是列计算排名
df.rank(method='first', axis='index')
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>a</th><th>b</th></tr></thead><tbody><tr><th>0</th><td>1.0</td><td>3.0</td></tr><tr><th>1</th><td>3.0</td><td>4.0</td></tr><tr><th>2</th><td>2.0</td><td>1.0</td></tr><tr><th>3</th><td>4.0</td><td>2.0</td></tr></tbody></table></div>



## 汇总和计算描述统计

### 求和: Sum()

```
sum(axis=None, skipna=None, level=None, numeric_only=None, **kwargs)
```
 - `axis`: 沿着索引还是列求和
 - `skipna`: 是否排除NaN. 若不排除, 只要有NaN, 和就为NaN
 - `level`: 层次索引的层次

其他平均数, 中位数, 方差, 标准差等方法类似

### 相关系数和协方差

```
corr(method='pearson', min_periods=1)
```

 - `method` : {‘pearson’, ‘kendall’, ‘spearman’}, 协方差计算公式
 - `min_periods` : 可选, 最小样本数

两个Series的相关系数和协方差的计算, 满足:

1. 按索引对齐
2. 索引重叠
3. 非NaN

DataFrame的协方差和相关系数会直接返回协方差矩阵:
> Compute pairwise correlation of columns, excluding NA/null values


```python
# 按列配对计算协方差
df.corr()
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>a</th><th>b</th></tr></thead><tbody><tr><th>a</th><td>1.000000</td><td>0.549442</td></tr><tr><th>b</th><td>0.549442</td><td>1.000000</td></tr></tbody></table></div>




```python
# 计算b列与df中每一列的协方差
df.corrwith(df.b)
```




    a    0.549442
    b    1.000000
    dtype: float64



### unique, value_counts和isin
```
Series.unique()
```
返回Series的唯一值序列

```
Series.value_counts(normalize=False, sort=True, ascending=False, bins=None, dropna=True)
```
 - `normalize`: 用频率替换频度
 - `sort`: 按频率排序
 - `ascending`: 升/降序排列, 默认是降序
 - `bins`: 自动划分阈值, 将数值类数据分为bins块
 - `dropna`: 去掉NaN


```python
s = pd.Series([7,3,2,2,5,6,6,5,9])
s.value_counts(bins=2)
```




    (1.992, 5.5]    5
    (5.5, 9.0]      4
    dtype: int64



```
DataFrame.isin(values)
```
 - `values`: 数值列表
返回masked DataFrame, 在`values`中的为True, 否则为False

1. 当values为序列时, 很显然
2. 当values为字典时, DataFrame的列匹配key, 值匹配value
3. 当values为DataFrame时, 相当于掩码, 见以下例子


```python
df = pd.DataFrame({'A': [1, 2, 3], 'B': ['a', 'b', 'f']})
other = pd.DataFrame({'A': [1, 3, 3, 2], 'B': ['e', 'f', 'f', 'e']})
print(df)
print(other)
```

       A  B
    0  1  a
    1  2  b
    2  3  f
       A  B
    0  1  e
    1  3  f
    2  3  f
    3  2  e



```python
df.isin(other)
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>A</th><th>B</th></tr></thead><tbody><tr><th>0</th><td>True</td><td>False</td></tr><tr><th>1</th><td>False</td><td>False</td></tr><tr><th>2</th><td>True</td><td>True</td></tr></tbody></table></div>



## 处理缺失数据

### dropna()
```
DataFrame.dropna(axis=0, how='any', thresh=None, subset=None, inplace=False)
```
 - `axis`: {'index', 'columns'}, 丢掉含有NaN的行/列, 默认为行
 - `how`: {‘any’, ‘all’}, 只要有NaN就丢弃还是全NaN才丢弃
 - `thresh`: 大于等于多少个NaN就丢弃
 - `subset`: 如果丢弃行, 这就是一个列名的列表, 处于这个列表中的列才会计入NaN
 - `inplace`: 就地丢弃

### fillna()
```
DataFrame.fillna(value=None, method=None, axis=None, inplace=False, limit=None, downcast=None, **kwargs)
```
参数不解释了, 只提一下value还可以是dict, Series, 乃至DataFrame, 填充方式与掩码相似

## 层次化索引
index是可以使用多组数组的, 代表了不同层次的索引


```python
# 初始化层次索引
s = pd.Series(np.random.randn(10),
              index=[['a','a','a','b','b','b','c','c','d','d'],
                    [1,2,3,1,2,3,1,2,2,3],
                    [9,8,7,9,8,7,9,8,8,7]])

```




    a  1  9    0.320638
       2  8   -0.666312
       3  7   -0.549029
    b  1  9   -0.375222
       2  8    0.942717
       3  7    0.588951
    c  1  9    2.538862
       2  8   -1.635290
    d  2  8   -1.602510
       3  7    0.187961
    dtype: float64




```python
# 索引属于MultiIndex类
s.index
```




    MultiIndex(levels=[['a', 'b', 'c', 'd'], [1, 2, 3], [7, 8, 9]],
               labels=[[0, 0, 0, 1, 1, 1, 2, 2, 3, 3], [0, 1, 2, 0, 1, 2, 0, 1, 1, 2], [2, 1, 0, 2, 1, 0, 2, 1, 1, 0]])



### 层次化索引方式


```python
s['a']
```




    1  9    0.320638
    2  8   -0.666312
    3  7   -0.549029
    dtype: float64




```python
s['b':'c']
```




    b  1  9   -0.375222
       2  8    0.942717
       3  7    0.588951
    c  1  9    2.538862
       2  8   -1.635290
    dtype: float64




```python
s[['b', 'd']]
```




    b  1  9   -0.375222
       2  8    0.942717
       3  7    0.588951
    d  2  8   -1.602510
       3  7    0.187961
    dtype: float64




```python
# 这里代表了第一层和第二层索引, 而不是行和列
s[:, 2]
```




    a  8   -0.666312
    b  8    0.942717
    c  8   -1.635290
    d  8   -1.602510
    dtype: float64



### 转化为DataFrame: unstack()
将最内层的索引变成新的列名, 形成新的DataFrame


```python
# 拆分
s.unstack()
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th></th><th>7</th><th>8</th><th>9</th></tr></thead><tbody><tr><th rowspan="3" valign="top">a</th><th>1</th><td>NaN</td><td>NaN</td><td>0.320638</td></tr><tr><th>2</th><td>NaN</td><td>-0.666312</td><td>NaN</td></tr><tr><th>3</th><td>-0.549029</td><td>NaN</td><td>NaN</td></tr><tr><th rowspan="3" valign="top">b</th><th>1</th><td>NaN</td><td>NaN</td><td>-0.375222</td></tr><tr><th>2</th><td>NaN</td><td>0.942717</td><td>NaN</td></tr><tr><th>3</th><td>0.588951</td><td>NaN</td><td>NaN</td></tr><tr><th rowspan="2" valign="top">c</th><th>1</th><td>NaN</td><td>NaN</td><td>2.538862</td></tr><tr><th>2</th><td>NaN</td><td>-1.635290</td><td>NaN</td></tr><tr><th rowspan="2" valign="top">d</th><th>2</th><td>NaN</td><td>-1.602510</td><td>NaN</td></tr><tr><th>3</th><td>0.187961</td><td>NaN</td><td>NaN</td></tr></tbody></table></div>




```python
# 聚合
s.unstack().stack()
```




    a  1  9    0.320638
       2  8   -0.666312
       3  7   -0.549029
    b  1  9   -0.375222
       2  8    0.942717
       3  7    0.588951
    c  1  9    2.538862
       2  8   -1.635290
    d  2  8   -1.602510
       3  7    0.187961
    dtype: float64



### 重排层次顺序: swaplevel()
```
# 交换两个index的层次顺序
swaplevel(indexName1, indexName2)
```

### 根据级别汇总统计
参考`Sum()`的`level`参数

### 使用DataFrame的列


```python
# 初始化并命名索引
df = s.unstack()
df.index.names = ['index1', 'index2']
df
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th></th><th>7</th><th>8</th><th>9</th></tr><tr><th>index1</th><th>index2</th><th></th><th></th><th></th></tr></thead><tbody><tr><th rowspan="3" valign="top">a</th><th>1</th><td>NaN</td><td>NaN</td><td>0.320638</td></tr><tr><th>2</th><td>NaN</td><td>-0.666312</td><td>NaN</td></tr><tr><th>3</th><td>-0.549029</td><td>NaN</td><td>NaN</td></tr><tr><th rowspan="3" valign="top">b</th><th>1</th><td>NaN</td><td>NaN</td><td>-0.375222</td></tr><tr><th>2</th><td>NaN</td><td>0.942717</td><td>NaN</td></tr><tr><th>3</th><td>0.588951</td><td>NaN</td><td>NaN</td></tr><tr><th rowspan="2" valign="top">c</th><th>1</th><td>NaN</td><td>NaN</td><td>2.538862</td></tr><tr><th>2</th><td>NaN</td><td>-1.635290</td><td>NaN</td></tr><tr><th rowspan="2" valign="top">d</th><th>2</th><td>NaN</td><td>-1.602510</td><td>NaN</td></tr><tr><th>3</th><td>0.187961</td><td>NaN</td><td>NaN</td></tr></tbody></table></div>



#### 将列转换为索引: set_index()


```python
# 把7,8两列变成索引
df.set_index([7, 8])
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th></th><th>9</th></tr><tr><th>7</th><th>8</th><th></th></tr></thead><tbody><tr><th rowspan="2" valign="top">NaN</th><th>NaN</th><td>0.320638</td></tr><tr><th>-0.666312</th><td>NaN</td></tr><tr><th>-0.549029</th><th>NaN</th><td>NaN</td></tr><tr><th rowspan="2" valign="top">NaN</th><th>NaN</th><td>-0.375222</td></tr><tr><th>0.942717</th><td>NaN</td></tr><tr><th>0.588951</th><th>NaN</th><td>NaN</td></tr><tr><th rowspan="3" valign="top">NaN</th><th>NaN</th><td>2.538862</td></tr><tr><th>-1.635290</th><td>NaN</td></tr><tr><th>-1.602510</th><td>NaN</td></tr><tr><th>0.187961</th><th>NaN</th><td>NaN</td></tr></tbody></table></div>



#### 将索引转换为列: reset_index()


```python
# 把'index1', 'index2'两个索引变成列
df.reset_index(['index1', 'index2'])
```



<div><style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }</style><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>index1</th><th>index2</th><th>7</th><th>8</th><th>9</th></tr></thead><tbody><tr><th>0</th><td>a</td><td>1</td><td>NaN</td><td>NaN</td><td>0.320638</td></tr><tr><th>1</th><td>a</td><td>2</td><td>NaN</td><td>-0.666312</td><td>NaN</td></tr><tr><th>2</th><td>a</td><td>3</td><td>-0.549029</td><td>NaN</td><td>NaN</td></tr><tr><th>3</th><td>b</td><td>1</td><td>NaN</td><td>NaN</td><td>-0.375222</td></tr><tr><th>4</th><td>b</td><td>2</td><td>NaN</td><td>0.942717</td><td>NaN</td></tr><tr><th>5</th><td>b</td><td>3</td><td>0.588951</td><td>NaN</td><td>NaN</td></tr><tr><th>6</th><td>c</td><td>1</td><td>NaN</td><td>NaN</td><td>2.538862</td></tr><tr><th>7</th><td>c</td><td>2</td><td>NaN</td><td>-1.635290</td><td>NaN</td></tr><tr><th>8</th><td>d</td><td>2</td><td>NaN</td><td>-1.602510</td><td>NaN</td></tr><tr><th>9</th><td>d</td><td>3</td><td>0.187961</td><td>NaN</td><td>NaN</td></tr></tbody></table></div>




```python

```

