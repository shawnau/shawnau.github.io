# 梯度提升(Gradient Boosting)


自己写的老物了, 整理一下发出来, 可能会修改

<!--more-->

![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/50782598_p0.png)

### 1. 任意损失函数的Boosting
损失函数的一般表示是:
$$ L(y\_i, f(x\_i)) $$

考虑使用[前向分步算法]({{< ref "/posts/2016-06-22-forward-stagewise-algorithm" >}})求解一个任意损失函数:

$$ (\beta\_m, \gamma\_m) = arg\min\limits\_{\beta, \gamma} \sum\limits\_{i=1}^N L(y\_i, f\_{m-1}(x\_i) + \beta b(x\_i;\gamma)) \tag{4.1}$$

既然 $\beta b(x\_i;\gamma)$ 和 $f\_{m-1}(x\_i)$ 相比是等价无穷小量, 使用 **泰勒级数** 在 $f\_{m-1}(x\_i)$ 附近展开:

$$ L \approx \frac{1}{N} \sum\limits\_{i=1}^N L(y\_i, f\_{m-1}(x\_i)) + \beta \sum\limits\_{i=1}^N \left. { \frac{\partial L(y\_i, s)}{\partial s} } \right |\_{s=f\_{m-1}(x\_i)} b(x\_i;\gamma) \tag{4.2}$$

为 (2) 添加正则化项防止 $b(x\_i;\gamma)$ 变得太大, 既然我们已经有了 $\beta$ 去调整这个项的大小了:

$$\begin{aligned} L & \approx \frac{1}{N} \sum\limits\_{i=1}^N L(y\_i, f\_{m-1}(x\_i)) + \frac{\beta}{2} \sum\limits\_{i=1}^N \left. { 2 \frac{\partial L(y\_i, s)}{\partial s} } \right |\_{s=f\_{m-1}(x\_i)} b(x\_i;\gamma) + b^2(x\_i;\gamma) \\\ \text{(Strip the constants)} & = \beta \sum\limits\_{i=1}^N 2 \frac{\partial L}{\partial s} b(x\_i;\gamma) + b^2(x\_i;\gamma) \\\ & = \beta \sum\limits\_{i=1}^N (b(x\_i;\gamma) + \frac{\partial L}{\partial s})^2 - (\frac{\partial L}{\partial s} )^2 \end{aligned}\tag{4.3}$$

> 参考了林轩田的机器学习技法MOOC

### 2. Gradient Boosting
现在可以最小化损失函数:

1. 求解 $b(x\_i;\gamma)$.
从 (3) 可知: $$\gamma\_m = arg\min\limits\_\gamma \beta \sum\limits\_{i=1}^N \left(b(x\_i;\gamma) + \left. { \frac{\partial L(y\_i, s)}{\partial s} } \right |\_{s=f\_{m-1}(x\_i)} \right)^2$$

 也就是在每一步 $m$ 中, 利用损失函数的梯度 $-\frac{\partial L(y\_i, s)}{\partial s}$ 训练基分类器 $b(x\_i;\gamma\_m)$. 这就是为什么它被称为梯度提升算法

 $$ \text{fit } b(x\_i;\gamma\_m) = - \left. { \frac{\partial L(y\_i, s)}{\partial s} } \right |\_{s=f\_{m-1}(x\_i)}$$
2. 求解 $\beta$ $$\beta\_m = arg\min\limits\_\beta \sum\limits\_{i=1}^N L(y\_i, f\_{m-1}(x\_i) + \beta b(x\_i;\gamma\_m))$$ 既然我们已经有了 $y\_i$, $f\_{m-1}(x\_i)$ 和 $b(x\_i;\gamma\_m)$, 那原问题就变成了一个简单的一维变量最优化问题, 那就很容易解决了整个算法的思想很简单, 最常用的基分类器是决策树, 称为gradient boosting decision tree (GBDT)

### 3. GBDT回归算法
基分类器是CART回归树

 - 输入: 训练集${\displaystyle \{(x\_{i},y\_{i})\}\_{i=1}^{n},}$可导损失函数${\displaystyle L(y,F(x)),}$基分类器个数/迭代个数$M$
 - 输出: 集成回归器$ F\_M(x) $
 1. 初始化 $$ F\_0(x) = \underset{\gamma}{\arg\min} \sum\_{i=1}^n L(y\_i, \gamma) $$
 2. 对m个基分类器/回归器 1. 计算损失函数对每个样本的一阶导数近似: $$ r\_{im} = -\left[\frac{\partial L(y\_i, F(x\_i))}{\partial F(x\_i)}\right]\_{F(x)=F\_{m-1}(x)} \quad {i=1, \ldots, n}$$
 3. 使用$\{(x\_i, r\_{im})\}\_{i=1}^n$训练下一个回归树 $${\displaystyle h\_{m}(x)=\sum \_{j=1}^{J\_{m}}c\_{jm}I(x\in R\_{jm}),}$$
 4. 用一维线性搜索最优化基分类器在每个区间的输出$$ c\_{mj} = \underbrace{arg\; min}\_{c}\sum\limits\_{x\_i \in R\_{tj}} L(y\_i,f\_{m-1}(x\_i) +c) $$ 其中$R\_{tj}, j =1,2,..., J$, $J$为叶子节点的数量
 5. 更新强学习器 $$ F\_{M}(x) = f\_{m-1}(x) + \sum\limits\_{j=1}^{J}c\_{mj}I(x \in R\_{tj}) $$

### 4. GBDT分类算法
1. 二分类算法
 定义预测模型 $$ p(x)=\frac{e^{F(x)}}{e^{F(x)}+e^{-F(x)}} $$ $$ F(x)=\frac{1}{2}log(\frac{p(x)}{1-p(x)}) $$
 预测时将$F(x)$转换成$p(x)$, 然后比较大小. 在此之上定义对数损失函数(MLE损失)
2. 多分类算法
 类似于Softmax $$ p\_k(x) = \frac{\exp(f\_k(x))} { \sum\limits\_{l=1}^{K} exp(f\_l(x))} $$ $$ F\_k(x) = log(p\_k(x)) - \frac{1}{K} \sum\limits\_{i=1}^K log(p\_i(x))$$ 损失函数为: $$ L(y, f(x)) = - \sum\limits\_{k=1}^{K}y\_klog\;p\_k(x) $$ 集合上两式，我们可以计算出第t轮的第i个样本对应类别k的负梯度误差为: $$ r\_{tik} = -\bigg[\frac{\partial L(y, f(x\_i)))}{\partial f(x\_i)}\bigg]\_{f\_k(x) = f\_{k, t-1}\;\; (x)} = y\_{il} - p\_{k, t-1}(x\_i) $$
过程:
 1. 初始化先验分布为均匀分布$ F\_k(x) = 1/K $
 2. 对 $t = 1, 2, \cdots, T$ a. 对每个label, 计算$p\_k(x)$ b. 对每个label, 计算损失函数$f\_{l, t}$在每个样本$i$处的一阶导数 c. 分别拟合一阶导数, 更新$F\_k(x)$

### 5. 正则化
1. Shrinkage
$$ f\_{k}(x) = f\_{k-1}(x) + \nu h\_k(x) $$采用步长$0< \upsilon <1$降低每个模型的贡献, 往往伴随着$M$的增加, 即缩减每个基分类器的贡献, 同时使用更多的基分类器, 降低过拟合
2. Stochastic gradient boosting 类似于Bagging的手段, 每棵树生成的时候都使用一部分训练数据
3. Number of observations in leaves 在停止条件里加入叶节点的样本数量, 小于等于就停止
4. Penalize Complexity of Tree

### 6. 一些其他内容
1. DART(Dropout Addictive Regression Tree): 为了解决Boosting过程中, 首先产生的树拥有更大的贡献, 采取类似于Dropout的方法: 每次新加一棵树，这棵树要拟合的并不是之前全部树ensemble后的残差，而是随机抽取的一些树ensemble；同时新加的树结果要规范化一下.

## XGBoost
两篇文章已经介绍得很好了:

1. [关于特征选择: GBDT算法原理深入解析](https://www.zybuluo.com/yxd/note/611571#gbdt算法原理深入解析)
2. [关于特征重要度: Tree ensemble算法的特征重要度计算](https://www.zybuluo.com/yxd/note/614495)

一句话, 在子树中, 某一个特征的重要度就是所有使用该特征分裂的节点的损失的减少值, 亦即:$$ Gain= - \frac12 \left[ \frac{(G\_L+G\_R)^2}{H\_L+H\_R+\lambda} - \frac{G\_L^2}{H\_L+\lambda} - \frac{G\_R^2}{H\_R+\lambda} \right] - \gamma $$对某个特征求gain之和, 再对所有基分类器求算术平均值即可得到特征重要度.

> [梯度提升树(GBDT)原理小结](http://www.cnblogs.com/pinard/p/6140514.html)

