# Machine Learning Notes(5)-Programming Exercise 1


Programming Exercise 1: Gradient Descent for Linear regression

<!--more-->

## Gradient Descent for Linear regression

**Notification: This is a simplified code example, if you are attempting this class, don't copy & submit since it won't even work...**

## Step 1 - Load & Initialize Data

```matlab
data = load('ex1data1.txt');
X = data(:, 1); y = data(:, 2);
X = [ones(m, 1), data(:,1)];
theta = zeros(2, 1);

iterations = 1500;
alpha = 0.01;
```

## Step 2 - Feature Normalize
This is done before adding a column to X, which representing $x_0$
```matlab
function [X_norm, mu, sigma] = featureNormalize(X)

    X_norm = X;
    mu = zeros(1, size(X, 2));
    sigma = zeros(1, size(X, 2));

    mu = mean(X);
    sigma = std(X);

    X_norm = (X .- mu)./sigma

end
```

## Step 3 - Gradient Descent

```matlab
function [theta, J_history] = gradientDescent(X, y, theta, alpha, num_iters)

    m = length(y);

    for iter = 1:num_iters
        summary = X' * (X*theta .- y);
        theta = theta .- alpha * (1/m) * summary;
    end
end
```
How gradient works:
$$
\mathbf{X}  \mathbf{\theta} - \mathbf{y}
 = \begin{bmatrix}\mathbf{x\_1} & \mathbf{x\_2} & \cdots & \mathbf{x\_m} \end{bmatrix}  \begin{bmatrix}\theta\_1 \\\ \theta\_2 \\\ \vdots \\\ \theta\_n \end{bmatrix} - \begin{bmatrix}y\_1 \\\ y\_2 \\\ \vdots \\\ y\_n \end{bmatrix}
 = \begin{bmatrix}h\_\theta(\mathbf{x}^{(1)}) - y^{(1)} \\\ h\_\theta(\mathbf{x}^{(2)}) - y^{(2)} \\\ \vdots \\\ h\_\theta(\mathbf{x}^{(m)}) - y^{(m)} \end{bmatrix}$$
$$
\begin{align}
(\mathbf{X}^T) \* (\mathbf{X}  \mathbf{\theta} - \mathbf{y}) \\\
& = \begin{bmatrix}\mathbf{x\_1} \\\ \mathbf{x\_2} \\\ \cdots \\\ \mathbf{x\_m} \end{bmatrix} \* 
   \begin{bmatrix}h\_\theta(\mathbf{x}^{(1)}) - y^{(1)} \\\ h\_\theta(\mathbf{x}^{(2)}) - y^{(2)} \\\ \vdots \\\ h\_\theta(\mathbf{x}^{(m)}) - y^{(m)} \end{bmatrix}
& = \begin{bmatrix} \sum\limits\_{i=1}^{m}((h\_\theta(\mathbf{x}^{(i)}) - y^{(i)}) x\_1^{(i)}) \\\ \sum\limits\_{i=1}^{m}((h\_\theta(\mathbf{x}^{(i)}) - y^{(i)}) x\_2^{(i)}) \\\ \cdots \\\ \sum\limits\_{i=1}^{m}((h\_\theta(\mathbf{x}^{(i)}) - y^{(i)}) x\_i^{(i)}) \end{bmatrix}
\end{align}
$$

