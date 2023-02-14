# Hidden Markov Model


![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/4099531_p0_master1200.jpg)
使用HMM的赌徒, 你还好吗? hmmm....

<!--more-->

## 0. 定义

![HMM](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/HMM.png)

如图所示, 线性HMM由状态$V$和观察$Q$的序列$I$和$O$组成,  其中$I$是马尔科夫链(Markov Chain). 状态之间通过转移矩阵$A$(transition)决定状态转移的概率分布, 每一时间步的状态通过观察矩阵$B$(emission)决定观察结果的概率分布.而状态一般是未知的.


### 0.0 符号表

|符号|含义|备注|
|:-|-|:-:|
|$Q = \lbrace q\_1, q\_2, \cdots, q\_{N\_Q} \rbrace$|状态(state)||
|$I = (I\_1, I\_2, \cdots, I\_T)$|状态序列|$I\_t \in Q$|
|$V = \lbrace v\_1, v\_2, \cdots, v\_{N\_V} \rbrace$|观察(observation)||
|$O = (o\_1, o\_2, \cdots, o\_T )$|观察序列|$o\_t \in V$|
|$A\_{ij} \in R^{N\_Q \times N\_Q}$|状态转移矩阵|$P(I\_t = q\_j \lvert I\_{t-1} =  q\_i)$|
|$B\_{q\_i}(v\_j ) \in R^{N\_Q \times N\_V}$|观测概率矩阵(emission)|$P(o\_t = v\_j \lvert I\_t =  q\_i)$|
|$\pi \in R^{N\_Q}$|初始状态分布|$P(I\_1 = q\_i)$|
|$\lambda=(\pi, A, B)$|模型参数|HMM的参数由$\lambda$组成|


 - 状态转移矩阵代表了从状态$q\_i$转移到$q\_j$的概率
 - 观测概率矩阵代表了从状态$q\_i$观察到输出$v\_j$的概率


### 0.1 基本假设

1. **一阶齐次Markov假设**: 任意时刻的状态$I\_t$只依赖于前一时刻的状态$I\_{t-1}$
2. 任意时刻的观察$O\_t$只依赖于该时刻的状态$I\_t$

### 0.2 基本问题

HMM作为一个生成模型, 也涉及**概率计算**, **学习问题**和**预测问题**(也叫decoding)

1. 概率计算: 已知$\lambda$, 求$P(O|\lambda)$
2. 学习问题: 已知$O$, 求$arg\max\limits\_{\lambda}P(O|\lambda)$
3. 预测问题: 已知$O, \lambda$, 求$arg\max\limits\_{I}P(I|O, \lambda)$

## 1. 概率计算
已知$\lambda=(A, B, \pi)$, 计算观测序列$O$出现的概率$P(O|\lambda)$.

### 1.1 直接计算
在已知$O$的情况下, 需要知道所有可能的状态序列$I$, 对所有状态序列与观察结果的联合概率分布求和.

1. 状态序列$(I\_1, I\_2, \cdots, I\_T)$的概率为
 $$
 P(I) = \pi\_{I\_1} \prod\_{t=2}^{T} A\_{I\_{t-1} I\_t}
 \tag{1.1.1}
 $$
2. 状态序列为$I$, 观测序列为$O$的概率为
 $$
 \begin{align}
 P(O, I) &= P(O | I)P(I) \\\
 &= \pi\_{I\_1}B\_{I\_1}(o\_1) \prod\_{t=2}^{T} A\_{I\_{t-1} I\_t}B\_{I\_t}(o\_t)
 \end{align}
 \tag{1.1.2}
 $$
3. 对所有可能的$I$求和, 时间复杂度为$O(N\_V^T)$, 每次计算时间复杂度为$O(T)$, 总复杂度为$O(TN\_V^T)$
 $$
 P(O) = \sum\_I P(O, I)
 $$

### 1.2 前向算法
对所有的$I$求和, 由于每一步$I$都可能属于$Q$中的任何一个, 相当于在以下$N \times T$矩阵中寻找从左边到右边的所有路径的概率和.
$$
\begin{align}
&\begin{bmatrix}
o\_1 & o\_2 & \cdots & o\_T\\\
\end{bmatrix} \\\
&\begin{bmatrix}
q\_1 & q\_1 & \cdots & q\_1 \\\
q\_2 & q\_2 & \cdots & q\_2 \\\
\vdots & \vdots & \ddots & \vdots \\\
q\_N & q\_N & \cdots & q\_N \\\
\end{bmatrix} 
\end{align}
\tag{1.2.1}
$$

![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/7.PNG)
(示意图, 侵删)

对所有的路径求和涉及到大量的重复运算. 假设已经计算出了$(I\_1, I\_2, \cdots, I\_t=q\_t)$的所有可能的路径的概率之和, 那么在计算$(I\_1, I\_2, \cdots, I\_{t+1})$时, 所有经过节点$I\_t=q\_t$的路径即可复用之前的计算结果. 写成递推式即为:
$$
\alpha\_{t+1}(q\_i) = \sum\_{j=1}^N \alpha\_t(q\_j) A\_{ji}
\tag{1.2.2}
$$
输出是已知的, 可以求出给定输出的概率为:
$$
\alpha\_{t+1}(q\_i) = B\_{q\_i}(o\_{t+1})\sum\_{j=1}^N \alpha\_t(q\_j) A\_{ji}
\tag{1.2.3}
$$
其中
$$\alpha\_{t}(q\_i) = P((o\_1, o\_2, \cdots, o\_t), I\_t=q\_i)\tag{1.2.4}$$
使用此式递推, 时间复杂度为$O(TN^2)$

### 1.3 后向算法
类似地定义后向算法. 假设已经计算出了$(I\_t=q\_i, I\_{t+1}, \cdots, I\_T)$的所有可能的路径的概率之和, 那么在计算$(I\_{t-1}, I\_t, \cdots, I\_T)$时, 所有经过节点$I\_t=q\_t$的路径即可复用之前的计算结果. 
$$
\beta\_{t-1}(q\_i) = \sum\_{j=1}^N A\_{ij} \beta\_{t}(q\_j) B\_{q\_j}(o\_{t})
\tag{1.3.1}
$$
其中
$$\beta\_{t}(q\_i) = P((o\_{t+1}, o\_{t+2}, \cdots, o\_T), I\_t=q\_i)\tag{1.3.2}$$

### 1.4 小结
结合前向和后向算法的表达式, 可以发现
$$
P(O, I\_t=q\_i) = \alpha\_{t}(q\_i) \beta\_{t}(q\_i)
\tag{1.4.1}
$$
给定输出$O$, 状态$I\_t=q\_i$的概率可以利用全概率公式得到:
$$
\begin{align}
 \gamma\_t(q\_i)=P(I\_t=q\_i|O) &= \frac{P(O, I\_t=q\_i)}{P(O)} \\\
&= \frac{P(O, I\_t=q\_i)}{\sum\_j P(O, I\_t=q\_j)} \\\
&= \frac{\alpha\_{t}(I\_t=q\_i) \beta\_{t}(I\_t=q\_i)}{\sum\_j \alpha\_{t}(I\_t=q\_j) \beta\_{t}(I\_t=q\_j)}
\end{align}
\tag{1.4.2}
$$
类似方法可以求出给定输出$O$, 状态$I\_t=q\_i$, $I\_{t+1}=q\_j$的概率:
$$
\begin{align}
\xi\_t(q\_i, q\_j) &= P(I\_t=q\_i, I\_{t+1}=q\_j|O) \\\
&=\frac{P(I\_t=q\_i, I\_{t+1}=q\_j, O)}{P(O)} \\\
&=\frac{P(I\_t=q\_i, I\_{t+1}=q\_j, O)}{\sum\_i \sum\_j P(I\_t=q\_i, I\_{t+1}=q\_j, O)} 
\end{align}
\tag{1.4.3}
$$
其中$P(I\_t=q\_i, I\_{t+1}=q\_j, O)=\alpha\_t(q\_i)A\_{ij}B\_{q\_j}(o\_{t+1})\beta\_{t+1}(q\_j)$
$(1.4.1)$和$(1.4.3)$可以被用于化简学习问题的求解

## 2. 学习问题
### 2.1 Supervised: MLE
已知$I, O$, 求$A, B$的MLE, 直接用频数计算
$$
A\_{ij} = \frac{\sum\limits\_{t=1}^{T-1} I(I\_t = q\_i, I\_{t+1} = q\_j)}{\sum\limits\_{t=1}^{T-1} \sum\limits\_{j=1}^{N\_Q} I(I\_t = q\_i, I\_{t+1} = q\_j)}
\tag{2.1.1}
$$

$$
B\_{q\_i}(v\_j) = \frac{\sum\limits\_{t=1}^{T-1} I(I\_t = q\_i, o\_t = v\_j)}{\sum\limits\_{t=1}^{T-1} \sum\limits\_{j=1}^{N\_V} I(I\_t = q\_i, o\_t = v\_j)}
\tag{2.1.2}
$$

### 2.2 Non-Supervised: Baum-Welch

只知道$O$, 不知道$I$, 求$A, B$的MLE, 采用EM算法

1. E步: 给定参数先验$\overline \lambda$, 计算Q函数
 $$Q(\lambda, \overline \lambda) = \sum\limits\_I  P(I| O, \overline \lambda) logP(I, O | \lambda) \tag{2.2.1}$$
2. M步: 求$Q(\lambda, \overline\lambda)$对$\lambda$的最大值
 $$ \max\limits\_{\lambda} \sum\limits\_I  P(I| O, \overline \lambda) logP(I, O | \lambda) \tag{2.2.2}$$
3. $\overline \lambda = \lambda$, 重复1和2.

由于
$$
 P(I | O, \overline \lambda) = \frac{P(I, O| \overline \lambda)}{P(O | \overline \lambda)}
 \tag{2.2.3}
$$
而优化目标是最大化$P(I, O | \lambda)$对$I$的期望,和$P(O | \overline \lambda)$无关 故可以用$P(I, O| \overline \lambda)$替代

其中
$$
P(I, O | \lambda) = \pi\_{I\_1} B\_{I\_1}(o\_1)\prod\limits\_{t=1}^{T-1} A\_{I\_{t}I\_{t+1}}B\_{I\_{t+1}}(o\_{t+1}) \tag{2.2.4}
$$
将$(2.2.4)$带入$(2.2.2)$, 使用拉格朗日乘子法可以求出$\lambda$

$$
\pi\_{q\_i} = \gamma\_1(q\_i) \\\
A\_{ij} = \frac{\sum\limits\_{t=1}^{T-1}\xi\_t(q\_i, q\_j)}{\sum\limits\_{t=1}^{T-1} \gamma\_t(q\_i)} \\\
B\_{q\_i}(v\_j) = \frac{\sum\limits\_{t=1}^{T-1} [\gamma\_t(q\_i) I(o\_t=v\_j)]}{\sum\limits\_{t=1}^{T-1} \gamma\_t(q\_i)}
\tag{2.2.5}
$$

## 3. Decode
已知$\lambda=(A, B, \pi), O=(o\_1, o\_2, \cdots, o\_T)$, 求使得$P(I|O,\lambda)$最大的状态序列$I$. 由于
 $$P(I | O, \lambda) = \frac{P(I, O| \lambda)}{P(O | \lambda)}$$
而$P(O | \lambda)$与状态序列$I$无关, 因此该问题等价于
$$ 
\begin{align}
\max\limits\_{I} P(I|O, \lambda) & \Leftrightarrow \max\limits\_{I} P(I, O| \lambda) \\\
& \Leftrightarrow \max\limits\_{I} \prod\limits\_{t=0}^{T-1} A\_{I\_{t}I\_{t+1}}B\_{I\_{t+1}}(o\_{t+1})
\end{align}
\tag{3.1}
$$
其中为了简化计算, 令$A\_{I\_0I\_1} = \pi\_{I\_1}$.

由此可见, 问题转化为在$(1.2.1)$中找到一条路径使得$(4.1)$最大. 可以使用类似于Dijkstra算法的动态规划, 只不过这里加权路径使用的是累乘计算. 该算法被称为Viterbi算法. 由于图是层次的, 因此比Dijkstra算法简单
### 3.1 Viterbi算法
1. 初始化
 $$
 \delta\_1(q\_i) = \pi\_{q\_i} \\\
 \phi\_1(q\_i) = 0
 \tag{3.1.1}
 $$
2. 递推. 对$t=[2, T]$:
 $$
 \delta\_t(q\_i) = \max\limits\_{1 \leq j \leq N\_Q} [\delta\_{t-1}(q\_j)A\_{ji}] B\_{q\_i}(o\_t) \\\
 \phi\_t(q\_i) = arg\max\limits\_{1 \leq j \leq N\_Q} [\delta\_{t-1}(q\_j)A\_{ji}]
 \tag{3.1.2}
 $$
3. 终止
 $$
 P = \max\limits\_{1 \leq i \leq N\_Q} \delta\_T(q\_i) \\\
 I\_T = arg\max\limits\_{1 \leq i \leq N\_Q} \delta\_T(q\_i)
 \tag{3.1.3}
 $$
4. 逆推最优状态序列. 对$t=T-1, T-2, \cdots, 1$
 $$
 I\_t = \phi\_{t+1}(I\_{t+1})
 \tag{3.1.4}
 $$

其中
$$
\delta\_t(q\_i)=\max\limits\_{I\_i \cdots I\_{t-1}} P((I\_1, I\_2, \cdots, I\_t=q\_i), O | \lambda)
\tag{3.1.5}
$$
$\delta\_t(q\_i)$代表输出为$O$且以$q\_i$结尾长度为$t$的任意状态序列的最大联合概率
$\phi\_t(q\_i)$代表$(3.1.5)$时的$I\_{t-1}$, 顺着逆推即可得到概率最大路径

## 4. Python 实现
> to be continued....
