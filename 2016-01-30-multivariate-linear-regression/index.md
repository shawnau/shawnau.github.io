# Machine Learning Notes(3)-Multivariate Linear Regression


Multivariate Linear Regression

<!--more-->

## Gradient Descent for Multiple Variables
1. Suppose we have n variables, set hypothesis to be:
$$h\_\theta(\mathbf{x}) = \sum\_{i=0}^n \theta\_i x\_i = \theta^T \mathbf{x}, x\_0 =1$$
in which $\mathbf{x} = \begin{bmatrix}x\_1 \\\ x\_2 \\\ \vdots \\\ x\_n \end{bmatrix}$, $\mathbf{\theta} = \begin{bmatrix}\theta\_1 \\\ \theta\_2 \\\ \vdots \\\ \theta\_n \end{bmatrix}$

2. Cost Function
$$J(\theta^T) = \frac {1}{2m} \sum\_{i=1}^m (h\_\theta (\mathbf{x}^{(i)}) - y^{(i)} )^2$$
3. Gredient Descent Algorithm
$$\begin{aligned} \text{repeat until convergence: } \lbrace & \\\ \theta\_j := & \theta\_j - \alpha \frac{1}{m} \sum\limits\_{i=1}^{m}\left((h\_\theta(\mathbf{x}^{(i)}) - y^{(i)}) x_j^{(i)}\right) \\ \rbrace&\end{aligned}$$
4. Feature Scalling
 - Get every feature into approximately [-1, 1]. Just normalize all the parameters :)
 - $x_i := \frac {xi-\mu}{\sigma}$
 - Do not apply to $x_0$
5. Learning Rate
 -  Not too big(fail to converge), not too small(too slow)
6. Polynormal Regression
 - Use feature scalling. (Somewhat like normalizing dimension)

## Normal Equation
1. Set $\mathbf{X} = \begin{bmatrix}\mathbf{x\_1} & \mathbf{x\_2} & \cdots & \mathbf{x\_m} \end{bmatrix}$, $\mathbf{y^T} = \begin{bmatrix} y\_1 & y\_2 & \cdots & y\_m \end{bmatrix}$ as our training data:
 - If $\exists \mathbf{\theta\_h} \forall \mathbf{x\_i} \in \mathbf{X} (\theta\_h^T \mathbf{x} = y^{(i)})$, then $\mathbf{\theta\_h}$ should be our perfect fit of the hypothesis. However, in most cases this fit doesn't exist, so we should find a set of $\mathbf{\theta}$ as close as the best fit.
 - The question above equals solving this set of linear equations: $\mathbf{X^T} \mathbf{\theta} = \mathbf{y}$, while in most cases there's no solution. Instead we try to find a $\mathbf{\theta}$ let $\mathbf{X^T} \mathbf{\theta}$ as close as $\mathbf{y}$.
 - Let $\Delta = |\mathbf{y} - \mathbf{X^T} \mathbf{\theta}|$, we need this $\Delta$ to be as small as possible. Use a little imagination we could get the right answer: In this case, the right $\mathbf{\theta}$ would let $\mathbf{X^T} \mathbf{\theta}$ be the projection of $\mathbf{y}$ to the column space of $\mathbf{X^T}$, which means $\Delta$ vertical to the column space of $\mathbf{X^T}$, which means:
$$\mathbf{X} \Delta = \mathbf{0}$$
so $\mathbf{X} (\mathbf{y} - \mathbf{X^T} \mathbf{\theta}) = \mathbf{0}$, so $\mathbf{\theta} = {(\mathbf{X}\mathbf{X^T})}^{-1} \mathbf{X}\mathbf{y}$
 - Notice our $\mathbf{X}$ is a little different from the course video, in which the $x\_i$ are arranged parallel while our $x\_i$ are arranged vertically.
 - Feature scalling is not necessary here.
2. Gradient Descent vs Normal Equation
 - Just know Gradient Descent works well even when n is very large, while Normal Equation gets slow since it needs a lot calculation solving ${(\mathbf{X}\mathbf{X^T})}^{-1}$




