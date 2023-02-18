# 吴恩达 ML 公开课笔记(9)-Programming Exercise 3


Programming Exercise 3

<!--more-->

## Multi-class Classification
1. CostFunction for fmincg
 ```matlab
function [J, grad] = lrCostFunction(theta, X, y, lambda)

% Initialize some useful values
m = length(y); % number of training examples
n = size(theta,1);

J = (1/m)*sum(-y .* log(sigmoid(X*theta)) - (1 .- y) .* log(1 .- sigmoid(X*theta))) + (lambda/(2*m))*(theta'(2:n) * theta(2:n));

grad(1) = ((X')*(sigmoid(X*theta) - y)*(1/m))(1);
grad(2:n,:) = ((X')*(sigmoid(X*theta) - y)*(1/m) + (lambda/m)*theta)(2:n,:);

end
 ```
 - The same as is in Programming Exercise 2

2. One-vs-all Classification
 ```matlab
function [all_theta] = oneVsAll(X, y, num_labels, lambda)

% Some useful variables
m = size(X, 1);
n = size(X, 2);

% Initialize X and all_theta
X = [ones(m, 1) X];
all_theta = zeros(num_labels, n + 1);

% Initialize options for fmincg
initial_theta = zeros(n + 1, 1);
options = optimset('GradObj', 'on', 'MaxIter', 50);

% Generate rows of all_theta row by row using fmincg
for iter = 1:num_labels
	all_theta(iter,:) = (fmincg (@(t)(lrCostFunction(t, X, (y == iter), lambda)), initial_theta, options))';
end;

end

 ```
 - ONEVSALL trains multiple logistic regression classifiers and returns all the classifiers in a matrix all_theta, where the i-th row of all_theta corresponds to the classifier for label i
 - The parameter `y == iter` returns a vector of the same size as y with ones at positions where the elements of y are equal to iter and zeroes where they are different.

3. One-vs-all Prediction
 ```matlab
function p = predictOneVsAll(all_theta, X)

m = size(X, 1);
num_labels = size(all_theta, 1);

% You need to return the following variables correctly 
p = zeros(size(X, 1), 1);

% Add ones to the X data matrix
X = [ones(m, 1) X];
      
[value,p] = max(X*all_theta', [], 2);
p = p';

end
 ```
 - Notice that the max element's column-index in each row of `X*all_theta'` happensto be the number it represents(10 for 0)

4. Main Function
 ```matlab
%% Initialization
clear ; close all; clc
%% =========== Part 1: Loading Data =============
load('ex3data1.mat');
load('ex3data1.mat');
m = size(X, 1);
%% ============ Part 2: Vectorize Logistic Regression ============
lambda = 0.1;
[all_theta] = oneVsAll(X, y, num_labels, lambda);
%% ================ Part 3: Predict for One-Vs-All ===============
pred = predictOneVsAll(all_theta, X);
fprintf('\nTraining Set Accuracy: %f\n', mean(double(pred == y)) * 100);
end
 ```

## Neural Networks
1. Feedforward Propagation and Prediction
 ```matlab
function p = predict(Theta1, Theta2, X)

% Useful values
m = size(X, 1);
num_labels = size(Theta2, 1);
% Add ones to the X data matrix
X = [ones(m, 1) X];

% Feedforward Propagation
a_2 = [ones(1, m);sigmoid(Theta1*X')];
a_3 = sigmoid(Theta2*a_2);

[values, p] = max(a_3);
p = p';
end
 ```
 - The parameters of each layer has already been trained into Theta1 and Theta2
 - Each column of the output layer a_3 is the output for each row of X, which is each row-vectorized picture.
 - The prediction part, especially `[values, p] = max(a_3)` works the same as One-vs-all Prediction function, except for the row-index rather than column-index.

2. Main Function
 ```matlab
%% =========== Part 1: Loading Data =============
load('ex3data1.mat');
m = size(X, 1);
%% =========== Part 2: Loading Pameters =========
load('ex3weights.mat');
%% =========== Part 3: Implement Predict ========
pred = predict(Theta1, Theta2, X);
fprintf('\nTraining Set Accuracy: %f\n', mean(double(pred == y)) * 100);
end
 ```

