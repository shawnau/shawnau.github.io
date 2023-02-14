# Boosting(2) - Adaboost and Forward Stagewise


Boosting理论基础: 和前向分步算法的等价性

<!--more-->

### Forward stagewise
Consider an **additive model** like adaboost:
$$
f(x) = \sum\limits\_{m=1}^M \beta\_m b(x; \gamma\_m)
$$
in which $b(x; \gamma\_m)$ is the base model, $\gamma\_m$ is the model's parameters, $\beta\_m$ is it's coefficient.

To minimim the loss function, we have the **forward stagewise algorithm**, which **minimize one model & coefficient at a time**, that is, take the models already trained as constant, that's the core concept of this algorithm:

Input: Training data $T = \lbrace (x^{(1)}, y^{(1)}), (x^{(2)}, y^{(2)}), \cdots,  (x^{(N)}, y^{(N)}) \rbrace$, lost function$L(y, f(x))$ and base model $b(x; \gamma)$
Output: additive model $f(x)$
1. Initialize $f\_0(x) = 0$
2. For $m = 1,2,\cdots,M$:
    (1) Minimize loss function:
    $$  (\beta\_m, \gamma\_m) = arg\min\limits\_{\beta, \gamma} \sum\limits\_{i=1}^N L(y^{(i)}, f\_{m-1}(x^{(i)}) + \beta b(x^{(i)};\gamma)) $$
    (2) Update $f(x)$:
    $$ f\_m(x) = f\_{m-1}(x) + \beta\_m b(x^{(i)};\gamma\_m)$$
3. Get additive model:
    $$ f(x) = \sum\limits\_{m=1}^M \beta\_m b(x; \gamma\_m) $$

---
### Equivalence of adaboost and Forward stagewise
When using **additive model & exponential loss function**, forward stagewise algorithm is equivalent to adaboost. Here is the proof:

Assume the **exponential loss function** is:
$$ L(y, f(x)) = exp(-yf(x)) $$
In the $m$th iteration, we have:
$$
f\_m(x) = f\_{m-1}(x) + \alpha\_mG\_m(x), \\\
\text{in which } f\_{m-1}(x) = \sum\limits\_{m=1}^{M-1} \alpha\_mG\_m(x)
$$
To minimize
$$\begin{align}
(\alpha\_m, G\_m) & = arg\min\limits\_{\alpha, G} \sum\limits\_{i=1}^N exp \left\lbrace -y^{(i)}(f\_{m-1}(x^{(i)}) + \alpha\_m G\_m(x^{(i)}))\right\rbrace \\\
& = arg\min\limits\_{\alpha, G} \sum\limits\_{i=1}^N w\_{mi} exp \left\lbrace -y^{(i)} \alpha\_m G\_m(x^{(i)}) \right\rbrace
\end{align}
$$
in which $w\_{mi} = exp(-y^{(i)} f\_{m-1}(x^{(i)}))$, it depends on neither $\alpha$ nor $G$.

To prove the $\alpha\_m^\*, G\_m^\*$ achieved here is exatly the $\alpha\_m, G\_m$ in adaboost:

1. For any $\alpha > 0$, $G\_m^\*(x)$ which maximize the loss function is the one who has the minimum error rate:
$$
G\_m^\*(x) =  arg\min\limits\_{G} \sum\limits\_{i=1}^N w\_{mi} I(y^{(i)} \neq G(x^{(i)}))
$$
That's exatly what we train the base model to do, so $G\_m^\*(x) = G\_m(x)$
2. Then calculate $\alpha\_m$
 $$\begin{align} 
 L(\alpha, G\_m) & =  \sum\limits\_{i=1}^N w\_{mi} exp \left\lbrace -y^{(i)} \alpha G\_m(x^{(i)}) \right\rbrace \\\
 & = e^{-\alpha} \sum\limits\_{=} w\_{mi}  + e^{\alpha} \sum\limits\_{\neq} w\_{mi} \\\
 & = e^{-\alpha} \sum\limits w\_{mi} - e^{-\alpha} \sum\limits\_{\neq} w\_{mi} + e^{\alpha} \sum\limits\_{\neq} w\_{mi} \\\
 & = (e^{\alpha} - e^{-\alpha}) \sum\limits\_{\neq} w\_{mi} + e^{-\alpha} \sum\limits w\_{mi}
 \end{align} $$
 in which $\sum\limits\_{\neq}$ is abbreviation for $\sum\limits\_{y^{(i)} \neq G(x^{(i)})}$, $\sum$ is abbreviation for $\sum\limits\_{i=1}^N$
Let the partial derivative=0, we have:
$$\frac {\partial L(\alpha, G\_m)} {\partial \alpha} = (e^{\alpha} + e^{-\alpha}) \sum\limits\_{\neq} w\_{mi} - e^{-\alpha} \sum\limits w\_{mi} = 0$$
s.t. $$(e^{\alpha} + e^{-\alpha}) \frac{\sum\limits\_{\neq} w\_{mi}}{\sum\limits w\_{mi}} - e^{-\alpha}  = 0$$
s.t. $$(e^{\alpha} + e^{-\alpha}) \epsilon\_m - e^{-\alpha}  = 0$$
s.t. $$ \alpha^\* = \frac{1}{2}ln \frac{1-\epsilon\_m}{\epsilon\_m} $$
in which $\epsilon\_m = \frac{\sum\limits\_{\neq} w\_{mi}}{\sum\limits w\_{mi}}$, which is the error rate of $G\_m^\*(x)$

Finally we could see that the $(\alpha\_m, G\_m)$ we get here is exatly the same as in adaboost.
