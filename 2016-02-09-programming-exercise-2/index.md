# Machine Learning Notes(7)-Programming Exercise 2


Programming Exercise 2

<!--more-->

**Notification: This is a simplified code example, if you are attempting this class, don't copy & submit since it won't even work...**

## Plot function
 ```matlab
function plotData(X, y)

figure; hold on;
pos = find(y==1); neg = find(y == 0);
 
plot(X(pos, 1), X(pos, 2), 'k+','LineWidth', 2, ...
'MarkerSize', 7);
plot(X(neg, 1), X(neg, 2), 'ko', 'MarkerFaceColor', 'y', ...
'MarkerSize', 7);
 
hold off;
 
end
 ```
## Sigmiod function
 ```matlab
function g = sigmoid(z)

g = zeros(size(z));

% Compute the sigmoid of each value of z (z can be a matrix,
%               vector or scalar).
for row_iter = 1:size(z,1)
    for col_iter = 1:size(z,2)
        g(row_iter, col_iter) = 1/(1+e^(-z(row_iter,col_iter)));
    end;
end

end
 ```
## Cost function & gradient
```matlab
function [J, grad] = costFunction(theta, X, y)

m = length(y); 

J = (1/m)*sum(-y .* log(sigmoid(X*theta)) - (ones(m,1) - y) .* log(ones(m,1) - sigmoid(X*theta)));
grad = (X')*(sigmoid(X*theta) - y)*(1/m);

```
How gradient works:
$$ \begin{aligned} (\mathbf{X}^T) \* (\mathbf{X}  \mathbf{\theta} - \mathbf{y}) & = \begin{bmatrix}\mathbf{x\_1} \\\ \mathbf{x\_2} \\\ \cdots \\\ \mathbf{x\_m} \end{bmatrix} \* \begin{bmatrix}h\_\theta(\mathbf{x}^{(1)}) - y^{(1)} \\\ h\_\theta(\mathbf{x}^{(2)}) - y^{(2)} \\\ \vdots \\\ h\_\theta(\mathbf{x}^{(m)}) - y^{(m)} \end{bmatrix} \\\ & = \begin{bmatrix} \sum\limits\_{i=1}^{m}((h\_\theta(\mathbf{x}^{(i)}) - y^{(i)}) x\_1^{(i)}) \\\ \sum\limits\_{i=1}^{m}((h\_\theta(\mathbf{x}^{(i)}) - y^{(i)}) x\_2^{(i)}) \\\ \cdots \\\ \sum\limits\_{i=1}^{m}((h\_\theta(\mathbf{x}^{(i)}) - y^{(i)}) x\_i^{(i)}) \end{bmatrix} \end{aligned} $$

## Cost function for regularzation
```matlab
function [J, grad] = costFunctionReg(theta, X, y, lambda)

m = length(y);
n = size(theta,1);

J = (1/m)*sum(-y .* log(sigmoid(X*theta)) - (1 .- y) .* log(1 .- sigmoid(X*theta))) + (lambda/(2*m))*(theta'(2:n) * theta(2:n));

grad(1) = ((X')*(sigmoid(X*theta) - y)*(1/m))(1);
grad(2:n,:) = ((X')*(sigmoid(X*theta) - y)*(1/m) + (lambda/m)*theta)(2:n,:);

end
```
 - Notice that the only difference from last section is the penalize term
 - Notice we have to reinitialize $\theta\_0$ both in J and grad

## Prediction using the results
```matlab
function p = predict(theta, X)

m = size(X, 1); % Number of training examples

p = sigmoid(X*theta);
for iter = 1:m
    if (p(iter) >= 0.5)
        p(iter) = 1;
    else p(iter) = 0;
    end;
end;

end
```
 - Recall what X*theta is in the last section.

