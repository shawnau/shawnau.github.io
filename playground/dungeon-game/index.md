# Dungeon Game


<!--more-->

[LeetCode 174](https://leetcode.com/problems/dungeon-game/)

## A False Example
We demonstrate a false example to show that there is no top-down, left-right dp solution to this problem. 
1. We need to test all possible paths from $(0,0)$ to $(i,j)$, find the lowest bound of the path sum along each path, from start to end. (Actually the value is lowest bound + 1, which keeps the knight alive)
2. Keep two tables, $dpSum[i][j]$ records the path sum to have $dpMin[i][j]$, which records the lowest bound upon current position along this path.

If we have both $(i-1, j)$ and $(i, j-1)$ for above two tables, do we have $dp[i][j]$ ? The answer is no. Think about it.

## Solution

Actually we need to calculate at a bottom-up, right-left order. Let $dp[i][j]$ to be the lowest HP to start from $(i,j)$ to the end, we have:

$$dp[i][j] = \begin{cases} min(dp[i][j-1], dp[i-1][j]) - grid[i][j],  & \text{$min(dp[i][j-1], dp[i-1][j]) - grid[i][j] > 0$} \\\ 1, & \text{$min(dp[i][j-1], dp[i-1][j]) - grid[i][j] \leq 0$} \end{cases}$$

```python
class Solution:
    def calculateMinimumHP(self, grid: List[List[int]]) -> int:
        n_row, n_col = len(grid), len(grid[0])
        
        dp = [[float('inf')]*n_col for _ in range(n_row)]
        dp[n_row-1][n_col-1] = 1 - min(0, grid[n_row-1][n_col-1])
        # init last row and col
        for i in range(n_row-2, -1, -1):
            need = dp[i+1][n_col-1] - grid[i][n_col-1]
            dp[i][n_col-1] = need if need > 0 else 1
        for j in range(n_col-2, -1, -1):
            need = dp[n_row-1][j+1] - grid[n_row-1][j]
            dp[n_row-1][j] = need if need > 0 else 1
        # init dp table
        for i in range(n_row-2, -1, -1):
            for j in range(n_col-2, -1, -1):
                need = min(dp[i+1][j], dp[i][j+1]) - grid[i][j]
                dp[i][j] = need if need > 0 else 1
                
        return dp[0][0]
```
