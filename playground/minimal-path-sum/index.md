# Minimal Path Sum


<!--more-->

[LeetCode 64](https://leetcode.com/problems/minimum-path-sum/)

This is pretty much like [unique paths]({{< relref "unique-paths.md" >}}).

Let $dp[i][j]$ to be the total unique paths to $board[i][j]$, the state transision is easy to find:

$$ dp[i][j] = \min \lbrace dp[i-1][j], dp[i][j-1] \rbrace + grid[i][j] $$

As in the unique paths, we define a custom indexing function to take care of corner cases.

```python
class Solution:
    def minPathSum(self, grid: List[List[int]]) -> int:
        n_row, n_col = len(grid), len(grid[0])
        min_sum = float("inf")
        
        def index(i, j):
            if i in range(n_row) and j in range(n_col):
                return dp[i][j]
            else:
                return float("inf")
        
        dp = [[float("inf")]*n_col for _ in range(n_row)]
        dp[0][0] = grid[0][0]
        for i in range(n_row):
            for j in range(n_col):
                if i == 0 and j == 0:
                    continue
                dp[i][j] = min(index(i-1, j), index(i, j-1)) + grid[i][j]
        return dp[n_row-1][n_col-1]
```
