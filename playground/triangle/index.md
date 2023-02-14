# Triangle

<!--more-->

[LeetCode 120](https://leetcode.com/problems/triangle)

The method is identical to [Minimal Path Sum]({{< relref "minimal-path-sum.md" >}}, only that the shape changes from rectangle to triangle.

```python
class Solution:
    def minimumTotal(self, triangle: List[List[int]]) -> int:
        n_row = len(triangle)
        dp = [[] for _ in range(n_row)]
        dp[0].append(triangle[0][0])

        for i in range(1, n_row):
            for j in range(i+1):
                top_left = dp[i-1][j-1] if j-1>=0 else float("inf")
                top_right = dp[i-1][j] if j < len(triangle[i-1]) else float("inf")
                minimal = min(top_left, top_right) + triangle[i][j]
                dp[i].append(minimal)
        return min(dp[-1])
```

Notes
1. Init the first element manually
2. Init the dp table by appending instead of indexing
3. For each element to iterate, check left bound for top left, check right bound for top right
