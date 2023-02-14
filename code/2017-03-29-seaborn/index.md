# 数据可视化工具Seaborn指南


Seaborn是基于matplotlib的封装, 使得制作图表更为简便

<!--more-->

## 一维数据的分析


```python
%matplotlib inline

import numpy as np
import pandas as pd
from scipy import stats, integrate
import matplotlib.pyplot as plt

import seaborn as sns
sns.set(color_codes=True)
```

### 1. `displot()`绘制柱状图
 - bins={int}: 柱子的密度
 - kde={True, False}: 是否显示KDE拟合密度曲线
 - rug={True, False}: 是否显示条状密度标志


```python
x = np.random.normal(size=100)
sns.distplot(x, bins=20, kde=True, rug=True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11693bdd0>




![png](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/output_3_1.png)


### 2. `jointplot()`绘制二维柱状图
 - x: x坐标
 - y: y坐标
 - data: DataFrame数组


```python
mean, cov = [0, 1], [(1, .5), (.5, 1)]
data = np.random.multivariate_normal(mean, cov, 200)
df = pd.DataFrame(data, columns=["x", "y"])

sns.jointplot(x="x", y="y", data=df)
```




    <seaborn.axisgrid.JointGrid at 0x11923bbd0>




![png](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/output_5_1.png)


## 二维数据的分析

### 1. `regplot()`做回归分析
参数和`jointplot()`类似


```python
tips = sns.load_dataset("tips")
sns.regplot(x="total_bill", y="tip", data=tips)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x1169a6cd0>




![png](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/output_8_1.png)


### 2. `lmplot()`更高自由度的二维图绘制
 - `fit_reg=True`: 设置为`False`即可关掉线性回归
还有各种参数见文档, `regplot()`是`lmplot()`的一个子集

### 3. 用`pairplot()`展示不同特征的关系
`hue`参数可以在数据点上用颜色标注某一特征的不同值.


```python
iris = sns.load_dataset("iris")
sns.pairplot(iris, hue="species")
```




    <seaborn.axisgrid.PairGrid at 0x11c290610>




![png](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/output_11_1.png)


### 4. `heatmap()`展示不同特征的协方差


```python
corrmat = iris.corr()
sns.heatmap(corrmat, cbar=True, annot=True, square=True, fmt='.2f', annot_kws={'size': 10})
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11f1aaf10>




![png](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/output_13_1.png)


## 分类数据的分析

### 1. `stripplot()`统计各个类的数据
**注**: `jitter=True`参数可以引入横向噪音, 用于显示分布密度, 避免样本堆叠


```python
tips = sns.load_dataset("tips")
sns.stripplot(x="day", y="total_bill", data=tips)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11dc739d0>




![png](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/output_15_1.png)


### 2. `boxplot()`绘制箱形图


```python
sns.boxplot(x="day", y="total_bill", data=tips)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11ddf3ad0>




![png](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/output_17_1.png)



```python

```

