# Poisson Distribution Summary


泊松分布是随机过程中的一个重要分布

<!--more-->

## Poisson分布的直观解释
定义
$$ {\displaystyle P(k{\text{ events in interval}})=e^{-\lambda }{\frac {\lambda ^{k}}{k!}}} $$

>that expresses the probability of a given number of events occurring in a fixed interval of time and/or space if these events occur with a known average rate and independently of the time since the last event.

 - 以上是wiki的定义: 如果已知事件的发生的平均频率$\lambda$, 而且事件之间条件独立, 那么在固定的时／空段内发生给定次数(k)事件的概率符合Poisson分布.
 - 继续拿wiki举例: 已知一个人平均每天收到4封邮件, 如果邮件之间条件独立, 那么每天收到的邮件数目符合Poisson分布. 
 - 而这个概率就是Poisson分布的唯一参数$\lambda$. 把前面的例子扩展一下, 每两天收到的邮件数目的概率符合$\lambda = 8$的Poisson分布

## Poisson分布和其他分布的关系
### 与二项分布的关系
> The binomial distribution converges towards the Poisson distribution as the number of trials goes to infinity while the product np remains fixed or at least p tends to zero. Therefore, the Poisson distribution with parameter λ = np can be used as an approximation to B(n, p) of the binomial distribution if n is sufficiently large and p is sufficiently small. According to two rules of thumb, this approximation is good if n ≥ 20 and p ≤ 0.05, or if n ≥ 100 and np ≤ 10.

对一个二项分布, 在p较小, n较大的情况下, 可以作如下等价: 每次Bernoulli实验都相当于在一定时间范围内发生1次事件的概率是p, 那么重复n次之后, 使用Poisson分布来定义的话就是:
>在n时间范围内, 平均发生$np$次的事件, 发生给定次数的概率分布, 符合$\lambda=np$的Poisson分布.

### 与指数分布的关系
> If for every t > 0 the number of arrivals in the time interval [0, t] follows the Poisson distribution with mean λt, then the sequence of inter-arrival times are independent and identically distributed exponential random variables having mean 1/λ.

 - 泊松过程中，第k次随机事件与第k+1次随机事件出现的时间间隔服从参数为$\frac{1}{\lambda}$的指数分布. 
 - Proof:
 设两次随机事件之间间隔的时间为$X$, 第k次随机事件之后长度为t的时间段内，第k+1次随机事件出现的概率等于1减去这个时间段内没有随机事件出现的概率, 而这个概率即为$N \sim Poi(t\lambda)$中的$P(N = 0)$
 $$ P(X>t) = 1 - \frac{(\lambda t)^0}{0!} e^{-\lambda t} = 1-e^{-\lambda t}$$ 即指数分布的PDF

 > 待续
