# Unique Paths

Count all paths from a to b
<!--more-->

## Description

See [LeetCode 62](https://leetcode.com/problems/unique-paths/)

## Dynamic Programming Solution

Let $dp[i][j]$ to be the total unique paths to $board[i][j]$, the state transision is easy to find:

```
            dp[i-1][j]
               ^
               |
dp[i][j-1]<--dp[i][j]
```

$$ dp[i][j] = dp[i-1][j] + dp[i][j-1] $$

1. Define a custom indexing function to take care of corner cases.
2. The filling order is described as follow for $row=3, col=4$:

| 0 | 1 | 2 | 3 |
|---|---|---|---|
| 4 | 5 | 6 | 7 |
| 8 | 9 | 10 | 11 |

```python
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        dp = [[0]*n for _ in range(m)]
        dp[0][0] = 1
        def index(i, j):
            if i in range(m) and j in range(n):
                return dp[i][j]
            else:
                return 0

        for i in range(m):
            for j in range(n):
                if i == 0 and j == 0:
                    continue
                dp[i][j] = index(i-1, j) + index(i, j-1)
        
        return dp[m-1][n-1]
```
