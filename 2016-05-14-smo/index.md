# Support Vector Machine(3)-SMO


序列最小化算法

<!--more-->

Recall from the last section:
$ \begin{align}
& \min\limits\_{\alpha} \quad \frac{1}{2}  \sum\limits\_{i=1}^{N}  \sum\limits\_{j=1}^{N} \alpha\_i \alpha\_j y^{(i)} y^{(j)} K(x^{(i)},  x^{(j)}) - \sum\limits\_{i=1}^{N} \alpha\_i 
\end{align} \tag{3.2}$

$ s.t. \quad \sum\limits\_{i=1}^{N} \alpha\_i y^{(i)} = 0, \quad 0 \le \alpha\_i \le C \tag{3.4}$

To solve the optimization problem above, we introduce the *Sequantial Minimal Optimization*(SMO) method.


---

### Solve the 2-variable Quadratic Programming

Assume from all of the $\alpha\_i$, set only $\alpha\_1, \alpha\_2$ to be variables, all the other $\alpha\_i(i \neq 1, 2)$ are constant, set $\alpha\_2^{i}$ to be the $i$th iterate of $\alpha\_2$, first we define some things for convenience:

$$ L = \begin{cases}
max\lbrace 0, \alpha\_2^i - \alpha\_1^i \rbrace,  & y^{(1)} \neq y^{(2)} \\\
max\lbrace 0, \alpha\_2^i + \alpha\_1^i - C \rbrace,  & y^{(1)} = y^{(2)}
\end{cases}
$$

$$
H =  \begin{cases}
min \lbrace C, \alpha\_2^i - \alpha\_1^i + C \rbrace, & y^{(1)} \neq y^{(2)} \\\
min \lbrace C, \alpha\_2^i + \alpha\_1^i \rbrace, & y^{(1)} = y^{(2)}
\end{cases}
$$

$$E\_i = g(x^{(i)}) - y^{(i)} = \left( \sum\limits\_{j=1}^{N} \alpha\_j y^{(j)} K(x^{(i)},  x^{(j)})+b \right)- y^{(i)}
\tag{SMO.1}$$

Then using $H$ and $L$ to cut the $\alpha\_2$, in the $(i+1)$th iterate, we have:

$$ \alpha\_2^{i+1, unc} = \alpha\_2^{i} + \frac { y^{(2)}(E\_1 - E\_2)}{K\_{11}+K\_{22}-2K\_{12}} \tag{SMO.2}$$

$$ \alpha\_2^{i+1} = 
\begin{cases}
H, & \alpha\_2^{i+1, unc} > H \\\
\alpha\_2^{i+1, unc}, & L \le \alpha\_2^{i+1, unc} \le H \\\
L, & \alpha\_2^{i+1, unc} < L
\end{cases}
\tag{SMO.3}
$$

$$  \alpha\_1^{i+1} = \alpha\_1^{i} + y^{(1)}y^{(2)}(\alpha\_2^{i} - \alpha\_2^{i+1}) \tag{SMO.4} $$

---

### The Choosing of $\alpha\_1, \alpha\_2$
The KKT conditions for each $\alpha\_i$ are:
$$ \begin{align}
\alpha\_i = 0 \quad & \Leftrightarrow \quad y^{(i)} g(x^{(i)}) \ge 1 \quad \text{(Out of the border)} \\\
0 < \alpha\_i < C \quad & \Leftrightarrow \quad y^{(i)} g(x^{(i)}) = 1 \quad \text{(On the border)} \\\
\alpha\_i = C \quad & \Leftrightarrow \quad y^{(i)} g(x^{(i)}) \le 1 \quad \text{(Inside the border)}
\end{align}
\tag{SMO.5} $$

1. Select $\alpha\_1$: 
 - Iterate through the points on the border(which is most likely to be the support vector), then all the other points, for the points that breaks KKT conditions
2. Select $\alpha\_2$: 
 - Find $(x^{(i)}, y^{(i)})$ which gives the largest $\vert E^i\_1 - E^i\_2 \vert$. In order to save time for calculation, save all the $E\_i$ in advance

---

### Calculate new $b$ and $E\_i$
After
1. Calculate $b$
 $$ b\_1^{i+1} = b^i -[E\_1^i + y^{(1)}K\_{11}(\alpha\_1^{i+1} - \alpha\_1^{i}) + y^{(2)}K\_{21}(\alpha\_2^{i+1} - \alpha\_2^{i}) ] \\\ 
b\_2^{i+1} = b^i -[E\_2^i + y^{(1)}K\_{12}(\alpha\_1^{i+1} - \alpha\_1^{i}) + y^{(2)}K\_{22}(\alpha\_2^{i+1} - \alpha\_2^{i}) ]
 \tag{SMO.6} $$

$$b^{i+1} = 
\begin{cases}
b\_{1}^{i+1} = b\_{2}^{i+1}, & 0 <   \alpha\_1^{i+1}, \alpha\_2^{i+1} < C \\\
\frac {1}{2} (b\_{1}^{i+1} + b\_{2}^{i+1}), & \alpha\_1^{i+1}, \alpha\_2^{i+1} = 0 \; or \; C
\end{cases}
$$ 
2. Calculate $E\_i$ using $\alpha\_1^{i+1}, \alpha\_2^{i+1}, b^{i+1}$
$$ E\_i = \sum\limits\_{j \in S} \alpha\_j y^{(j)} K(x^{(i)},  x^{(j)})+b^{i+1} - y^{(i)} $$
