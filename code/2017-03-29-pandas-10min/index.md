# pandas速查


pandas, python栈的数据处理最强package, 没有之一(?

<!--more-->

```python
# -*- coding: utf-8 -*-
```


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
```

### 1. Series()


```python
s = pd.Series([1,3,5,np.nan,6,8])
```


```python

```




    0    1.0
    1    3.0
    2    5.0
    3    NaN
    4    6.0
    5    8.0
    dtype: float64

一些实用方法
 - `sort_values()`: 按值排序
 - `sort_index()`: 按行名排序

### 2. date_range()


```python
dates = pd.date_range('20130101', periods=6)
```


```python
dates
```




    DatetimeIndex(['2013-01-01', '2013-01-02', '2013-01-03', '2013-01-04',
                   '2013-01-05', '2013-01-06'],
                  dtype='datetime64[ns]', freq='D')

### 3. DataFrame()

第一个参数是numpy数组, 第二个参数是行号(index), 第三个参数是列号(culumns), 行列号都是list类


```python
df = pd.DataFrame(np.random.randn(6,4), index=dates, columns=list('ABCD'))
df
```
<div><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>A</th><th>B</th><th>C</th><th>D</th></tr></thead><tbody><tr><th>2013-01-01</th><td>-0.507010</td><td>-2.196063</td><td>0.086058</td><td>0.489002</td></tr><tr><th>2013-01-02</th><td>-0.572061</td><td>-0.552969</td><td>-0.489184</td><td>0.090494</td></tr><tr><th>2013-01-03</th><td>-0.981979</td><td>0.199265</td><td>1.283072</td><td>0.632027</td></tr><tr><th>2013-01-04</th><td>0.406212</td><td>-1.425934</td><td>-0.361903</td><td>0.588864</td></tr><tr><th>2013-01-05</th><td>-0.614261</td><td>-0.393862</td><td>0.320324</td><td>-0.132443</td></tr><tr><th>2013-01-06</th><td>-0.186325</td><td>-0.269089</td><td>-1.798570</td><td>-0.271104</td></tr></tbody></table></div>



也可以使用字典形式初始化DataFrame类


```python
df2 = pd.DataFrame({ 'A' : 1.,
   ....:                      'B' : pd.Timestamp('20130102'),
   ....:                      'C' : pd.Series(1,index=list(range(4)),dtype='float32'),
   ....:                      'D' : np.array([3] * 4,dtype='int32'),
   ....:                      'E' : pd.Categorical(["test","train","test","train"]),
   ....:                      'F' : 'foo' })
```


```python
df2
```



<div><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>A</th><th>B</th><th>C</th><th>D</th><th>E</th><th>F</th></tr></thead><tbody><tr><th>0</th><td>1.0</td><td>2013-01-02</td><td>1.0</td><td>3</td><td>test</td><td>foo</td></tr><tr><th>1</th><td>1.0</td><td>2013-01-02</td><td>1.0</td><td>3</td><td>train</td><td>foo</td></tr><tr><th>2</th><td>1.0</td><td>2013-01-02</td><td>1.0</td><td>3</td><td>test</td><td>foo</td></tr><tr><th>3</th><td>1.0</td><td>2013-01-02</td><td>1.0</td><td>3</td><td>train</td><td>foo</td></tr></tbody></table></div>



也可以使用访问域的方法


```python
df.A
```




    2013-01-01   -0.507010
    2013-01-02   -0.572061
    2013-01-03   -0.981979
    2013-01-04    0.406212
    2013-01-05   -0.614261
    2013-01-06   -0.186325
    Freq: D, Name: A, dtype: float64



也可以使用访问字典的方法插入新的列


```python
df3 = df.copy()
df3['E'] = ['one', 'one','two','three','four','three']
df3
```



<div><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>A</th><th>B</th><th>C</th><th>D</th><th>E</th></tr></thead><tbody><tr><th>2013-01-01</th><td>-0.507010</td><td>-2.196063</td><td>0.086058</td><td>0.489002</td><td>one</td></tr><tr><th>2013-01-02</th><td>-0.572061</td><td>-0.552969</td><td>-0.489184</td><td>0.090494</td><td>one</td></tr><tr><th>2013-01-03</th><td>-0.981979</td><td>0.199265</td><td>1.283072</td><td>0.632027</td><td>two</td></tr><tr><th>2013-01-04</th><td>0.406212</td><td>-1.425934</td><td>-0.361903</td><td>0.588864</td><td>three</td></tr><tr><th>2013-01-05</th><td>-0.614261</td><td>-0.393862</td><td>0.320324</td><td>-0.132443</td><td>four</td></tr><tr><th>2013-01-06</th><td>-0.186325</td><td>-0.269089</td><td>-1.798570</td><td>-0.271104</td><td>three</td></tr></tbody></table></div>



### 4. DataFrame类的常用域和方法

1. `dtypes`: 显示每列的数据类型
2. `index`: 显示行名
3. `columns`: 显示列名
4. `values`: 显示值

#### 4.1 查看
1. `head()`, `tail()`: 显示头部和尾部, 第一个参数是显示多少行
2. `describe()`: 显示数据的统计信息
3. `info()`: 显示详细信息

#### 4.2 sort
1. `sort_index(axis=1, ascending=False)`: 按列名排序
2. `sort_values(by='col_name')`: 按列内容排序

#### 4.3 Indexing
1. `df['A']`: 按column名选择
2. `df[0:3]`: 按行号选择[0,1,2]三行
3. `df.loc[dates[0]]`: 按行名选择(自定义了行名)
4. `df.loc[:,['A','B']]`: 类似于numpy的切片
5. `df.iloc[3:5,0:2]`: 使用整数切片
6. `df.iloc[[1,2,4],[0,2]]`: 使用整数数组指定切片
7. `df.iloc[1,1]`: 得到固定位置的值

#### 4.4 Boolean Indexing
1. `df[df.A > 0]`: 列出A列>0的所有行
2. `df[df > 0]`: 所有小于等于0的元素都置为NaN

#### 4.5 Filtering


```python
df3['E'].isin(['two','four'])
```




    2013-01-01    False
    2013-01-02    False
    2013-01-03     True
    2013-01-04    False
    2013-01-05     True
    2013-01-06    False
    Freq: D, Name: E, dtype: bool



返回一个boolean列表之后, 可以使用列表切片, 类似于掩码的形式(masking)


```python
df3[df3['E'].isin(['two','four'])]
```



<div><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>A</th><th>B</th><th>C</th><th>D</th><th>E</th></tr></thead><tbody><tr><th>2013-01-03</th><td>-0.981979</td><td>0.199265</td><td>1.283072</td><td>0.632027</td><td>two</td></tr><tr><th>2013-01-05</th><td>-0.614261</td><td>-0.393862</td><td>0.320324</td><td>-0.132443</td><td>four</td></tr></tbody></table></div>



#### 4.6 更改值
1. `df.at[dates[0],'A'] = 0`: 利用label搜索
2. `df.iat[0,1] = 0`: 利用整数数组搜索
3. 甚至可以利用np数组修改


```python
df.loc[:,'D'] = np.array([5] * len(df))
df
```



<div><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>A</th><th>B</th><th>C</th><th>D</th></tr></thead><tbody><tr><th>2013-01-01</th><td>-0.507010</td><td>-2.196063</td><td>0.086058</td><td>5</td></tr><tr><th>2013-01-02</th><td>-0.572061</td><td>-0.552969</td><td>-0.489184</td><td>5</td></tr><tr><th>2013-01-03</th><td>-0.981979</td><td>0.199265</td><td>1.283072</td><td>5</td></tr><tr><th>2013-01-04</th><td>0.406212</td><td>-1.425934</td><td>-0.361903</td><td>5</td></tr><tr><th>2013-01-05</th><td>-0.614261</td><td>-0.393862</td><td>0.320324</td><td>5</td></tr><tr><th>2013-01-06</th><td>-0.186325</td><td>-0.269089</td><td>-1.798570</td><td>5</td></tr></tbody></table></div>


#### 4.7 其他

> 施工中

### 5. 缺失值处理

#### 5.1 reindex()


```python
df1 = df.reindex(index=dates[0:4], columns=list(df.columns) + ['E'])
# 把E列填入一些非NaN的值
df1.loc[dates[0]:dates[1],'E'] = 1
df1
```



<div><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>A</th><th>B</th><th>C</th><th>D</th><th>E</th></tr></thead><tbody><tr><th>2013-01-01</th><td>-0.507010</td><td>-2.196063</td><td>0.086058</td><td>5</td><td>1.0</td></tr><tr><th>2013-01-02</th><td>-0.572061</td><td>-0.552969</td><td>-0.489184</td><td>5</td><td>1.0</td></tr><tr><th>2013-01-03</th><td>-0.981979</td><td>0.199265</td><td>1.283072</td><td>5</td><td>NaN</td></tr><tr><th>2013-01-04</th><td>0.406212</td><td>-1.425934</td><td>-0.361903</td><td>5</td><td>NaN</td></tr></tbody></table></div>



#### 5.2 dropna()
`dropna()`可以去掉包含NaN值的行. 参数how的用法待考
**注意**: 不会改变df本身, 返回的是新的df


```python
df1.dropna(how='any')
```



<div><table border="1" class="dataframe"><thead><tr style="text-align: right;"><th></th><th>A</th><th>B</th><th>C</th><th>D</th><th>E</th></tr></thead><tbody><tr><th>2013-01-01</th><td>-0.507010</td><td>-2.196063</td><td>0.086058</td><td>5</td><td>1.0</td></tr><tr><th>2013-01-02</th><td>-0.572061</td><td>-0.552969</td><td>-0.489184</td><td>5</td><td>1.0</td></tr></tbody></table></div>



#### 5.3 fillna(value=VALUE_TO_FILL)
填充空值, 其中value可以是平均值,常用.

### 6. 统计操作

#### 1. `mean(axis=None, skipna=None, level=None, numeric_only=None, **kwargs)[source]`
返回平均值. 
`axis : {index (0), columns (1)}` 0返回每个特征的平均值(竖着求和), 1返回每个样本的平均值(横着求和), 默认是0

### 7. 文件存取
1. `df.to_csv('foo.csv')`: 保存到csv
2. `pd.read_csv('foo.csv')`: 读取csv

> 待续


```python

```

