# Support Vector Machine(2.1)-Solving dual problem for hard margin maximization


SVM之硬间隔最大化

<!--more-->

In the last section, (1.8) is a *convex quadratic programming* problem.
Using  [the method of Lagrange multipliers](https://www.wikiwand.com/en/Lagrange_multiplier), (1.8) could be represented as:
$$ L(w,b,\alpha) = \frac{1}{2} {\Vert w \Vert}^2 + \sum\limits\_{i=1}^{N} \alpha\_i (1-y^{(i)}(w \cdot x^{(i)} + b)), \alpha\_i \ge 0 \tag{2.1}$$

The dual problem is:
$$\max\limits\_{\alpha} \ \min\limits\_{w,b} L(w,b,\alpha) \tag{2.2}$$


----

###  Solving $\min\limits\_{w,b} L(w,b,\alpha)$
 $$ \nabla\_w  L(w,b,\alpha) = w -  \sum\limits\_{i=1}^{N} \alpha\_i y^{(i)} x^{(i)} = 0  \\\
  \nabla\_b  L(w,b,\alpha) = \sum\limits\_{i=1}^{N} \alpha\_i y^{(i)} = 0 \tag{2.3}$$
 
 Combine (2.3) with (2.1), we have the dual problem:
 $$ \begin{align}
& \max\limits\_{\alpha} \left\lbrace - \frac{1}{2}  \sum\limits\_{i=1}^{N}  \sum\limits\_{j=1}^{N} \alpha\_i \alpha\_j y^{(i)} y^{(j)} (x^{(i)} \cdot x^{(j)}) + \sum\limits\_{i=1}^{N} \alpha\_i \right\rbrace  \\\  
& s.t. \quad \sum\limits\_{i=1}^{N} \alpha\_i y^{(i)} = 0, \alpha\_i \ge 0
\end{align} \tag{2.4}$$

----

###  Solving $\alpha$ for $\max\limits\_{\alpha} \ \min\limits\_{w,b} L(w,b,\alpha) $
$$ \begin{align}
& \min\limits\_{\alpha} \left\lbrace \frac{1}{2}  \sum\limits\_{i=1}^{N}  \sum\limits\_{j=1}^{N} \alpha\_i \alpha\_j y^{(i)} y^{(j)} (x^{(i)} \cdot x^{(j)}) - \sum\limits\_{i=1}^{N} \alpha\_i \right\rbrace \\\  
& s.t. \quad \sum\limits\_{i=1}^{N} \alpha\_i y^{(i)} = 0, \alpha\_i \ge 0
\end{align} \tag{2.5}$$

 The solved parameter $\alpha^{\*}$ should satisfy the KKT condition (2.3), (2.4) and $ \alpha\_i^{\*} (1-y^{(i)}(w^{\*} \cdot x^{(i)} + b^{\*})) = 0 $

----

### Solving $w,b$
Once $\alpha^{\*}$ is solved, we have:
$$ w^{\*} =  \sum\limits\_{i=1}^{N} \alpha\_i^{\*} y^{(i)} x^{(i)} \\\
b^{\*} = y^{(j)} - \sum\limits\_{i=1}^{N} \alpha\_i^{\*} y^{(i)} (x^{(i)} \cdot x^{(j)})
 \tag{2.6}$$
 in which the $(x^{(j)}, y^{(j)})$ satisfies: $ 1-y^{(j)}(w^{\*} \cdot x^{(j)} + b^{\*}) = 0 $, which means it's on the classification border.
 
Notice that when $\alpha\_i = 0$, $(x^{(i)}, y^{(i)})$ has no contribution in deciding the $w$ and $b$, which means that only a few points on the border decides the SVM with their $\alpha\_i > 0$, and $ 1-y^{(i)}(w^{\*} \cdot x^{(i)} + b^{\*}) = 0 $, they are support vectors.


