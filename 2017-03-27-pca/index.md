# PCA


先看Refer, 该介绍的都介绍到位了:
> 17/10/09 更新了预备知识和详细计算过程

<!--more-->

 - [机器学习中的数学(4)-线性判别分析（LDA）, 主成分分析(PCA)](http://www.cnblogs.com/LeftNotEasy/archive/2011/01/08/lda-and-pca-machine-learning.html)
 - [强大的矩阵奇异值分解(SVD)及其应用](http://www.cnblogs.com/LeftNotEasy/archive/2011/01/19/svd-and-applications.html)
 - 机器学习西瓜书
 - [CS231n翻译](https://zhuanlan.zhihu.com/p/21560667?refer=intelligentunit)
 - 深度学习(花书)

---
## 预备知识
### 坐标变换
设$\vec x$在$\Bbb R^d$的坐标为
$$ \vec x = [x\_1, x\_2, x\_3, \cdots, x\_d]^T $$
$W$是$\Bbb R^d$的一组正交标准基:
$$ W = [\vec w\_1, \vec w\_2, \vec w\_3, \cdots, \vec w\_{d'}] $$
假设$d'<d$, 则$W$的列空间$\Bbb R^{d'}$是参考系的子空间, 或者说是参考系中的超平面. 

$\vec x$在超平面上的投影即为$\vec x$在$\Bbb R^{d'}$的坐标, 设为$\vec c$:
$$ \vec c = W^T \vec x \tag{1}$$
而$\vec c$在$\Bbb R^d$的坐标是
$$ \hat x = W \vec = WW^T \vec x c \tag{2}$$

### 迹和矩阵
矩阵的迹和Frobenius范数的关系:
$$
||A||\_F = \sqrt{tr(AA^T)}
\tag{3}
$$
矩阵乘积顺序不改变迹的结果
$$
tr(ABC) = tr(CAB) = tr(BCA) 
\tag{4}
$$
## 协方差矩阵特征值分解方法
PCA需要满足的优化目标为:

1. 最近重构性: 样本点到超平面/投影点的距离最近
2. 最大可分性: 样本点在超平面上的距离足够分开(即方差最大)

可以证明两个目标是等价的.

### 优化目标
假设样本进行了中心化
$$ \sum\_i x\_i = 0 \tag{5}$$

根据**最近重构性**写出优化目标
$$
\begin{align}
arg \min || \vec x - \hat x || & = || \vec x - W \vec c || \\\
& = || \vec x - W W^T \vec x ||
\end{align}
\tag{6}
$$
假设样本由$m$个列向量组成:
$$
X = [\vec x\_1, \vec x\_2,  \cdots, \vec x\_m]
\tag{7}
$$

优化目标转化为
$$
\begin{align}
arg \min || X - \hat X ||\_F & = || X - W C ||\_F \\\
& = || X - W W^T X ||\_F \\\
& = tr((X - W W^T X)(X - W W^T X)^T) \\\
& = tr((X - W W^T X)(X^T - X^TWW^T)) \\\
\end{align}
\tag{8}
$$
此处使用了矩阵Frobenius范数的迹表示. 展开后得到
$$
\begin{align}
tr(XX^T - XX^TWW^T - WW^TXX^T + WW^TXX^TWW^T)
\end{align}
\tag{9}
$$

1. $XX^T$和优化目标无关, 可忽略
2. $tr(XX^TWW^T) = tr(WW^TXX^T) = tr(W^TXX^TW)$
3. 第4项可简化为
$$ \begin{align}
tr(WW^TXX^TWW^T) &= tr(WW^TWW^TXX^T) \\\ 
&= tr(WW^TXX^T) \\\ 
&= tr(W^TXX^TW) 
\end{align} $$

最后得到优化结果:
$$
arg\min\_W -tr(W^TXX^TW) \Leftrightarrow arg\max\_W tr(W^TXX^TW)
\tag{10}
$$

### 两个条件的等价性
注意到$(10)$中$W^TX = C$, $X^TW = C^T$, $W^TXX^TW = CC^T$, 而$tr(CC^T) = tr(C^TC) = \sum\limits\_{i=1}^m \vec c\_i^T \vec c\_i =  \sum\limits\_{i=1}^m || \vec c\_i ||^2$, 这正是**最大可分性**的优化目标, 因此两者是等价的

### 特征值分解

由于$W$有$d'$个正交标准基, 假设$d' = 1$, 取$\vec w\_1$观察优化目标:
$$
arg\max\_{\vec w\_1} tr(\vec w\_1^TXX^T\vec w\_1)
$$
由于结果是标量, 设
$$
tr(\vec w\_1^TXX^T\vec w\_1) = \vec w\_1^TXX^T\vec w\_1 = \lambda\_1 = \vec w\_1^T \lambda\_1  \vec w\_1
$$

即
$$
 XX^T\vec w\_1 = \lambda\_1  \vec w\_1
$$
对协方差矩阵$XX^T$做特征值分解, 使$\lambda\_1$最大即为找到最大的特征值与特征向量. 推广至$W$, 即为找到前$d'$组最大的特征值与特征向量.

## SVD方法
实践中常常直接对$X$进行SVD分解:
$$X\_{d \times m} \approx U\_{d \times r} \Sigma\_{r \times r} V^T\_{r \times m}$$
其中$U$和$V$都是正交标准矩阵, $\Sigma$是对角方阵. 因此
$$
\begin{align}
XX^T &= U\_{d \times r} \Sigma\_{r \times r} V^T\_{r \times m} V\_{m \times r} \Sigma\_{r \times r} U^T\_{r \times d} \\\
&= U\_{d \times r} \Sigma\_{r \times r}^2 U^T\_{r \times d}
\end{align}
$$
因此这里的$U$即为上一节的$W$. 稍作变换有:
$$ XX^T U\_{d \times r} = U\_{d \times r} \Sigma\_{r \times r}^2$$
这意味着对列进行了压缩, 从$d$列压缩到了$r$列, 这里的$r$实际上就是PCA中的$d'$

此外, 分析一下压缩空间中的点可以发现:
$$
CC^T = (W^TX)(W^TX)^T = W^TXX^TW = W^T W \Sigma^2 W^TW = \Sigma^2
$$
因为$\Sigma$是对角阵, 因此压缩空间中的元素彼此无关(协方差为0)

## 总结
 - 主成分分析(Principal Component Analysis, PCA)是无监督算法, 使用了特征值分解, SVD等方法, 通过选择方差最大的投影来压缩矩阵信息, 去掉不重要/线性相关的信息, 保留重要/正交信息
 - 线性判别分析(Linear Discriminant Analysis, LDA)是有监督算法, 目标是投影后类内方差小, 类间方差大

## Numpy实现
除了PCA以外, 也包括了一些常用的数据预处理

1. 均值减法(Mean subtraction): 将均值变为0, 也叫中心化
 `X -= np.mean(X, axis=0)`
2. 归一化(Normalization): 将方差变为1
 `X /= np.std(X, axis=0)`
3. PCA
 假设输入数据矩阵X的尺寸为[N x D], 在中心化之后:
  - 得到数据的协方差矩阵
  `cov = np.dot(X.T, X) / X.shape[0]` 
  - 作SVD分解, U的列是特征向量，S是装有奇异值的1维数组
 `U,S,V = np.linalg.svd(cov)`  
  - PCA: 取前100个基进行压缩
 `Xrot_reduced = np.dot(X, U[:,:100])` 
4. 白化(whiten):
 `Xwhite = Xrot_reduced / np.sqrt(S + 1e-5)`
 这个算法中让压缩矩阵的每个维度都除以其特征值, 使得协方差相等, 表现在数据中就是使得整个数据符合高斯分布
