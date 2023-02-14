# Survival Models



![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/58319465_p0.jpg)
生存模型(Survival Models)属于General Linear Model, 被广泛用于Censored Data的建模, 譬如用户流失预测. 这里介绍下最基本的生存模型以及在Censored Data上的MLE估计

<!--more-->

## Survival Function
Assume $T$ is a continuous random variable indicates the death occurrence time, we have:

$$
F(t) = P\lbrace T < t\rbrace = \int\_0^t f(t) dt
\tag{1.1}
$$

Then the **Survival Function** should be:

$$
S(t) = P\lbrace  T > t\rbrace = 1 - F(t) = \int\_t^\infty f(t) dt
\tag{1.2}
$$

## Harzard Function
An alternative way to characterization the distribution is given by **harzard function**, or *instantaneous rate of occurrence of the event*:

$$
\begin{align}
\lambda(t) &= \lim\_{dt \to 0} \frac{P\lbrace t \le T < t + dt | T \ge t\rbrace}{dt} \\\
&= \lim\_{dt \to 0} \frac{P\lbrace t \le T < t + dt \rbrace}{P \lbrace T \ge t\rbrace dt} \\\
&= \lim\_{dt \to 0} \frac{f(t)dt}{S(t) dt} \\\
&= \frac{f(t)}{S(t)} 
\end{align}
\tag{2.1}
$$

Given $(1.2)$ we have $\frac{d}{dt} S(t) = -f(t)$, so $(2.1)$ has another form

$$
\lambda(t) = -\frac{d}{dt} log S(t)
\tag{2.2}
$$

We could derive survival function from harzard function as well:

$$
S(t) = exp\lbrace  - \int\_0^t \lambda(x)dx \rbrace = exp\lbrace  -\Lambda(t) \rbrace
\tag{2.3}
$$

In which $\Lambda(t) = \int\_0^t \lambda(x)dx$,  called *cumulative hazard*

---

### Example 2.1

Here we're modeling a constant risk over time:
$$
\lambda(t) = \lambda
$$
From $(2.2)$, we could solve corresponding survival function and pdf
$$
\begin{align}
S(t) &= exp\lbrace  - \int\_0^t \lambda(x)dx \rbrace = e^{-\lambda t} \\\
f(t) &= \lambda e^{-\lambda t}
\end{align}
$$
That is exactly an **exponential distribution**

---

### Expectation of Life
Given $S(t)$ or $\lambda(t)$, it's easy to denote expected value of $T$
$$
\mu = \int\_0^\infty tf(t)dt =\int\_0^\infty S(t)dt
$$

## Censoring and the likelihood function
### Censoring Type
1. *Type I*
 Typically 2 types of observatioin:
    - A sample of $n$ units is followed for a fixed time $\tau$
    - Generalization, *fixed censoring*: each unit has a fixed time $\tau\_i$
 
 In cases above, number of deaths is a random variable.

2. *Type II*
  - A sample of $n$ units is followed as long as necessary until $d$ units have experienced the event
  - Generalization, *random censoring*: Each unit has:
      - Censoring time $C\_i$
      - Potential lifetime $T\_i$
      - Observe time $Y\_i = min\lbrace  C\_i, T\_i\rbrace$
      - Indicator $d\_i, \delta\_i$ tells us whether the observation is terminated by death or censoring

### Likelihood of censoring model
1. Unit died at $t\_i$. Since we know it is dead while survives till $t\_i$, we have:
$$
L\_i = f(t\_i) = S(t\_i)\lambda(t\_i)
\tag{3.1}
$$

2. Unit still alive at $t\_i$. We only know it survives till $t\_i$
$$
L\_i = f(t\_i) = S(t\_i)
\tag{3.2}
$$

Given 2 conditions above, we have:
$$
L = \prod\limits\_{i=1}^{n}L\_i = \prod\limits\_{i} \lambda(t\_i)^{d\_i}S(t\_i)
\tag{3.3}
$$
Taking logs, considering $(2.3)$, we have:
$$
log L = \sum\limits\_{i=1}^{n} \lbrace  d\_ilog\lambda(t\_i) - \Lambda(t\_i) \rbrace
\tag{3.4}
$$

---

### Example 3.1
Considering exponential distribution $\lambda(t) = \lambda$, from$(3.4)$, we have
$$
log L = \sum\limits\_{i=1}^{n} \lbrace  d\_ilog\lambda - \lambda t\_i \rbrace
$$

We could estimate $\lambda$ using MLE:

Let $D=\sum d\_i$ denotes the total number of deaths, $T = \sum t\_i$ denotes total number of observation time:

$$
\begin{align}
log L &= Dlog\lambda - T\lambda \\\
\frac{\partial}{\partial \lambda} L &= \frac{D}{\lambda} - T
\end{align}
$$

Letting $\frac{\partial}{\partial \lambda} L = 0$ we get the estimation of $\lambda$

$$
\hat \lambda = \frac{D}{T}
$$

---

> [Reference](http://data.princeton.edu/wws509/notes/c7.pdf)
