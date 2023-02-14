# Unique Paths 2

Count all paths from a to b, with obstacles.
<!--more-->

## Description

See [LeetCode 63](https://leetcode.com/problems/unique-paths-ii/)

## Dynamic Programming Solution

Let $dp[i][j]$ to be the total unique paths to $board[i][j]$, the state transision is easy to find:

```
            dp[i-1][j]
               ^
               |
dp[i][j-1]<--dp[i][j]
```
$$ dp[i][j] = \begin{cases} dp[i-1][j] + dp[i][j-1],  & \text{$board[i][j] = 0$} \\\ 0, & \text{$board[i][j] = 1$} \end{cases} $$

Watch out for the initial condition: we con't initialize all values to 1 on the first row and first col.

```python
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        n_row, n_col = len(obstacleGrid), len(obstacleGrid[0])
        dp = [[0]*n_col for _ in range(n_row)]
        if obstacleGrid[0][0] != 1:
            dp[0][0] = 1
        
        def index(i, j):
            if i in range(n_row) and j in range(n_col):
                return dp[i][j]
            else:
                return 0
        
        for i in range(n_row):
            for j in range(n_col):
                if i == 0 and j == 0:
                    continue
                if obstacleGrid[i][j] == 0:
                    dp[i][j] = index(i-1, j) + index(i, j-1)
        
        return dp[n_row-1][n_col-1]
```

