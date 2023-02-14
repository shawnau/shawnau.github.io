# Edit Distance

This is about calculation Levenshtein Distance

<!--more-->

[LeetCode 72](https://leetcode.com/problems/edit-distance/)

## Levenshtein Distance

Check [Levenshtein Distance](https://www.wikiwand.com/en/Levenshtein_distance) for information.

Let $\operatorname{lev}_{a,b}(i,j)$ be the distance between the first $i$ characters of $a$ and the first $j$ characters of $b$. $i$ and $j$ are 1-based indices. We have:

$$ \operatorname {lev}\_{a,b}(i,j) = \begin{cases} \max(i,j) & \text{ if $min(i,j)=0$}, \\\ \min {\begin{cases} \operatorname {lev}\_{a,b}(i-1,j)+1 \\\ \operatorname {lev}\_{a,b}(i,j-1)+1 \\\ \operatorname {lev}\_{a,b}(i-1,j-1)+1\_{(a\_{i} \neq b\_{j})} \end{cases}} &{\text{otherwise.}} \end{cases} $$

### Explanation

For simplification, Let `dp[i][j]` represent $\operatorname{lev}_{a,b}(i,j)$.

We have 3 ways to reach `dp[i][j]`. Take `a='abxy', b='abc'` as an example:
1. From `dp[i-1][j]`: Having `abx` -> `abc`, we need one extra **del** step: 
    - `abxy` -> `abcy` (del `y`)-> `abc`
2. From `dp[i][j-1]`: Having `abxy` -> `ab`, we need one extra **add** step: 
    - `abxy` -> `ab` (add `c`)-> `abc`
3. From `dp[i-1][j-1]`: Having `abx` -> `ab`, we need one extra **replace** step: 
    - `abxy` -> `aby` (replace `y` to `c`)-> `abc`
    - If `a[i] == b[j]`, We could skip the replace step.

### DP Solution

Then we consider the corner cases. We have 0-indexing system, in which situation we can achive `-1` indexing.

1. If `i = -1` or `j = -1`, then we need to convert an empty string to another non-empty string, by inserting, the cost is the length of the non-empty string, in the above example, when `i=-1` and `j=1`, we convert `''` to `ab`, the cost is 2, `dp[i][j]` = `j+1` = `2`
2. If `i = -1` and `j = -1`, we convert an empty string to another empty string, `dp[-1][-1]` = `0`


There are two strategies to take care of these situations.
1. Using a custom indexing system to take care of index overflow. We could use the dp as it was.
2. Taking care of the indexing on the fly. This is less readable than method 1, personally not recommended.
3. Add an extra row and column, start from (1,1) instead of (0,0), which has pros and cons compared to method 1.

We use method 1:

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)
        
        if m == 0 and n == 0: return 0
        elif m == 0: return n
        elif n == 0: return m

        dp = [[float("inf")]*n for _ in range(m)]
        
        def index(i, j):
            if i == -1 and j == -1: return 0
            elif i == -1: return j+1
            elif j == -1: return i+1
            else: return dp[i][j]
        
        for i in range(m):
            for j in range(n):
                cost = int(word1[i]!=word2[j])
                dp[i][j] = min(index(i-1,j)+1, index(i,j-1)+1, index(i-1,j-1)+cost)
        
        return dp[-1][-1]
```

This method has problem to deal with empty string, in which we generate the result in `index()` on the fly, which is out of the dp table itself. Try method 3:

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)
        dp = [[float("inf")]*(n+1) for _ in range(m+1)]
        dp[0][0] = 0
        for i in range(1, n+1):
            dp[0][i] = i
        for j in range(1, m+1):
            dp[j][0] = j
        
        for i in range(1, m+1):
            for j in range(1, n+1):
                cost = int(word1[i-1]!=word2[j-1])
                dp[i][j] = min(dp[i-1][j]+1,dp[i][j-1]+1,dp[i-1][j-1]+cost)
        
        return dp[-1][-1]
```
Notes:
1. word index changes from `i`, `j` to `i-1`, `j-1` in method 3.
2. We may optimize the space of the code to use only two vectors, or even one.

