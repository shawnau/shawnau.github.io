# 吴恩达 ML 公开课笔记(2) - A Linear Regression Example


This is my notes for the open course [Machine Learning](https://www.AndrewNg's ML.org/learn/machine-learning/) from AndrewNg's ML.

<!--more-->

## Model and Cost Function - A Linear Regression Example
1. Hypothesis: $$h\_\theta(x) = \theta\_0 + {\theta\_1}x$$
2. Cost function: $$J(\theta\_0, \theta\_1) = \frac {1}{2m} \sum\_{i=1}^m (h\_\theta (x^{(i)}) - y^{(i)} )^2$$
 - Basically this function is derived from the maximum likelihood estimation of a set of $\theta\_a, \theta\_b \sim N(0, \sigma^2)$
 - $\exists J(\theta\_a, \theta\_b) (J(\theta\_a, \theta\_b) = min \lbrace J(\theta\_0, \theta\_1) \rbrace ) \to \theta\_a, \theta\_b$ is the best fit of hypothesis

## Parameter Learning - Minimize Cost Function
1. Gradient descent algorithm 
 - repeat until convergence:$$\theta\_j := \theta\_j - \alpha \frac{\partial}{\partial \theta\_j} J(\theta\_0, \theta\_1)$$
2. Notifications
 - $\alpha$: learning rate. The larger $\alpha$ is, the faster the algorithm will be(but rougher, or even fail to convergence)
 - $\theta\_0, \theta\_1$ should be updated simultaneously(using multiple temp var should work!)
 - Gradient descent can converge to a local minimum even with the learning rate fixed, as the step will automatically become smaller

## Gradient Descent For Linear Regression
$$\begin{aligned} \text{repeat until convergence: } \lbrace & \\\ \theta\_0 := & \theta\_0 - \alpha \frac{1}{m} \sum\limits\_{i=1}^{m}(h\_\theta(x^{(i)}) - y^{(i)}) \\\ \theta\_1 := & \theta\_1 - \alpha \frac{1}{m} \sum\limits\_{i=1}^{m}\left((h\_\theta(x^{(i)}) - y^{(i)}) x^{(i)}\right) \\ \rbrace&\end{aligned}$$
 - The $J(\theta\_0, \theta\_1)$ is a convex function, which means it has only one global minimun, which means gradient descent will always hit the best fit
 - "Batch" Gradient Descent: "Batch" means the algo is trained from all the samples every time

