# 吴恩达 ML 公开课笔记(12)-Advice for Applying Machine Learning


Advice for Applying Machine Learning

<!--more-->

## Debugging a learning algorithm
 - Get more training data
 - Try smaller sets of features
 - Getting additional features
 - Changing fit hypothesis
 - Changing $\lambda$

## Evaluate hypothesis
1. Training/testing procedure
 - Splitting data into **training set** and **test set**
 - Learn parameters from training data
 - Compute test error
 - If $J\_{train}(\theta)$ is low while $J\_{test}(\theta)$ is high, then it might be overfitting.
2. For logistic regression
 - Misclassification error:
 $$
err(h\_\theta (x), y) = 
\begin{cases}
1, & [h\_\theta (x)] \text{ XOR } y = 0 \\\
0, & [h\_\theta (x)] \text{ XOR } y = 1 \\\
\end{cases} \\\
TestError = \frac {1}{m\_{test}} \sum\limits\_{i=1}^{m\_{test}} err(h\_\theta (x), y)
 $$
 - It's an alternative option other than cost function

## Model Selection
1. Set d = degree of polynomial as the extra parameter other than $\theta$ 
2. Try different models, calculate cost function on training set, choose the d with the minimum cost function. **Problem**: May fit training set well but not generalize to all data.
3. Solution: Split training data into 3 pieces: training set(60%), cross validation set(20%), test set(20%)
4. Using validation set to choose the model:
 - Min(Training error)
 - Min(Validation error), Pick
 - Test error shows how well the model generalizes

## Bias vs. Variance
1. Diagnosing Bias vs. Variance
 - ![errorWithD](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/bias_variance.png)
 - Cross validation error and training error with different degree of polynomial d
 - The left side indicates high bias while the right side indicates high variance
2. 'Error - training set size graph' affected by different hypothesis
 - ![errorWithBias](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/bias_variance2.png)
 - The error with different training set size when the hypothesis is of high bias
 - ![errorWithVariance](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/bias_variance_3.png)
 - The error with different training set size when the hypothesis is of high variance
3. 'Error - training set size graph' affected by different regularzation parameter
 - ![errorWithReg](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/regular_on_bias.png)
4. Debugging a learning algorithm
 - ![debugging](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/bias_variance_4.png)

## Building a Spam Classifier
 - Do a dirty and quick example in order to examine whether a method like stemming might help lowering the error

## Handling Skewed Data
1. Skewed data: A set of very unbalanced data
2. Precision/recall
 - ![precisionRecall](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/precision_recall.png)
 - High classfier threshold: High precision, low recall: Very confident, lots of overlook
 - Low classfier threshold: High recall, low precision: Not very confident, but less overlook
3. Choose threshold automatically
 - ![FScore](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/precision_recall2.png)
 - F score is better than average on deciding which algo to use based on precision/recall data

## Using Large Data Sets
 - Generally more training data will improve learning performance
 - ![LargeData](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/large_data2.png)
 - Having a significantly large set of data will improve learning performance: Low training error -> training error ≈ test error -> all errors are low
