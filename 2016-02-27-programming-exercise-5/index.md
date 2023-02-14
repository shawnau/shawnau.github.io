# Machine Learning Notes(13)-Programming Exercise 5


Programming Exercise 5

<!--more-->

**Notification: This is a simplified code example, if you are attempting this class, don't copy & submit since it won't even work...**

## Regularized linear regression
```matlab
function [J, grad] = linearRegCostFunction(X, y, theta, lambda)
% Initialize some useful values
m = length(y); % number of training examples
n = size(theta,1);
% You need to return the following variables correctly 
J = 0;
grad = zeros(size(theta));
% Calculate cost function
J = (1/(2*m))*sum((X*theta .- y).^2) + (lambda/(2*m))*(sum(theta.^2)-theta(1)^2);
% Calculate the partial derivative
grad(1) = ((X')*(X*theta - y)*(1/m))(1);
grad(2:n,:) = ((X')*(X*theta - y)*(1/m) + (lambda/m)*theta)(2:n,:);

grad = grad(:);

end
```

$$
J(\theta) = \frac {1}{2m} \sum\limits\_{i=1}^{m}(h\_\theta(x^{(i)})-y^{(i)})^2 + \frac{\lambda}{2m}(\sum\limits\_{j=1}^{n}\theta\_j^2)
$$

$$
\begin{cases}
\frac{\partial}{\partial \theta\_0} J(\theta) = \frac{1}{m}\sum\limits\_{i=1}^{m}[(h\_\theta (x^{(i)}) - y^{(i)})x\_j^{(i)}] & \text{for j=0} \\\
\frac{\partial}{\partial \theta\_j} J(\theta) = \frac{1}{m}\sum\limits\_{i=1}^{m}[(h\_\theta (x^{(i)}) - y^{(i)})x\_j^{(i)}] + \frac{\lambda}{m}\theta\_j & \text{for j>0}
\end{cases}
$$

## Learning curves
```matlab
function [error_train, error_val] = ...
    learningCurve(X, y, Xval, yval, lambda)
% Number of training examples
m = size(X, 1);
n = size(Xval, 1);
% You need to return these values correctly
error_train = zeros(m, 1);
error_val   = zeros(m, 1);
% Calculate error_train and error_val for each train set
for i = 1:m
	[theta] = trainLinearReg(X(1:i,:), y(1:i), lambda);
	error_train(i) = (1/(2*i))*sum((X(1:i,:)*theta .- y(1:i)).^2);
	error_val(i) = (1/(2*n))*sum((Xval*theta .- yval).^2);
end;

end
```
$$
J\_{train}(\theta) = \frac {1}{2m} \sum\limits\_{i=1}^{m}(h\_\theta(x^{(i)})-y^{(i)})^2
$$

## Polynomial regression
```matlab
function [X_poly] = polyFeatures(X, p)
% You need to return the following variables correctly.
n = numel(X);
X_poly = zeros(n, p);

for j = 1:p
    for i = 1:n
        X_poly(i,j) = X(i)^j;
    end
end;

end
```
 - Given a vector X, return a matrix X_poly where the p-th column of X contains the values of X to the p-th power

## Learning Polynomial Regression
```matlab
function [lambda_vec, error_train, error_val] = ...
    validationCurve(X, y, Xval, yval)
% Selected values of lambda (you should not change this)
lambda_vec = [0 0.001 0.003 0.01 0.03 0.1 0.3 1 3 10]';
m = size(X, 1);
n = size(Xval, 1);
% You need to return these variables correctly.
error_train = zeros(length(lambda_vec), 1);
error_val = zeros(length(lambda_vec), 1);

for i = 1:length(lambda_vec)
	lambda = lambda_vec(i);
	[theta] = trainLinearReg(X, y, lambda);
	error_train(i) = (1/(2*m))*sum((X*theta .- y).^2);
	error_val(i) = (1/(2*n))*sum((Xval*theta .- yval).^2);
end;

end
```
 - This function returns the train and validation errors (in error_train, error_val) for different values of lambda, same as `linearRegCostFunction()`
