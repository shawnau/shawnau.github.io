# 决策树


决策树的小结

<!--more-->

## 第一部分: 熵, 条件熵和信息增益

### 1. 熵
1. 熵的定义和变量的概率分布有关, 和变量本身的值无关, 定义如下:
 $$ H(X) = - \sum\limits\_{i=1}^{n} p\_i log p\_i \\\ 
 \text{其中} P(X=x\_i) = p\_i, i= 1,2,\cdots, n$$
2. $H(X)$的一个主要特点是$x$的分布越平均, 熵越大, 因此熵代表了分布的混乱程度
3. 另外, 对数底一般取2或e, 其单位为bit或nat

### 2. 条件熵
1. 假设两个随机变量$(X, Y)$的联合概率分布为$$P(X=x\_i, Y=y\_j) = p\_{ij} \\\ |X| = n, |Y| = m$$
2. 根据贝叶斯公式, $Y$对于$X$的条件概率为$$ P(Y = y\_j | X=x\_i) = \frac{P(X=x\_i, Y=y\_j)}{P(X=x\_i)} = \frac{p\_{ij}}{p\_i}$$
3. 那么在给定$X= x\_i$的条件下$Y$的条件概率分布的熵为
 $$
 \begin{align} 
 H(Y | X=x\_i) & = - \sum\limits\_{j=1}^{m} P(Y = y\_j | X=x\_i) log P(Y = y\_j | X=x\_i) \\\
 & =  - \sum\limits\_{j=1}^{m} \frac{p\_{ij}}{p\_i}log \frac{p\_{ij}}{p\_i}
 \end{align}
 $$
4. 最后, $H(Y | X=x\_i)$对于$X$的数学期望就是$Y$对于$X$的条件熵: 
 $$ H(Y | X) =  \sum\limits\_{i=1}^{n} p\_i H(Y | X=x\_i)$$

### 3. 一个例子
条件熵的概念, 可以通过例子理解

 - 假设$X$是某位长者前去魔都的概率分布, 去是1, 不去是0
 $$P(X=1) = 0.5, P(X=0) = 0.5$$
 - 假设$Y$是魔都天气的概率分布, 1是晴, 0是雨
 $$P(Y=1) = 0.5, P(X=0) = 0.5$$
 - 也就是说, 去不去看心情, 天气好不好看老天. 然而
 $$ P(Y=1 |X=1 ) = 1 \\\ P(Y=1 |X=0 ) = 0 \\\ P(Y=0 |X=1 ) = 0 \\\  P(Y=0 |X=0 ) = 1$$
 也就是说主席一来, 天气晴朗, 主席一走, 立马下雨
 - 那么求一下$Y$的条件概率分布的熵
 $$H(Y | X=1) = - (\frac{P\_{11}}{P\_{1}} log \frac{P\_{11}}{P\_{1}} + \frac{P\_{01}}{P\_{1}} log \frac{P\_{01}}{P\_{1}}) = -2 \\\
 H(Y | X=不去) = - (\frac{P\_{10}}{P\_{0}} log \frac{P\_{10}}{P\_{0}} + \frac{P\_{00}}{P\_{0}} log \frac{P\_{00}}{P\_{0}}) = -2
 $$
 - 最后得到条件熵 $$ H(Y | X) =p\_1 H(Y | X=1) + p\_0H(Y | X=0) = -2$$
 - 总结一下, $H(X) = H(Y) = 1$, 然而$H(Y | X) = -2$, 也就是在主席到来的约束下, $Y$的条件熵下降了, 也就是天气分布的不确定性下降了, 可见主席来不来对天气好坏是有很大影响的

### 4. 信息增益
信息增益表示得知特征$X$的信息而使得类Y的信息的不确定性减少的程度.
拿之前的例子来说, 选择主席是否来这个特征对天气数据集的信息增益为
$$ g(Y, X) = H(Y) - H(Y|X) = 1 - (-2) = 3$$

决策树的特征选择基础就是建立在选择最大信息增益的特征上的.

### 5. 信息增益比

信息增益的一个缺点是决策树会偏向于选择取值较多的特征. 例如假设每个$X$的取值对应于独一无二的值(item\_id类型的数据)有:

$$
\begin{align} 
H(Y | X=x\_i) & = - \sum\limits\_{j=1}^{m} P(Y = y\_j | X=x\_i) log P(Y = y\_j | X=x\_i) \\\
& = - \sum\limits\_{j=1}^{m} \frac{p\_{ij}}{p\_i}log \frac{p\_{ij}}{p\_i} \\\
& = - \frac{1/N}{1/N}log \frac{1/N}{1/N} = 0
\end{align}
$$

$$ H(Y | X) =  \sum\limits\_{i=1}^{n} p\_i H(Y | X=x\_i) = 0 $$
条件熵降到了0, 显然信息增益是最大的. 但这样的分类毫无意义, 属于极端过拟合, 为了解决这个问题, 引入信息增益比.

首先由于$X$的划分行为, 导致数据集label$Y$的熵增加了(每一类内部的熵减少了)

$$ H\_X(Y) = - \sum\limits^{n}\_{i=1} P(X = x\_i) log P(X = x\_i) $$

$$ g\_R(Y, X) = \frac{H(Y) - H(Y|X)}{H\_X(Y)} $$

这样一来使用之前的例子, $ H\_X(Y) = - n \times \frac{1}{n} log \frac{1}{n} = - log \frac{1}{n} $, $n$越大, 分得越细, 信息增益的分母越大, 信息增益比越小.

## 第二部分: ID3和C4.5算法

### 1. ID3
 Input: Train set $S$, feature set $A$, tolerance $\epsilon$
 Output: Decision Tree $T$

1. If $s\_i \in C\_j$ for all $i$: $label = c\_k$, return T 
2. If $A = \varnothing$: $label=c\_j: \max(|c\_j|)$, return T
3. Else: calculate $g(S, A\_i)$ for all $i$, choose $A\_g$ in which $g = argmax(g(S, A\_i))$
4. If $g(S, A\_g) < \epsilon$: $label= c\_j: \max(|c\_j|)$, return T
5.  Else: Split $S$ into $S\_i$ for each $a\_i$ in $A\_g$,  $label\_i= c\_j: \max(|c\_j|), c\_j \in S\_i$, return T
6. For each $a\_i$ as a node, train set = $S\_i$, feature set = $A - A\_g$, run (1) ~ (5) recursively, return $T\_i$

### 2. C4.5
 使用 信息增益率替代信息增益. 但信息增益率倾向于选择特征取值较少的, 故用**启发式算法**: 先选择信息增益高于平均水平的特征,再从中选择信息增益率最高的

## 第三部分: CART

### 1. 回归树

1. 训练集: 
 $$D = \lbrace (X^{(1)}, y^{(1)}), (X^{(2)}, y^{(2)}), \cdots, (X^{(N)}, y^{(N)})  \rbrace \\\ X = (x\_1, x\_2, \cdots, x\_n)$$
2. 损失函数: 平方损失函数
 $$\sum\limits\_i (y^{(i)} - f(X^{(i)}))^2$$
3. 划分手段: 对于第$f$个维度的特征$X\_f$的某个值$v$(**假设特征的值都是离散值**), 将所有样本划分为两部分:
 $$ R\_1(f, v) = \lbrace X | X\_f \leq v  \rbrace \\\ R\_2(f, v) = \lbrace X | X\_f > v  \rbrace $$
4. 搜索最佳特征$f$和值$v$
 $$ \min\limits\_{f, v}[\sum\limits\_{X^{(i)} \in R\_1} (y^{(i)} - \hat c\_1 ) + \sum\limits\_{X^{(i)} \in R\_2} (y^{(i)} - \hat c\_2 )] $$
 其中$\hat c\_1, \hat c\_2$是划分区域内部所有样本的$y^{(i)}$的均值, 这样使得平方误差最小.

### 2. 分类树

1. 基尼指数: 从数据集中随机抽取两个样本, 其数据标记不一样的概率.
2. 假设$y$有$K$个类, 每个类有$C\_k$个样本, 样本容量为|D|, 基尼指数被定义为:
$$ \begin{align}
Gini(p) & = \sum\limits^{K}\_{i = 1} p\_i(1-p\_i)  \\\ 
& = 1 -  \sum\limits^{K}\_{i = 1}p\_i^2 \\\
& = 1 -  \sum\limits^{K}\_{i = 1} (\frac{|C\_i|}{D})^2
\end{align}
$$
3. 和条件熵一样, 当数据集$D$被某一特征$A$按照是否取某一个值a的规则分割为两个集合$D\_1, D\_2$时, "条件基尼指数"为:
$$ Gini(D, A) =  \frac{|D\_1|}{|D|}Gini(D\_1) + \frac{|D\_2|}{|D|}Gini(D\_2) $$
4. CART分类树的构造规则和ID3类似, 选择条件基尼指数最大的特征递归构造

## 第四部分: 剪枝

### 1. ID3/C4.5的剪枝
损失函数为:
$$ C\_{\alpha}(T) = \sum\limits\_{t=1}^{|T|} N\_t H\_t(T) + \alpha|T| $$
其中$N\_t$代表第$t$个叶结点的容量, $H\_t(T)$代表第$t$个叶结点的熵, 而$\alpha$即为正则化参数

### 2. CART的剪枝

1. 预剪枝: 比较剪枝前后在验证集上的精度来决定是否剪枝
2. 后剪枝: 自下而上替换节点, 欠拟合风险小, 泛化能力优于预剪枝

## 第五部分: 连续与缺失值处理
### 1. 连续值: 连续特征离散化
二分法(bi-partition), C4.5使用

1. 对特征值排序:{啊,吖,阿,( ⊙ o ⊙ )啊！}
2. 取两个样本的中点作为分割点, 一共有n-1个分割点, 找到使得信息增益/率最大的分割点, 作为二分类特征处理
3. 与离散特征不同, 连续特征使用过的还可以继续使用

### 2. 缺失值
1. 对每个特征, 只取不缺失的行, 假设占总行数的比例为$\rho$
2. 改变信息增益计算方法: 在信息增益上乘以系数$\rho$即可

## 第六部分: 多变量决策树
使用特征的线性组合作为新的特征, 可以做到斜划分

## 第七部分: python实现

[源代码](https://github.com/shawnau/machine\_learning/tree/master/Decision\_Tree), 注释施工中
