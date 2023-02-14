# Boosting(1) - AdaBoost


Adaptive Boosting 算法

<!--more-->

### Combine different classifiers & weights
Test data: 
$$T = \left\lbrace (x^{(1)}, y^{(1)}), (x^{(2)}, y^{(2)}), \cdots, (x^{(N)}, y^{(N)})\right\rbrace, \quad 
x \in \chi \subseteq R^N, y \in \lbrace +1, -1 \rbrace \\\
\text{with weights: } D\_i = (w\_{i1}, w\_{i2}, \cdots, w\_{iN})$$

Classifier: 
$$G\_m(x): \chi \to \lbrace +1, -1\rbrace, \quad G(x) = \sum\limits^M\_{i=1} \alpha\_m G\_m(x) \\\
\text{with weights: } A = (\alpha\_{1}, \alpha\_{2}, \cdots, \alpha\_{M})
$$


---

### AdaBoost Algorithm
Input: Training data $T$, classifier $G\_m(x)$
Output: Ensemble $G(x)$

1. Initialize data weights:
$$ D\_1 = (w\_{11}, w\_{12}, \cdots,  w\_{1N}), \quad w\_{1i} = \frac{1}{N} $$
2. For $m = 1,2, \cdots, M$:
    (a) Train data with $D\_m = (w\_{m1}, w\_{m2}, \cdots,  w\_{mN})$ weights on $T$, generate $G\_m(x)$
    (b) Calculate error rate of $G\_m(x)$ on $T$:
    $$ e\_m = P(G\_m(x^{(i)}) \neq y^{(i)}) = \sum\limits^N\_{i=1} w\_{mi}I(G\_m(x^{(i)}) \neq y^{(i)}) \tag{1}$$
    (c) Calculate $G\_m(x)$'s weight $\alpha\_m$:
    $$ \alpha\_m = \frac{1}{2} ln \frac{1-e\_m}{e\_m} \tag{2}$$
    (d) Update $D\_{m+1}$:
    $$ D\_{m+1} = (w\_{m+1, 1}, w\_{m+1, 2}, \cdots,  w\_{m+1, N}) $$
    $$w\_{m+1, i}  =\begin{cases} 
    \frac{w\_{mi}}{Z\_m} e^{-\alpha\_m},  & G\_m(x^{(i)}) = y^{(i)} \\\ \\\
    \frac{w\_{mi}}{Z\_m}e^{\alpha\_m},  & G\_m(x^{(i)}) \neq y^{(i)} 
    \end{cases}
    = \frac {w\_{mi}e^{-\alpha\_m y^{(i)} G\_m(x^{(i)})}} {Z\_m} \tag{3}$$
    $$Z\_m = \sum\limits^N\_{i=1}w\_{mi}e^{-\alpha\_m y^{(i)} G\_m(x^{(i)})} $$
    In which $Z\_m$ is normalization factor, which makes $D\_{m+1}$ a probability distribution, keeping $\sum\limits^N\_{i=1}w\_{m+1, i} = 1$
3. Combine all the $G\_m(x)$:
    $$ f(x) =  \sum\limits^M\_{i=1} \alpha\_m G\_m(x), \quad G(x) = sign(f(x))$$

Note on implementation: The weights on datasets could be implemented with sampling the original dataset with a probability distribution of $D\_i$

---

### Intuitive Explanation
![function](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/Logistic.png)
 
 - Here is how $G\_m(x)$'s coefficient, $ \alpha\_m = \frac{1}{2} ln \frac{1-e\_m}{e\_m} $ varies through $e\_m$, it's easy to see that the more precise model will get a larger weight.
 - Contrarily, from (3) it's easy to tell that the samples get classified wrong will increase their weights, while the weights of the samples get classified correct will decrease, both in an exponential rate.
