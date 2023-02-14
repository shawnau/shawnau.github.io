# 决策树 (Decision Tree)


决策树

<!--more-->

## CN ver.
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

信息增益的一个缺点是决策树会偏向于选择取值较多的特征. 例如假设每个$X$的取值对应于独一无二的值(item_id类型的数据)有:

$$
\begin{align} 
H(Y | X=x\_i) & = - \sum\limits\_{j=1}^{m} P(Y = y\_j | X=x\_i) log P(Y = y\_j | X=x\_i) \\\
& = - \sum\limits\_{j=1}^{m} \frac{p\_{ij}}{p\_i}log \frac{p\_{ij}}{p\_i} \\\
& = - \frac{1/N}{1/N}log \frac{1/N}{1/N} = 0
\end{align}
$$

$$ H(Y | X) =  \sum\limits\_{i=1}^{n} p\_i H(Y | X=x\_i) = 0 $$, 条件熵降到了0, 显然信息增益是最大的. 但这样的分类毫无意义, 属于极端过拟合.因此引入信息增益比.

首先由于$X$的划分行为, 导致数据集label$Y$的熵增加了(每一类内部的熵减少了)

$$ H\_X(Y) = - \sum\limits^{n}\_{i=1} P(X = x\_i) log P(X = x\_i) $$

$$ g\_R(Y, X) = \frac{H(Y) - H(Y|X)}{H\_X(Y)} $$

这样一来使用之前的例子, $ H\_X(Y) = - n \times \frac{1}{n} log \frac{1}{n} = - log \frac{1}{n} $, $n$越大, 分得越细, 信息增益的分母越大, 信息增益比越小.

---

## EN ver.
### 1. Sample Partition
$$
\begin{array}{c|c}
\text{A|C} &  c\_{1}, \ c\_{2}, \ \cdots, \ c\_{K}  \\\
\hline
a\_1 &  s\_{11}, s\_{12}, \cdots, s\_{1K} \\\
a\_2 &  s\_{21}, s\_{22}, \cdots, s\_{2K} \\\
a\_3 &  s\_{31}, s\_{32}, \cdots, s\_{3K} \\\
\vdots & \vdots  \\\
a\_N & s\_{N1}, s\_{N2}, \cdots, s\_{NK} 
\end{array}
$$

Assume a set of samples have $K$ different labels named  $c\_{1}, \ c\_{2}, \ \cdots, \ c\_{K}$, a feature $A$ has $N$ different values named $a\_1, a\_2, \cdots, a\_N$, for any sample $s\_{ij}$, $s\_{ij} \in A\_i \cap C\_j$

<!--more-->

---

### 2. Entropy, Conditional Entropy and Information Gain
Using Maximum Likelihood Estimation, the empirical entropy & empirical conditional entropy could be represented as:
$$
H(S) = - \sum\limits\_{j=1}^{K} P(s \in C\_j) log\_2 P(s \in C\_j) = - \sum\limits\_{j=1}^{K} \frac{|C\_j|}{|S|} log\_2(\frac{|C\_j|}{|S|}), \quad
|S| = \sum\limits\_{j=1}^{K} |C\_j|
\tag{1}
$$

$$
\begin{align}
H(S|A) & = \sum\limits\_{i=1}^{N} P(s \in A\_i)  H(S|s \in A\_i) \\\
& = \sum\limits\_{i=1}^{N} P(s \in A\_i) \left [  - \sum\limits\_{j=1}^{K} P(s\_{ij} \in A\_i \cap C\_j)log\_2 P(s\_{ij} \in A\_i \cap C\_j) \right ] \\\
& =  \sum\limits\_{i=1}^{N} \frac{|s\_{i\cdot}|}{|S|} \left [ - \sum\limits\_{j=1}^{K} \frac{|s\_{ij}|}{|s\_{i\cdot}|}log\_2 \frac{|s\_{ij}|}{|s\_{i\cdot}|}\right ]
\end{align}
\tag{2}
$$
Using the two formulas above, the information gain from $A$ in sample $S$ is:
$$g(S, A) = H(S) - H(S|A) \tag{3}$$
Using information gain to choose the feature has one disadvantage: it tends to select the feature which has the largest amount of values. To fix this, using **information gain ratio** instead:
$$
g\_R(S, A) = \frac{g(S, A)}{H\_A(S)}, \quad H\_A(S) = - \sum\limits\_{i=1}^{N} \frac{|s\_{i\cdot}|}{|S|}log\_2\frac{|s\_{i\cdot}|}{|S|}
\tag{4}
$$

---

### 3. Tree Building
1. ID3
 Input: Train set $S$, feature set $A$, tolerance $\epsilon$
 Output: Decision Tree $T$
 (1) If $s\_i \in C\_j$ for all $i$: $label = c\_k$, return T 
 (2) If $A = \varnothing$: $label=c\_j: \max(|c\_j|)$, return T
 (3) Else: calculate $g(S, A\_i)$ for all $i$, choose $A\_g$ in which $g = argmax(g(S, A\_i))$
 (4) If $g(S, A\_g) < \epsilon$: $label= c\_j: \max(|c\_j|)$, return T
 (5) Else: Split $S$ into $S\_i$ for each $a\_i$ in $A\_g$,  $label\_i= c\_j: \max(|c\_j|), c\_j \in S\_i$, return T
 (6) For each $a\_i$ as a node, train set = $S\_i$, feature set = $A - A\_g$, run (1) ~ (5) recursively, return $T\_i$

2. C4.5
 Using $g\_R(S, A)$ instead of $g(S, A)$ in ID3 step (3) and (4)
 Notice: Using information gain ratio tends to choose the feature which have less values, so C4.5 uses a heuristic method: first choose the ones have information gain above the average, then choose the one with the larger information gain ratio

---

### 4. Pruning
Assume tree $T$ has $|T|$ leaf nodes $t$, each $t$ has $N\_t$ samples, in which $N\_{tk}$ samples $\in C\_k$, for each $t$, sum all the empirical entropy to get the cost function:
$$
C\_\alpha(T) = \sum\limits\_{t=1}^{|T|} N\_t H\_t(T) + \alpha|T| \\\
H\_t(T) = -\sum\limits\_k \frac{N\_{tk}}{N\_t} log\_2 \frac{N\_{tk}}{N\_t}
\tag{5}
$$
The pruning algorithm recursively calculates $C\_\alpha(T)$ to find the minimum cost. Another prunning method called pre-prunning using the test set and train set, if the current splitting causes worse performance on the test set, then cancel the prunning

