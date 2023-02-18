# Boosting(3)- Gradient Boosting


梯度提升算法简介

<!--more-->

### Boosting on different Loss Functions

In the last section, we use forward stagewise on an addictive model to solve the optimization of adaboost. The exponential loss function is relatively easy to handle, but other loss functions may not. 

Recall from the last section, we have loss function like this:
$$ L(y\_i, f(x\_i)) $$

Consider optimizing an arbitary loss function using forward stagewise:

$$  (\beta\_m, \gamma\_m) = arg\min\limits\_{\beta, \gamma} \sum\limits\_{i=1}^N L(y\_i, f\_{m-1}(x\_i) + \beta b(x\_i;\gamma)) \tag{1}$$


Now since $\beta b(x\_i;\gamma)$ is relatively small compared to $f\_{m-1}(x\_i)$, using **Taylor Series** we could unfold it around $f\_{m-1}(x\_i)$:

$$ L \approx \frac{1}{N} \sum\limits\_{i=1}^N L(y\_i, f\_{m-1}(x\_i)) + \beta \sum\limits\_{i=1}^N 
\left. { \frac{\partial L(y\_i, s)}{\partial s} } \right |\_{s=f\_{m-1}(x\_i)}
 b(x\_i;\gamma) \tag{2}$$

Add regularzation to (2) to avoid $b(x\_i;\gamma)$ being too big, since we already have $\beta$ to adjust the term's magnitude:

$$\begin{align}
 L & \approx \frac{1}{N} \sum\limits\_{i=1}^N L(y\_i, f\_{m-1}(x\_i)) + \frac{\beta}{2} \sum\limits\_{i=1}^N 
\left. { 2 \frac{\partial L(y\_i, s)}{\partial s} } \right |\_{s=f\_{m-1}(x\_i)} b(x\_i;\gamma) + b^2(x\_i;\gamma) \\\
\text{(Strip the constants)} & = \beta \sum\limits\_{i=1}^N 2 \frac{\partial L}{\partial s} b(x\_i;\gamma) + b^2(x\_i;\gamma) \\\
& = \beta \sum\limits\_{i=1}^N (b(x\_i;\gamma) + \frac{\partial L}{\partial s})^2 - (\frac{\partial L}{\partial s} )^2
\end{align}
\tag{3}$$

---

### Gradient Boosting

Now we could minimize the loss function using the last section's results:

1. Optimize $b(x\_i;\gamma)$. From (3) we get:
 $$\gamma\_m = arg\min\limits\_\gamma \beta \sum\limits\_{i=1}^N \left(b(x\_i;\gamma) + \left. { \frac{\partial L(y\_i, s)}{\partial s} } \right |\_{s=f\_{m-1}(x\_i)} \right)^2$$
 That is, just train the base model $b(x\_i;\gamma\_m)$ with the loss function's gradient $-\frac{\partial L(y\_i, s)}{\partial s}$ in every step $m$, that's why it calld gradient boosting:
 $$ \text{fit } b(x\_i;\gamma\_m) = - \left. { \frac{\partial L(y\_i, s)}{\partial s} } \right |\_{s=f\_{m-1}(x\_i)}$$

2. Optimize $\beta$
 $$\beta\_m = arg\min\limits\_\beta \sum\limits\_{i=1}^N L(y\_i, f\_{m-1}(x\_i) + \beta b(x\_i;\gamma\_m))$$
 Since we already have $y\_i$, $f\_{m-1}(x\_i)$ and $b(x\_i;\gamma\_m)$, this is just a one-dimensional optimization problem, which is easy to handle.

The full algorithm is similar, see the reference of wiki. The base model used most frequently is decision trees.

---

Reference:
[统计学习方法](https://www.amazon.cn/%E7%BB%9F%E8%AE%A1%E5%AD%A6%E4%B9%A0%E6%96%B9%E6%B3%95-%E6%9D%8E%E8%88%AA/dp/B007TSFMTA/ref=sr\_1\_1?ie=UTF8&qid=1466746855&sr=8-1&keywords=%E7%BB%9F%E8%AE%A1%E5%AD%A6%E4%B9%A0%E6%96%B9%E6%B3%95)
[机器学习技法](https://www.AndrewNg's ML.org/course/ntumltwo)
[Wikipedia](https://www.wikiwand.com/en/Gradient\_boosting)
