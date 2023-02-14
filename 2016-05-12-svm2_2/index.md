# Support Vector Machine(2.2)-Solving dual problem for soft margin maximization


SVM之软间隔最大化

<!--more-->

Recall from the 1st section, we have:
$$ \begin{align}
& \min\limits\_{w, \xi} \quad \frac{1}{2} {\Vert w \Vert}^2 + C \sum\limits\_{i=1}^{N}\xi\_i   \\\
s.t. \quad & y^{(i)} \left ( w \cdot x^{(i)} + {b} \right ) + \xi\_i - 1\ge  0 \\
& \xi\_i \ge 0\tag{1.10}
\end{align}$$
For the constraint condition and target function in (1.10), using  [the method of Lagrange multipliers](https://www.wikiwand.com/en/Lagrange\_multiplier), (1.10) could be represented as:



---

$$ \begin{align}
& L(w,b,\xi,\alpha,\mu) = \frac{1}{2} {\Vert w \Vert}^2 + C \sum\limits\_{i=1}^{N}\xi\_i + \sum\limits\_{i=1}^{N} \alpha\_i (1 -\xi\_i - y^{(i)}(w \cdot x^{(i)} + b)) + \sum\limits\_{i=1}^{N} \mu\_i(-\xi\_i)
\tag{3.1} \end{align}$$

###  Solving $\min\limits\_{w,b} L(w,b,\xi,\alpha,\mu)$
 $$ \nabla\_w  L(w,b,\xi,\alpha,\mu) = w -  \sum\limits\_{i=1}^{N} \alpha\_i y^{(i)} x^{(i)} = 0 \\\
 \nabla\_b  L(w,b,\xi,\alpha,\mu) = \sum\limits\_{i=1}^{N} \alpha\_i y^{(i)} = 0 \\\
 \nabla\_{\xi\_i}  L(w,b,\xi,\alpha,\mu) = C-\alpha\_i - \mu\_i = 0
\tag{3.2}$$

Luckily it has the same form as (2.4) and (2.5), see the next section below.

---

###  Solving $\alpha$ for $\max\limits\_{\alpha} \ \min\limits\_{w,b,\xi} L(w,b,\xi,\alpha,\mu) $
The dual problem for soft margin maximization is:
$ \begin{align}
& \min\limits\_{\alpha} \quad \frac{1}{2}  \sum\limits\_{i=1}^{N}  \sum\limits\_{j=1}^{N} \alpha\_i \alpha\_j y^{(i)} y^{(j)} (x^{(i)} \cdot x^{(j)}) - \sum\limits\_{i=1}^{N} \alpha\_i 
\end{align} \tag{3.2}$

$ s.t. \quad \sum\limits\_{i=1}^{N} \alpha\_i y^{(i)} = 0, \quad
 \alpha\_i \ge 0, \quad 
 \mu\_i \ge 0, \quad
 C- \alpha\_i-\mu\_i =0 
\tag{3.3} $

(3.3) could be simplified as:

$$ s.t. \quad \sum\limits\_{i=1}^{N} \alpha\_i y^{(i)} = 0, \quad 0 \le \alpha\_i \le C \tag{3.4}$$

---

### Solving $w,b$
Once $\alpha^{\*}$ is solved, we have:
$$ w^{\*} =  \sum\limits\_{i=1}^{N} \alpha\_i^{\*} y^{(i)} x^{(i)} \\\
b^{\*} = y^{(j)} - \sum\limits\_{i=1}^{N} \alpha\_i^{\*} y^{(i)} (x^{(i)} \cdot x^{(j)})
 \tag{3.5}$$
 in which the $(x^{(j)}, y^{(j)})$ satisfies: $ 0 \le \alpha\_j \le C $, which contains $ 1 - \xi\_i -y^{(j)}(w^{\*} \cdot x^{(j)} + b^{\*})) = 0 $, which means it's on the classification border 'belt'.

---

### Hinge loss function

Let $\quad [ 1 - y^{(i)} \left ( w \cdot x^{(i)} + {b} \right ) ]\_+ = \xi\_i $, (1.10) could also be represented as

$$\min\limits\_{w,b} \quad \sum\limits\_{i=1}^N \xi\_i + \lambda {\Vert w \Vert}^2 \tag{3.6}$$

Let $\lambda = \frac{1}{2C}$, then it is equivalent to (1.10), so the loss function  could also be written as the hinge loss function:

$$ \sum\limits\_{i=1}^N[ 1 - y^{(i)} \left ( w \cdot x^{(i)} + {b} \right ) ]\_+ +  \lambda {\Vert w \Vert}^2 \tag{3.7}$$
