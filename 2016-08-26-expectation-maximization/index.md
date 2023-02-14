# EM算法


Expectation Maximization Algorithm

<!--more-->

### 1. 三硬币模型
假设有三枚硬币A, B, C, 这些硬币正面出现的概率是$p\_a$, $p\_b$, $p\_c$.
进行如下掷硬币试验:
 1. 掷硬币A
 2. 若A为正面则选择B, 否则选择C
 3. 掷选出的硬币, 记录结果, 正面记作1, 反面记作0
 4. 独立重复n次1~3
假设只能观测到结果, 不能观察A的情况,如何估计$p\_a$, $p\_b$, $p\_c$?

三硬币模型中, 每一次掷硬币的过程可以写作
$$
\begin{align}
P(y | \theta) & = \sum\limits\_z P(y, z | \theta) = \sum\limits\_z  P(y|z,\theta) P(z |\theta) \\\
& = p\_a p\_b^y (1 - p\_b)^{1-y} + (1 - p\_a) p\_c^y (1-p\_c)^{1 - y}
\end{align}
$$
其中 $\theta = (p\_a, p\_b, p\_c)$, y为观测结果, z为掷硬币A的结果(即隐变量).
推广到掷n次硬币, 观测序列为$Y = (y\_1, y\_2, \cdots y\_n)^T$的似然函数(likelihood function)为
$$
\begin{align}
P(Y | \theta) & = \prod\_{j=1}^n  P(y\_j | \theta) = \prod\_{j=1}^n\sum\limits\_z  P(y\_j, z | \theta)\\\
&= \prod\_{j=1}^n \left\lbrace P(y\_j, z=1 | \theta) + P(y\_j, z=0 | \theta) \right\rbrace \\\
& = \prod\_{j=1}^n [p\_a p\_b^{y\_j} (1 - p\_b)^{1-y\_j} + (1 - p\_a) p\_c^{y\_j} (1-p\_c)^{1 - y\_j}]
\end{align}
$$

使用极大似然估计(MLE)来估计参数:
$$ 
\hat \theta = arg\max\limits\_{\theta} logP(Y | \theta)
$$
该问题没有解析解, 只能通过迭代求解, 而EM算法就是用于迭代求解该问题的方法之一. 因此EM算法就是含有隐变量的概率模型参数的极大似然估计

---

### 2. EM算法
#### 2.1 模型推广
将未观测到的硬币A序列推广为任意隐变量序列$Z = (z\_1, z\_2, \cdots z\_n)^T$, 其中$z$为隐变量. 目标是极大化观测序列$Y = (y\_1, y\_2, \cdots y\_n)^T$关于参数$\theta$的对数似然函数, 即极大化
$$
\begin{align}
L(\theta) &= log  P(Y | \theta) = log \sum\limits\_Z P(Y, Z | \theta) \\\
&= log \sum\limits\_Z  [ P(Y|Z,\theta) P(Z |\theta) ]
\end{align}
\tag{2.1}
$$
注意到式中需要对隐变量序列的所有可能求和, 但是隐变量往往是未知的, 且求和之后的函数难以处理, 因此直接极大化会遇到困难.

---

#### 2.2 Q函数的推导
假设使用迭代算法估计 $\theta$ , 其中第 $i$ 次迭代得到的参数估计值为$\theta\_i$, 使用其估计得到的似然函数必然小于 $L(\theta)$, 设法取其下界:

利用Jenson不等式, 对于譬如log一类的凹函数(concave function)有:
$$
f(\sum\limits\_i a\_i x\_i) \ge \sum\limits\_i a\_i f(x\_i) 
\text{ ,in which} \sum\limits\_i a\_i = 1
$$

故

$$
\begin{align}
L(\theta) - L(\theta\_i) &= log \sum\limits\_z  [ P(Y|Z,\theta) P(Z |\theta) ] - log  P(Y | \theta\_i) \\\
&= log \sum\limits\_Z P(Z|Y,\theta\_i ) \frac {P(Y|Z,\theta) P(Z |\theta)}{P(Z|Y,\theta\_i)} - log  P(Y | \theta\_i) \\\
&\ge \sum\limits\_Z P(Z|Y,\theta\_i ) log\frac {P(Y|Z,\theta) P(Z |\theta)}{P(Z|Y,\theta\_i)} - log  P(Y | \theta\_i) \\\
&= \sum\limits\_Z P(Z|Y,\theta\_i ) log \frac {P(Y|Z,\theta) P(Z |\theta)}{P(Z|Y,\theta\_i) P(Y|\theta\_i)}
\end{align}
$$
其中第二行推导利用了$\sum\limits\_Z P(Z|Y,\theta\_i ) = 1$ 的性质, 而第四行把$log  P(Y | \theta\_i)$ 化为 $\left\lbrace \sum\limits\_Z P(Z|Y,\theta\_i )\right\rbrace log  P(Y | \theta\_i)$ 以便和前一项合并.

由此可得 $L(\theta)$ 的下界:
$$
B(\theta, \theta\_i) = L(\theta\_i) + \sum\limits\_Z P(Z|Y,\theta\_i ) log \frac {P(Y|Z,\theta) P(Z |\theta)}{P(Z|Y,\theta\_i) P(Y|\theta\_i)}
\tag{2.2}
$$
易知 $B(\theta\_i, \theta\_i) = L(\theta\_i)$, 对任意$\theta\_{i+1}$, 若有$B(\theta\_{i+1}, \theta\_i) > B(\theta\_i, \theta\_i)$, 则:
$$
L(\theta\_{i+1}) \ge B(\theta\_{i+1}, \theta\_i) > B(\theta\_i, \theta\_i) = L(\theta\_i)
\tag{2.3}
$$
由(2.3)可知, 为了取尽量大的$L(\theta\_{i+1})$, 就需要取其尽量大的下界$B(\theta\_{i+1}, \theta\_i)$. 省略和 $\theta$ 无关的变量, 有:
$$
\begin{align}
\theta\_{i+1} &= arg\max\limits\_\theta B(\theta, \theta\_i) \\\
&= arg\max\limits\_\theta \sum\limits\_Z P(Z|Y,\theta\_i ) log  P(Y, Z | \theta) \\\
&= arg\max\limits\_\theta Q(\theta, \theta\_i)
\end{align}
$$ 
由此可知, EM算法是通过不断求解下界的极大化逼近求解对数似然函数极大值的算法, 但并不能求得全局极大值.

---

#### 2.3 Q函数的定义
$$
\begin{align}
Q(\theta, \theta\_i) &= E\_Z [log  P(Y, Z | \theta) | Y, \theta\_i]\\\
&= \sum\limits\_Z P(Z|Y,\theta\_i ) log  P(Y, Z | \theta)
\end{align}
\tag{2.4}
$$
完全观测数据的对数似然函数 $log  P(Y, Z | \theta)$ 在给定观测数据 $Y$ 和当前估计参数 $\theta\_i$ 下, 对隐变量序列 $Z$ 的条件概率分布 $P(Z|Y,\theta\_i )$ 的期望称为Q函数.

---

#### 2.4 EM算法
输入: 观测变量$Y$, 隐变量取值范围$Z$, 条件分布$P(Z|Y,\theta\_i )$, 联合分布 $P(Y, Z | \theta)$
输出: 模型参数$\theta$

(1) 初始化$\theta\_0$
(2) E步: 在第 $i$ 步计算出的参数为 $\theta\_i$, 第 $i+1$ 步, 计算
$$
\begin{align}
Q(\theta, \theta\_i) &= E\_Z [log  P(Y, Z | \theta) | Y, \theta\_i]\\\
&= \sum\limits\_Z P(Z|Y,\theta\_i ) log  P(Y, Z | \theta)
\end{align}
$$
(3) M步: 最大化$Q(\theta, \theta\_i)$, 得到 $\theta\_{i+1} = arg\max\limits\_\theta Q(\theta, \theta\_i)$
(4) 重复(2), (3)两步, 直到 $\theta$ 收敛

简而言之, EM算法的E步: 从观测序列 $Y$ 和估计参数 $\theta\_i$ 计算出隐变量序列的概率分布 $P(Z|Y,\theta\_i )$ ,对每个观测序列 $Z$, 求出 $log  P(Y, Z | \theta)$ 的期望, 此时这是个仅含有 $\theta$ 的函数.

M步: 最大化$Q(\theta, \theta\_i)$, 得到 $\theta\_{i+1} = arg\max\limits\_\theta Q(\theta, \theta\_i)$

---

### 3. 利用EM算法求解三硬币模型
计算需要的参数:
$Y = (1, 1, 0, 1, 0, 0, 1, 0, 1, 1)^T$
$Z = \lbrace 1, 0 \rbrace$
$\theta = (p\_a, p\_b, p\_c)$

#### 3.1 E步
对于单步实验:
$$
\begin{align}
P(z=1 | y\_j,\theta\_i ) &= \frac {P(y\_j, z=1 | \theta\_i)} {P(y\_j | \theta\_i)} \text{ (Bayes公式) } \\\
&= \frac {P(y\_j, z=1 | \theta\_i)} {P(y\_j, z=1 | \theta\_i) + P(y\_j, z=0 | \theta\_i)} \\\
&= \frac {p\_a p\_b^{y\_j} (1 - p\_b)^{1-y\_j}} {p\_a p\_b^{y\_j} (1 - p\_b)^{1-y\_j} + (1 - p\_a) p\_c^{y\_j} (1-p\_c)^{1 - y\_j}} \\\
&= \mu\_j^{(i)}
\end{align}
$$

$P(z=0 | y\_j,\theta\_i ) = 1 - P(z=1 | y\_j,\theta\_i ) = 1 - \mu\_j^{(i)}$

$$ 
\begin{align}
&Q(\theta, \theta\_i) \\\
&= E\_Z [log  P(Y, Z | \theta) | Y, \theta\_i] \\\
&= \sum\limits\_j E\_z [logP(y\_j, z|\theta) | y\_j, \theta\_i] \\\
&= \sum\limits\_j \sum\limits\_z P(z|y\_j,\theta\_i ) log  P(y\_i, z | \theta) \\\
&=  \sum\limits\_j [\mu\_j^{(i)} logP(y\_j, z\_j=1 | \theta) + (1 - \mu\_j^{(i)})logP(y\_j, z\_j=0 | \theta)] \\\
&= \sum\limits\_j \left\lbrace \mu\_j^{(i)} log [p\_a p\_b^{y\_j} (1 - p\_b)^{1-y\_j}] + (1 - \mu\_j^{(i)})log[(1 - p\_a) p\_c^{y\_j} (1-p\_c)^{1 - y\_j}] \right\rbrace
 \end{align}
$$
注: 对于以上推导的1-2步, 由于Q函数是概率的期望值, 对于单步重复试验, 整个序列的期望值可以由单步实验的期望值相加取得.

#### 3.2 M步

其中 $\mu\_j^{(i)}$ 中包含了第 $i$ 次迭代得到的估计参数, 最优化 $Q(\theta, \theta\_i)$ 之后得到新的参数, 即 $\theta\_{i+1}$ . 优化手段直接求导并令导数等于零(过程略):

$$
p\_a^{(i+1)} = \frac {\partial Q} {\partial p\_a} = \frac {1}{n} \sum\limits\_{j=1}^n \mu\_j^{(i)} \\\
p\_b^{(i+1)} = \frac {\partial Q} {\partial p\_b} = \frac{\sum\limits\_{j=1}^n \mu\_j^{(i)} y\_j} {\sum\limits\_{j=1}^n \mu\_j^{(i)}} \\\
p\_c^{(i+1)} = \frac {\partial Q} {\partial p\_c} = \frac{\sum\limits\_{j=1}^n(1 - \mu\_j^{(i)}) y\_j} {\sum\limits\_{j=1}^n (1 - \mu\_j^{(i)})} 
$$
依次使用以上三式迭代, 直到收敛即可

---

### 4. 小结

 EM算法是对极大似然估计的自然推广. 

 -  对于没有隐变量的参数估计, 直接使用最大似然估计即可. 
 -  对含有隐变量的参数估计: 
      1. 首先先验地估计一组参数$\theta\_i$
      2. 因为不知道隐变量的分布情况, 所以先根据$\theta\_i$, 使用Bayes公式求出隐变量的概率分布$P(z|y,\theta\_i )$
      3. 根据该分布求出完全数据的对数似然函数$log  P(y, z | \theta)$的期望, 该期望即为Q函数.
      4. 最大化Q函数, 等价于最大化似然函数的下界.
      5. 获得新的 $\theta\_{i+1}$ 并进行迭代, 相当于提升了似然函数的值

---

### 5. Reference
[What is the expectation maximization algorithm?](http://ai.stanford.edu/~chuongdo/papers/em\_tutorial.pdf)
[统计学习方法](https://www.amazon.cn/%E7%BB%9F%E8%AE%A1%E5%AD%A6%E4%B9%A0%E6%96%B9%E6%B3%95-%E6%9D%8E%E8%88%AA/dp/B007TSFMTA/ref=sr\_1\_1?ie=UTF8&qid=1466746855&sr=8-1&keywords=%E7%BB%9F%E8%AE%A1%E5%AD%A6%E4%B9%A0%E6%96%B9%E6%B3%95)

> Written with [StackEdit](https://stackedit.io/).
