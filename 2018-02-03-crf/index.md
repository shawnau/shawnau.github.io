# 对数线性模型, MEMM与CRF

![](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/13010631_p0_master1200.jpg)
>你的PGM也会和Steins Gate的世界线一样收束吗?

可能是有史以来公式最多的文章. 敲到吐血

<!--more-->

## 序
设输入是$x$, 输出是$y$, HMM刻画的是联合概率分布$P(x, y | \lambda)$, 因此是生成模型, 可以同时进行推断$P(y|x, \lambda)$和解码$P(x|y, \lambda)$. 而CRF刻画的是条件概率分布$P(y|x, \lambda)$, 是判别模型

## Log-Linear Models
### 符号表
|符号|含义|
|-|-|
|$x \in \mathcal X$|input/feature|
|$y \in \mathcal Y$|output/label|
|$\vec{\phi}(x, y): \mathcal X \times \mathcal Y \rightarrow R^d$|特征函数, 将$(x, y)$映射为$R^d$维特征向量|
|$\vec{w} \in R^d$|weights for $\vec \phi$|

### 定义
Log Linear Models take the following form:

$$
\begin{align}
P(y|x; \vec w) &= \frac{1}{Z} \exp(\vec w \cdot \vec \phi(x, y)) \\\
Z &= \sum\limits\_{y\_i \in \mathcal Y} \exp(\vec w \cdot \vec \phi(x, y\_i))
\end{align}
\tag{1.1}
$$
其中$Z$是归一化常数, 使得输出构成概率分布. 这是条件概率, 因此是判别模型.
$\vec w \times \vec \phi$代表了给定$x$之后$y$的'可能性', 而模型的核心就是特征函数$\vec \phi$.

1. $\vec \phi$实际上是不同的特征函数, 每个特征函数匹配不同pattern并score, 赋予模型**分辨能力**. 例如在logistic regression中:
$$\vec \phi\_k(\vec x, y) =  \begin{cases} 
x\_k, & \text{y=1} \\\
0, & \text{y=0}
\end{cases} $$
特征函数赋予模型二分类能力(不论$\vec x$如何, 只要出现了不同的$y$, 模型就要区分出来)
2. $\vec w$让模型对$\vec \phi$ 的每个特征给予不同的权重, 赋予模型**判断特征权重**的能力
3. 使用指数控制其非负
4. 使用$Z$归一化之后就转化成了概率分布.

### 预测
给定一组容量为$n$的样本$S = \lbrace (x\_i, y\_i) \rbrace^{n}\_{i=1}$, 使用Log Linear Model计算其**对数似然**
$$
\begin{align}
\log P(S | \vec w) = \log L(\vec w) &= \log \prod\limits\_{i=1}^n P(y\_i|x\_i; \vec w) \\\
&= \sum\limits\_{i=1}^n \log P(y\_i|x\_i; \vec w)
\end{align}
\tag{1.2}
$$

### 学习
使用最大似然估计参数$\vec w$
$$
\vec w^{\*} = \arg \max\limits\_{\vec w \in R^d} \sum\limits\_{i=1}^n \log P(y\_i|x\_i; \vec w)
$$
常用的优化手段有梯度下降(负对数似然), 其中梯度计算方式如下:

$$
\begin{align}
\frac{\partial \log L(\vec w)}{\partial \vec w} &= \frac{\partial}{\partial \vec w} \sum\limits\_{i=1}^n \log P(y\_i|x\_i; \vec w) \\\
&= \frac{\partial}{\partial \vec w}  \sum\limits\_{i=1}^n \log \frac{\exp(\vec w \cdot \vec \phi(x\_i, y\_i))}{\sum\limits\_{y\_j \in \mathcal Y} \exp(\vec w \cdot \vec \phi(x, y\_j))} \\\
&= \sum\limits\_{i=1}^n \vec \phi(x\_i, y\_i) - \sum\limits\_{i=1}^n \sum\limits\_{y\_j \in \mathcal Y} P(y\_j|x\_i; \vec w) \vec \phi(x\_i, y\_j)
\end{align}
\tag{1.3}
$$

**正则化**是优化模型性能的重要方式, L2正则项为$\frac{1}{2} ||\vec w||^2$, 损失函数变为$\log L(\vec w) + \frac{1}{2} ||\vec w||^2$即可

### 案例: Logistic Regression
定义:
$$
\begin{align}
\vec x &\in R^n \\\
y &\in \lbrace 0, 1 \rbrace \\\
\vec \phi\_k(\vec x, y) &= 
\begin{cases} 
x\_k, & \text{y=1} \\\
0, & \text{y=0}
\end{cases}
\end{align}
\tag{e.0}
$$
由定义$(1.1)$求出条件概率
$$
P(y|\vec x; \vec w) = \frac{\exp(\vec w \cdot \vec \phi(\vec x, y))}{\exp(\vec w \cdot \vec \phi(\vec x, 1)) + 1} 
\tag{e.1}
$$

由$(1.2)$可知对数似然
$$
\begin{align}
\log L(\vec w) &= \sum\limits\_{i=1}^n \log p(y\_i|x\_i; \vec w) \\\
&= \sum\limits\_{i=1}^n \log \frac{\exp(\vec w \cdot \vec \phi(x\_i, y\_i))}{\exp(\vec w \cdot \vec \phi(x\_i, 1)) + \exp(\vec w \cdot \vec \phi(x\_i, 0))} \\\
&= \sum\limits\_{i=1}^n \log \frac{\exp(\vec w \cdot \vec \phi(x\_i, y\_i))}{\exp(\vec w \cdot \vec x\_i) + 1} \\\
&= \sum\limits\_{i=1}^n [\vec w \cdot \vec \phi(x\_i, y\_i) - \log (\exp(\vec w \cdot \vec x\_i) + 1)] 
\end{align} 
\tag{e.2}
$$
由$(1.3)$求出梯度
$$
\begin{align}
\frac{\partial \log L(\vec w)}{\partial \vec w}  &= \frac{\partial}{\partial \vec w} \sum\limits\_{i=1}^n [\vec w \cdot \vec \phi(x\_i, y\_i) - \log (\exp(\vec w \cdot \vec x\_i) + 1)] \\\
&= \sum\limits\_{i=1}^n [\vec \phi(x\_i, y\_i) - \frac{\exp(\vec w \cdot \vec x\_i) \vec x\_i}{\exp(\vec w \cdot \vec x\_i) + 1}] \\\
&= \sum\limits\_{i=1}^n [\vec \phi(x\_i, y\_i) - P(1|\vec x\_i; \vec w) \vec x\_i]
\end{align} 
\tag{e.3}
$$
事实上, 当$y \in \lbrace 0, 1 \rbrace$时, $(e.0)$中的特征函数可以简化为$\vec \phi(\vec x, y) = y \vec x$, $(e.2)$, $(e.3)$可以进一步简化, 但当$y$为其他取值时不通用, 因此没有扔进去算

## MEMM
把log linear models推广到序列上, 就有了MEMM

|符号|含义|
|-|-|
|$x \in \mathcal X$|input, for example the $j$’th word in a sentence|
|$\vec x = (x\_1,x\_2, \cdots, x\_T)$|input sequence|
|$s^i \in \mathcal S, 1 \leq i \leq \lvert \mathcal S \rvert$|state or output, like tag of a word|
|$\vec s = (s\_1,s\_2, \cdots, s\_T)$|state sequence|
|$\vec \phi(x, y): \mathcal X \times \mathcal Y \rightarrow R^d$|特征函数, 将$(x, y)$映射为$R^d$维特征向量|
|${\vec w} \in R^d$|weights for $\vec \phi$|


### 定义

MEMM的结构中, 状态链是马尔科夫链, 符合**局部Markov性**, 而transition和emission矩阵由**特征函数**刻画. 整个模型刻画了条件概率分布:
$$
\begin{align}
P((s\_1,s\_2, \cdots, s\_T)|\vec x)
&= \prod\limits\_{i=1}^{T} P(s\_i | (s\_1,s\_2, \cdots, s\_{i-1}), \vec x) \\\
&= \prod\limits\_{i=1}^{T} P(s\_i | s\_{i-1}, \vec x)
\end{align}
\tag{2.1}
$$
其中

1. 第一步使用了[chain rule of conditional probabilities](https://en.wikipedia.org/wiki/Chain_rule_(probability)), 可以把联合概率拆分为条件概率: 
 $$
 \begin{align}
 P(A\_n, \cdots, A\_1) &= P(A\_n | A\_{n-1}, \cdots, A\_1) P(A\_{n-1}, \cdots, A\_1) \\\
 P(A\_4, A\_3, A\_2, A\_1) &= P(A\_4|A\_3, A\_2, A\_1) P(A\_3|A\_2, A\_1) P(A\_2|A\_1)
\end{align}
 $$
2. 第2步使用了independence assumption: 
 $$P(s\_i|s\_{i-1}, s\_{i-2}, \cdots, s\_1) = P(s\_i|s\_{i-1})$$

为了计算$(2.1)$, 我们使用Log Linear Model刻画输出与输入/上一个输出之间的条件概率:
$$
P(s\_i | s\_{i-1}, \vec x; \vec w) = \frac{\exp \Bigl ( \vec w \cdot \vec \phi \bigl (  \vec x, i, s\_{i-1}, s\_i \bigr ) \Bigr )}{\sum\limits\_{s' \in \mathcal S} \exp \Bigl ( \vec w \cdot \vec \phi \bigl ( \vec x, i, s\_{i-1}, s' \bigr ) \Bigr )}
\tag{2.2}
$$

对于$ \vec \phi \bigl ( \vec x, i, s\_{i-1}, s\_i \bigr )$, 有:

1. $\vec x = (x\_1, \cdots, x\_T)$是整个输入序列
2. $i$是输出序号
3. $s\_{i-1}$是上一次输出
4. $s\_i$是输出

### 学习
获取$(x\_1,x\_2, \cdots, x\_T)$和$(s\_1,s\_2, \cdots, s\_T)$之后, 学习可以使用最大似然, 参考$(1.2)$和$(1.3)$

### 预测(Decode)
和Log Linear Models中的预测不同, 前者只需给定输入预测输出的概率分布即可, 但MEMM给定输入之后可以刻画$|\mathcal S|^T$种输出, 找到条件概率最大的输出序列就成为了一个任务:
$$
\arg \max\limits\_{(s\_1\cdots s\_T)} P\bigl ((s\_1,s\_2, \cdots, s\_T)|\vec x \bigr )
$$
这和HMM的Viterbi算法如出一辙. 使用动态规划解决:

$$
\begin{align}
\pi[i, s] &= \max\limits\_{s\_1 \cdots s\_{i-1}} P \bigl ( (s\_1, s\_2, \cdots, s\_{i}=s) | \vec x \bigr ) \\\
&= \max\limits\_{s\_1 \cdots s\_{i-1}} P\bigl ( (s\_i=s|s\_{i-1}, \vec x) \bigr)P \bigl ( (s\_1, s\_2, \cdots, s\_{i-1}) |\vec x \bigr ) \\\
&= \max\limits\_{s\_1 \cdots s\_{i-1}} P\bigl ( (s\_i=s|s\_{i-1}, \vec x) \prod\limits\_{j=1}^{i-1} P\bigl ( (s\_j|s\_{j-1}, \vec x) 
\end{align}
\tag{2.3}
$$

递推式为
$$
\pi[i, s] = \max\limits\_{s' \in \mathcal S} \pi[i-1, s'] \cdot P(s\_{i-1}=s' | s\_i=s, \vec x)
\tag{2.4}
$$

### MEMM与HMM

在计算transition & emission概率时, HMM和MEMM分别使用如下模型:
$$
\begin{align}
P(s\_i|s\_{i-1})P(x\_i|s\_i) &= A\_{s\_{i-1}s\_i}B\_{s\_i}(o\_i)\\\
P(s\_i | s\_{i-1}, \vec x) &= \frac{\exp \Bigl ( \vec w \cdot \vec \phi \bigl (  \vec x, i, s\_{i-1}, s\_i \bigr ) \Bigr )}{\sum\limits\_{s' \in \mathcal S} \exp \Bigl ( \vec w \cdot \vec \phi \bigl ( \vec x, i, s\_{i-1}, s' \bigr ) \Bigr )}
\end{align}
$$
MEMM的关键优势在于使用特征函数$\vec\phi$可以捕捉更多信息. 

 - HMM中$A \in R^{|\mathcal S| \times |\mathcal S|}$, $B \in R^{|\mathcal S| \times |\mathcal O|}$均为二维矩阵, 用来刻画transition和emission
 - MEMM中特征函数$\vec\phi \in R^{|\mathcal X| \times T \times |\mathcal S| \times |\mathcal S|}$, 捕捉特征的能力远强于HMM.

>For example, the transition probability can be sensitive to any word in the input sequence $x\_1 \cdots x\_T$. In addition, it is very easy to introduce features that are sensitive to spelling features (e.g., prefixes or suffixes) of the current word $x\_i$, or of the surrounding words. These features are useful in many NLP applications, and are difficult to incorporate within HMMs in a clean way.

并且, HMM刻画的是联合概率分布(joint probability), MEMM刻画的是条件概率分布(conditional probability), 后者往往有更高的precision

## CRF
### 定义
条件随机场的任务依然是刻画序列的条件概率$P(\vec s | \vec x)$:
$$
P(\vec s | \vec x; \vec w) = \frac{\exp \Bigl (\vec w \cdot \vec \Phi(\vec x, \vec s) \Bigr)}{\sum\limits\_{\vec s' \in \mathcal S^T} \exp \Bigl (\vec w \cdot \vec \Phi(\vec x, \vec s') \Bigr)}
\tag{3.1}
$$
然而这里的特征函数是直接建立在整个输入序列对输出序列的.
>feature vector maps an entire input sequence $\vec x$ paired with an entire state sequence $\vec s$ to some $d$-dimensional feature vector.
$$
\vec \Phi(\vec x, \vec s) = \vec x \times \vec s \rightarrow R^d
$$

但由于局部Markov性, 特征函数可以通过简单的方式进行分解
$$
\begin{align}
\vec \Phi(\vec x, \vec s) &=  \sum\limits\_{i=1}^T \vec \phi \bigl ( s\_{i-1}, s\_i, \vec x, i \bigr )
\end{align}
\tag{3.2}
$$
这里的$\phi \bigl ( s\_{i-1}, s\_i, \vec x, i \bigr )$和MEMM中的一致.

### 概率计算
由于CRF是序列对序列的, 计算概率的过程比较复杂
#### 非规范化概率的计算
非规范化概率的表示
$$
\begin{align}
\hat P(\vec s | \vec x; \vec w) &= \exp \Bigl (\vec w \cdot \vec \Phi(\vec x, \vec s) \Bigr) \\\
&= \exp  \Bigl ( \vec w \cdot \sum\limits\_{i=1}^T [ \vec t(s\_{i-1}, s\_i, \vec x, i) + \vec e(s\_i, \vec x, i) ] \Bigr ) \\\
&= \exp  \Bigl ( \sum\limits\_{k=1}^{|\vec t|}  \sum\limits\_{i=1}^T  \lambda\_k t\_k(s\_{i-1}, s\_i, \vec x, i) + \sum\limits\_{k=1}^{|\vec e|}  \sum\limits\_{i=1}^T \mu\_k e\_k(s\_i, \vec x, i)  \Bigr )
\end{align}
\tag{3.3}
$$
其中$\vec t, \vec e$分别是transition和emission特征, transition定义在边上, 依赖于当前和前一个位置, emission定义在节点上, 依赖于当前位置. $\lambda, \mu$分别是他们的权重, CRF完全由$\lambda, t, \mu, e$定义.

假设转移矩阵和位置及输入无关, 可以这样刻画transition和emission:
$$
\begin{align}
t[i][j] &= \lambda t(s^i, s^j) & s^i, s^j \in \mathcal S \\\
e[i][j] &= \mu e(x\_i, s^j) & x\_i \in \vec x, s^j \in \mathcal S
\end{align}
$$

为了处理边界条件, 定义起点为$s\_0$, 终点为$s\_{-1}$, 即
$$
\vec s = (s\_0, s\_1, \cdots, s\_T, s\_{-1})
$$


#### 规范化因子的计算: 前向算法
$(3.1)$分母是全局规范化因子, 涉及计算$\mathcal S^T$条路径的非规范化概率之和, 这和HMM中计算$P(O|\lambda)$时需要对所有状态求和是一致的, 都可以使用前向/后向算法解决.

举个例子, 假设$\mathcal S \in {a, b}$
$$
\begin{align}
\alpha\_1(a) = \hat P \bigl ( (s\_0),s\_1=a |(x\_1) \bigr ) &= 
\exp \bigl ( t(0, a) + e(a, x\_1) \bigr ) \\\
\alpha\_1(b) = \hat P \bigl ( (s\_0),s\_1=b |(x\_1) \bigr ) &=
\exp \bigl ( t(0, b) + e(b, x\_1) \bigr ) \\\
\end{align}
\tag{3.4}
$$
而
$$
\begin{align}
\alpha\_2(a) &= \hat P \bigl ( (s\_0, s\_1), s\_2=a |(x\_1, x\_2) \bigr ) \\\
&= \exp \bigl ( t(0, a) + e(a, x\_1) + t(a, a) + e(a, x\_2) \bigr ) \\\
&+ \exp \bigl ( t(0, b) + e(b, x\_1) + t(b, a) + e(a, x\_2) \bigr ) \\\
&= \alpha\_1(a) \cdot \exp \bigl ( t(a, a) + e(a, x\_2) \bigr ) \\\
&+ \alpha\_1(b) \cdot \exp \bigl ( t(b, a) + e(a, x\_2) \bigr ) 
\end{align}
\tag{3.5}
$$
由$(3.4)$和$(3.5)$可以归纳出递推式
$$
\alpha\_{t+1}(s^i) = \sum\limits\_{j = 1}^{|\mathcal S|} \alpha\_{t}(s^j) \cdot \exp\bigl (t(s^j, s^i) + e(s^i, x\_{t+1}) \bigr )
\tag{3.6}
$$
或
$$
\alpha\_{t+1}(s^i) = \sum\limits\_{j = 1}^{|\mathcal S|} \exp \bigl ( \log \alpha\_{t}(s^j) + t(s^j, s^i) + e(s^i, x\_{t+1}) \bigr )
\tag{3.7}
$$
或
$$
\beta\_{t+1}(s^i) = \log \sum\limits\_{j = 1}^{|\mathcal S|} \exp \bigl ( \beta\_{t}(s^j) + t(s^j, s^i) + e(s^i, x\_{t+1}) \bigr )
\tag{3.8}
$$
其中$\beta\_t(s^i) = \log \alpha\_t(s^i) = \log \hat P \bigl ( s\_t=s^i |(x\_1 \cdots x\_t) \bigr )$

### Decode: Viterbi算法
给定$\vec x$, 求$\arg \max\limits\_{\vec s \in \mathcal S^T} P(\vec s | \vec x; \vec w)$

$$
\begin{align}
\arg \max\limits\_{\vec s \in \mathcal S^T} P(\vec s | \vec x; \vec w) &= \arg \max\limits\_{\vec s \in \mathcal S^T} \frac{\exp \Bigl (\vec w \cdot \vec \Phi(\vec x, \vec s) \Bigr)}{\sum\limits\_{\vec s' \in \mathcal S^T} \exp \Bigl (\vec w \cdot \vec \Phi(\vec x, \vec s') \Bigr)} \\\
&= \arg \max\limits\_{\vec s \in \mathcal S^T} \vec w \cdot \vec \Phi(\vec x, \vec s) \\\
&= \arg \max\limits\_{\vec s \in \mathcal S^T} \vec w \cdot \sum\limits\_{i=1}^T \vec \phi \bigl ( \vec x, i, s\_{i-1}, s\_i \bigr ) \\\
&= \arg \max\limits\_{\vec s \in \mathcal S^T} \sum\limits\_{k=1}^{|\vec t|}  \sum\limits\_{i=1}^T  \lambda\_k t\_k(s\_{i-1}, s\_i, \vec x, i) + \sum\limits\_{k=1}^{|\vec e|}  \sum\limits\_{i=1}^T \mu\_k e\_k(s\_i, \vec x, i)
\end{align}
$$

依然是求$t + s$的最大带权路径. 使用Viterbi算法解决:

1. 初始化
 $$
 \delta\_1(s^i) = t(s\_0, s\_1=s^i) \\\
 \phi\_1(s\_1) = 0
 \tag{3.9.1}
 $$
2. 递推. 对$t=[2, T]$:
 $$
 \delta\_t(s^i) = \max\limits\_{1 \leq j \leq |\mathcal S|} [\delta\_{t-1}(s^j) + t(s^j, s^i)] + e(s^i, x\_t) \\\
 \phi\_t(q\_i) = arg\max\limits\_{1 \leq j \leq |\mathcal S|} [\delta\_{t-1}(s^j) + t(s^j, s^i)]
 \tag{3.9.2}
 $$
3. 终止
 $$
 P = \max\limits\_{1 \leq i \leq |\mathcal S|} \delta\_T(s^i) \\\
 I\_T = arg\max\limits\_{1 \leq i \leq |\mathcal S|} \delta\_T(s^i)
 \tag{3.9.3}
 $$
4. 逆推最优状态序列. 对$t=T-1, T-2, \cdots, 1$
 $$
 I\_t = \phi\_{t+1}(I\_{t+1})
 \tag{3.9.4}
 $$

其中
$$
\delta\_t(s^i)=\max\limits\_{(s\_1 \cdots s\_{t-1})} P \left ((s\_1 \cdots s\_{t-1}), s\_t=s^i | \vec x; \vec w \right )
\tag{3.9.5}
$$

## Appendix: CRF的计算
> 使用矩(跃)阵(迁)运(引)算(擎)加速, 并充分利用broadcast简化代码, 降低可读性(

### 概率计算
注意$\vec s = (s\_0, s\_1, \cdots, s\_T, s\_{-1})$, $|\mathcal S|$包括起点和终点

数据结构
$$
\begin{align}
\vec \beta\_t(s) &= 
\begin{bmatrix}
\beta\_t(s^1) \\\
\vdots \\\
\beta\_t(s^{|\mathcal S|}) 
\end{bmatrix} \in R^{|\mathcal S|}\\\
T &= 
\begin{bmatrix}
t(s^1, s^1) & \cdots & e(s^1, s^{|\mathcal S|}) \\\
\vdots & \ddots & \vdots \\\
t(s^{|\mathcal S|}, s^1) & \cdots & e(s^{|\mathcal S|}, s^{|\mathcal S|}) 
\end{bmatrix} \in R^{|\mathcal S| \times |\mathcal S|} \\\
E &= 
\begin{bmatrix}
e(s^1, x\_{1}) & \cdots & e(s^1, x\_{T}) \\\
\vdots & \ddots & \vdots \\\
e(s^{|\mathcal S|}, x\_{1}) & \cdots & e(s^{|\mathcal S|}, x\_{T})
\end{bmatrix} \in R^{|\mathcal S| \times T}
\end{align}
\tag{a.1}
$$
那么根据$(3.8)$
$$
\begin{align}
\beta\_{t+1}(s^i) &=  \log \sum\limits\_{j = 1}^{|\mathcal S|} \exp \left (
\begin{bmatrix}
\beta\_t(s^1) \\\
\vdots \\\
\beta\_t(s^{|\mathcal S|}) 
\end{bmatrix} + 
\begin{bmatrix}
t(s^1, s^i) \\\
\vdots \\\
t(s^{|\mathcal S|}, s^i) 
\end{bmatrix} + 
\begin{bmatrix}
e(s^i, x\_{t+1}) \\\
\vdots \\\
e(s^i, x\_{t+1})
\end{bmatrix} \right ) \\\
&= \log \sum\limits\_{j = 1}^{|\mathcal S|} \exp \left (\vec \beta\_t(s) + T[:, i] + E[i, t+1]\right ) \\\
\end{align}
\tag{e.1}
$$
再根据广播(broadcasting)运算, 不难得到
$$
\begin{align}
\vec \beta\_{t+1}^T(s) &= 
\log \sum\limits\_{j = 1}^{|\mathcal S|} \exp \left (\vec \beta\_t(s) + T + E^T[:, t+1] \right )
\end{align}
\tag{e.2}
$$

在$(e.1)$中, $E[i, t+1]$被广播到$\vec \beta\_t(s) + T[:, i]$的每一行
在$(e.2)$中, $\vec \beta\_t(s)$被广播到$T$的每一列, $E^T[:, t+1]$被广播到T的每一行, 最后逐列求`log_sum_exp`

```python
def neg_log_likelihood(self, x, s):
    """
    计算非规范化概率, 然后调用_forward_algo计算规范化因子, 返回负对数概率
    :param x: input, words encoded in a sentence
    :param s: output, tag sequence to calculate probability
    :return: negative log probability of given tag sequence
    """
    score = autograd.Variable(torch.FloatTensor([0]))
    # 添加起点和终点
    s = torch.cat([torch.LongTensor([0]),
                   s,
                   torch.LongTensor([self.num_label-1])])
    # emission直接用lstm求出来
    emits = self._lstm_forward(x)
    for i in range(len(emits)):
        score = score + self.trans[s[i+1], s[i]] + emits[i][s[i+1]]
    score = score + self.trans[-1, s[-2]]
    # 计算规范化因子
    forward_score = self._forward_algo(emits)
    return forward_score - score

def _forward_algo(self, emits):
    """
    前向算法, 计算规范化因子, 涉及计算$\mathcal S^T$条路径的非规范化概率之和
    :param emits: emit scores for each word
                  Tensor(n, m), n = words in the sentence; m = num_label
    :return: Tensor(1, 1), Z(s) used to normalize probability
    """
    beta_init = torch.FloatTensor(1, self.num_label).fill_(-10000.)
    beta_init[0][0] = 0.                        # init start tag
    beta_t = autograd.Variable(init_alphas)  # track grad
    # 利用e.2式
    for emit_t in emits:
        sum_score = beta_t.view(1, -1) + self.trans + emit_t.view(-1, 1)
        beta_t = log_sum_exp(sum_score).view(1, - 1)

    terminal_var = beta_t + self.trans[-1].view(1, -1)
    Z = log_sum_exp(terminal_var)
    return Z
```

### Decode
假设
$$
\vec \delta\_t = 
\begin{bmatrix}
\delta\_t(s^1) \\\
\vdots \\\
\delta\_t(s^{|\mathcal S|})
\end{bmatrix}
\tag{e.3}
$$
参考$(a.1)$

1. For $t \in [0, T-1]$
 $$\vec \delta\_{t+1} = \max(\vec \delta\_t + T)^T + E[:, t+1] \tag{e.4}$$
2. $t=T$, 这一步没有$E$
 $$\vec \delta\_{t+1} = \max(\vec \delta\_t + T)^T \tag{e.5}$$

其中$\max$函数取函数每一列的最大值. 

```python
def _viterbi_decode(self, emits):
        """
        Decode the tag sequence with highest un-normalized log-probability(max_score)
        :param emits: emit scores for each word
        :return: max_score: un-normalized log-probability(max_score)
                 tag_seq: tag sequence with highest max_score
        """
        backpointers = []

        # Initialize the viterbi variables in log space
        init_vvars = torch.FloatTensor(1, self.num_label).fill_(-10000.)
        init_vvars[0][0] = 0.

        # forward_var at step i holds the viterbi variables for step i-1
        # actually no need to calculate grad for decoding
        forward_var = autograd.Variable(init_vvars)
        for emit_t in emits:
            # bptrs_t holds the backpointers for this step
            # viterbivars_t holds the viterbi variables for this step
            next_step_score = self.trans + forward_var
            viterbivars_t, bptrs_t = torch.max(next_step_score, dim=1)
            forward_var = viterbivars_t.view(1, -1) + emit_t.view(1, -1)
            backpointers.append(bptrs_t.data.numpy())

        # Transition to end tag. We won't add terminal_var into packpointers
        terminal_var = forward_var + self.trans[-1].view(1, -1)
        max_score, best_tag = torch.max(terminal_var, dim=1)
        best_tag = best_tag.data[0]

        # Follow the back pointers to decode the best path.
        tag_seq = [best_tag]
        for bptrs_t in reversed(backpointers):
            best_tag = bptrs_t[best_tag]
            tag_seq.append(best_tag)

        # Pop off the start tag (we dont want to return that to the caller)
        start = tag_seq.pop()
        assert start == 0  # Sanity check
        tag_seq.reverse()
        return max_score, tag_seq
```

## Reference
1. <统计学习方法> 李航
2. http://www.cs.columbia.edu/~mcollins/notes-spring2013.html
3. http://pytorch.org/tutorials/beginner/nlp/advanced_tutorial.html
