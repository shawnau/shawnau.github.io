# 吴恩达 ML 公开课笔记(11)-Programming Exercise 4


Programming Exercise 4

<!--more-->

**Notification: This is a simplified code example, if you are attempting this class, don't copy & submit since it won't even work...**
## Cost function for fmincg
```matlab
function [J grad] = nnCostFunction(nn_params, ...
                                   input_layer_size, ...
                                   hidden_layer_size, ...
                                   num_labels, ...
                                   X, y, lambda)

% Reshape nn_params back into the parameters Theta1 and Theta2, the weight matrices
% for our 2 layer neural network
Theta1 = reshape(nn_params(1:hidden_layer_size * (input_layer_size + 1)), ...
                 hidden_layer_size, (input_layer_size + 1));

Theta2 = reshape(nn_params((1 + (hidden_layer_size * (input_layer_size + 1))):end), ...
                 num_labels, (hidden_layer_size + 1));

% Setup some useful variables
m = size(X, 1);
         
% You need to return the following variables correctly 
J = 0;
Theta1_grad = zeros(size(Theta1));
Theta2_grad = zeros(size(Theta2));

%% --------------Feedforward propagation---------------
for i = 1:m
	a_1 = [1 X(i,:)]';
	a_2 = [1; sigmoid(Theta1*a_1)];
	a_3 = sigmoid(Theta2*a_2);
% Transform y to standard nn outpout
	y_vector = zeros(num_labels,1);
	y_vector(y(i)) = 1;
% Calculate costfunction
	J = J + (1/m)*sum(-y_vector .* log(a_3) - (1 .- y_vector) .* log(1 .- a_3));
end;

% Calculate Regularzation
Square1 = Theta1.^2;
Square2 = Theta2.^2;

reg_sum = sum(sum(Square1)) - sum(Square1(:,1)) + sum(sum(Square2)) - sum(Square2(:,1));

J = J + (lambda/(2*m))*reg_sum;

%% ----------------Calculate Gradient-----------------
for i = 1:m
	a_1 = [1 X(i,:)]';
	a_2 = [1; sigmoid(Theta1*a_1)];
	a_3 = sigmoid(Theta2*a_2);
% Transform y to standard nn outpout
	y_vector = zeros(num_labels,1);
	y_vector(y(i)) = 1;
% Backpropagation
	sigma_3 = a_3 - y_vector;
	sigma_2 = Theta2'*sigma_3 .* a_2 .* (1 .- a_2);
% Vectorized partial derivative matrix
	Theta1_grad = Theta1_grad + sigma_2(2:end)*a_1';
	Theta2_grad = Theta2_grad + sigma_3*a_2';
end;
% Calculate regulazation terms
Theta1_grad = (1/m)*Theta1_grad .+ (lambda/m)*[zeros(size(Theta1,1),1),Theta1(:,2:end)];
Theta2_grad = (1/m)*Theta2_grad .+ (lambda/m)*[zeros(size(Theta2,1),1),Theta2(:,2:end)];

% -------------------------------------------------------------

% Unroll gradients
grad = [Theta1_grad(:) ; Theta2_grad(:)];


end

```

## Main function
```matlab
%% Initialization
clear ; close all; clc

%% Setup the parameters you will use for this exercise
input_layer_size  = 400;  % 20x20 Input Images of Digits
hidden_layer_size = 25;   % 25 hidden units
num_labels = 10;          % 10 labels, from 1 to 10   
                          % (note that we have mapped "0" to label 10)

%% =========== Part 1: Loading and Visualizing Data =============
load('ex4data1.mat');
m = size(X, 1);
%% ================ Part 2: Loading Parameters ================
% Load the weights into variables Theta1 and Theta2
load('ex4weights.mat');

% Unroll parameters 
nn_params = [Theta1(:) ; Theta2(:)];
%% ================ Part 3: Initializing Pameters ================
initial_Theta1 = randInitializeWeights(input_layer_size, hidden_layer_size);
initial_Theta2 = randInitializeWeights(hidden_layer_size, num_labels);

% Unroll parameters
initial_nn_params = [initial_Theta1(:) ; initial_Theta2(:)];
%% =============== Part 4: Implement Backpropagation ===============
%  value to see how more training helps.
options = optimset('MaxIter', 50);

%  You should also try different values of lambda
lambda = 1;

% Create "short hand" for the cost function to be minimized
costFunction = @(p) nnCostFunction(p, ...
                                   input_layer_size, ...
                                   hidden_layer_size, ...
                                   num_labels, X, y, lambda);

% Now, costFunction is a function that takes in only one argument (the
% neural network parameters)
[nn_params, cost] = fmincg(costFunction, initial_nn_params, options);

% Obtain Theta1 and Theta2 back from nn_params
Theta1 = reshape(nn_params(1:hidden_layer_size * (input_layer_size + 1)), ...
                 hidden_layer_size, (input_layer_size + 1));

Theta2 = reshape(nn_params((1 + (hidden_layer_size * (input_layer_size + 1))):end), ...
                 num_labels, (hidden_layer_size + 1));
%% ================= Part 5: Implement Predict =================
pred = predict(Theta1, Theta2, X);

fprintf('\nTraining Set Accuracy: %f\n', mean(double(pred == y)) * 100);
```
