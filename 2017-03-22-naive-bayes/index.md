# 朴素贝叶斯


朴素贝叶斯(**Naive** Bayes) 如此蛤意盎然的算法, 居然一直没写

<!--more-->

## 原理
1. 训练集:
 $T = \lbrace (X^{(1)}, y\_1), (X^{(2)}, y\_2), \cdots, (X^{(N)}, y\_N) \rbrace$, 其中
 $ X = (X\_1, X\_2, \cdots, X\_n), y \in \lbrace c\_1, c\_2, \cdots, c\_K \rbrace $

2. 学习过程: 根据贝叶斯模型, 需要学习的参数有
 $$ P(y = c\_k) = \frac{\sum\limits^{N}\_{i=1}I(y\_i = c\_k)}{N} \tag{1}$$
 $$
 \begin{align} 
 P(X\_j = X^{(i)}\_j | y = c\_k) & =  \frac{P(X\_j = X^{(i)}\_j, y = c\_k)}{P(y = c\_k)} \\\
 & = \frac{\sum\limits^{N}\_{i=1}I(X\_j = X^{(i)}\_j, y\_i = c\_k)} {\sum\limits^{N}\_{i=1}I(y\_i = c\_k)}
 \end{align} \tag{2}
 $$
 此外, 
 $$ P(X = X^{(i)} | y = c\_k) = \prod\limits^{n}\_{j=1} P(X\_j = X^{(i)}\_j | y = c\_k) \tag{3}$$

 其中(3)式利用了朴素贝叶斯的条件独立性假设

3. 预测过程:
 对于给定的实例$X = A$
 $$ \begin{align} P(y = c\_k | X = A) &= \frac{P(X = A | y = c\_k)P(y = c\_k)}{P(X = A)} \\\
 & = \frac{P(X = A | y = c\_k)P(y = c\_k)}{\sum\limits\_{k} P(X = A | y = c\_k)P(y = c\_k)} 
 \end{align}$$
 由于分母对任意$c\_k$都一样, 因此贝叶斯分类结果是
 $$ y = arg \max\limits\_{c\_k} P(X = A | y = c\_k)P(y = c\_k) \tag{4}$$

## 贝叶斯估计

如果样本比较少, 出现$P(X\_j = X^{(i)}\_j | y = c\_k) = 0$ 的情况, 那整个$P(X = X^{(i)} | y = c\_k) = 0$, 这会影响后验概率的计算结果. 使用常数项并保持概率分布的特性可以解决这一问题

$$
P\_{\lambda}(X\_j = X^{(i)}\_j | y = c\_k) = \frac{\sum\limits^{N}\_{i=1}I(X\_j = X^{(i)}\_j, y\_i = c\_k) + \lambda}{\sum\limits^{N}\_{i=1}I(y\_i = c\_k) + S\_j\lambda}
$$

$$ P(y = c\_k) = \frac{\sum\limits^{N}\_{i=1}I(y\_i = c\_k) + \lambda}{N + K\lambda} $$

其中$S\_j$代表$X$的第$j$个特征$X\_j$有多少种可能的取值. 一般$\lambda$取1, 称为拉普拉斯平滑
