# Support Vector Machine(4)-SMO implementation


SMO算法的python实现

<!--more-->

[Full implementation here](https://github.com/shawnau/MLiA/blob/master/SVM/smo_kernel.py)

### SMO.1: Calculating $L$, $H$ and $E\_i$
```
if p.y[i] != p.y[j]:
    L = max(0, p.a[j] - p.a[i])
    H = min(p.c, p.a[j] - p.a[i] + p.c)
else:
    L = max(0, p.a[j] + p.a[i] - p.c)
    H = min(p.c, p.a[j] + p.a[i])
```
```
def calc_ei(p, i):
    f_xi = float(np.dot((p.a * p.y).T, np.dot(p.x, p.x[i].T)) + p.b)
    ei = f_xi - float(p.y[i])
    return ei
```



---

### SMO.2: Calculating $\alpha\_2^{i+1, unc}$
```
eta = np.dot(p.x[i], p.x[i].T) + np.dot(p.x[j], p.x[j].T) - 2 * np.dot(p.x[i], p.x[j].T)
p.a[j] += p.y[j] * (ei - ej) / eta
```

---

### SMO.3: Clipping $\alpha\_2^{i+1, unc}$
```python
def clip_a(ai, H, L):
    if ai > H:
        return H
    elif ai < L:
        return L
    return ai
```
`p.a[j] = clip_a(p.a[j], H, L)`

---

### SMO.4: Calculating $\alpha\_1^{i+1}$
`p.a[i] = p.a[i] + p.y[i] * p.y[j] * (a_j_old - p.a[j])`

---

### SMO.5: Selecting $\alpha\_1$, $\alpha\_2$
 -  Selecting $\alpha_1$ that breaks the KKT conditions within precision `tol`
```
if ((p.a[i] < p.c) and (p.y[i] * ei < -p.tol)) or \
    ((p.a[i] > 0) and (p.y[i] * ei > p.tol)):
    j, ej = select_aj(i, p, ei)
```
 -  Selecting $\alpha_2$ which gives the largest $\vert E^i_1 - E^i_2 \vert$. In order to save time for calculation, save all the $E_i$ in `e_cache` advance. If there is none, using `random_select_aj()` randomly select one instead.
```
def select_aj(i, p, ei):
    max_index = -1; ej = 0; max_delta_e = 0
    p.e_cache[i] = [1, ei]
    valid_indexes = np.transpose(np.nonzero(p.e_cache[:, 0]))
    if len(valid_indexes) > 1:
        for k in valid_indexes:
            if k == i: continue
            ek = calc_ei(p, k)
            delta_e = abs(ei - ek)
            if delta_e > max_delta_e:
                max_index = k; max_delta_e = delta_e; ej = ek
        return max_index, ej
    else:
        j = random_select_aj(i, p.m)
        ej = calc_ei(p, j)
    return j, ej
```

---

### SMO.6: Calculating $b$
```
bi = p.b - (ei + p.y[i] * np.dot(p.x[i], p.x[i].T) * (p.a[i] - a_i_old) +
          p.y[j] * np.dot(p.x[j], p.x[i].T) * (p.a[j] - a_j_old))
bj = p.b - (ej + p.y[i] * np.dot(p.x[i], p.x[j].T) * (p.a[i] - a_i_old) +
          p.y[j] * np.dot(p.x[j], p.x[j].T) * (p.a[j] - a_j_old))
if (p.a[i] > 0) and (p.a[i] < p.c):
    p.b = bi
elif (p.a[j] > 0) and (p.a[j] < p.c):
    p.b = bj
else:
    p.b = (bi + bj) / 2.0
```
