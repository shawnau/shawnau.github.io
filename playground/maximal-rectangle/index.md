# Maximal Rectangle


<!--more-->

[LeetCode 85](https://leetcode.com/problems/maximal-rectangle/)

Borrow the idea from [Largest Rectangle in Histogram]({{< relref "largest-rectangle-in-histogram.md" >}}), we can take the matrix as many histograms for **each row level**, in which we scan from top to down cumulatively to construct it:

<table>
<tr><th>Matrix</th><th>Histograms</th></tr>
<tr><td>

| 1 | 1 | 0 | 0 |
|---|---|---|---|
| 1 | 0 | 1 | 0 |
| 0 | 1 | 1 | 0 |
| 0 | 1 | 1 | 1 |
| 1 | 1 | 1 | 1 |

</td><td>

| 1 | 1 | 0 | 0 |
|---|---|---|---|
| 2 | 0 | 1 | 0 |
| 0 | 1 | 2 | 0 |
| 0 | 2 | 3 | 1 |
| 1 | 3 | 4 | 2 |

</td></tr> </table>

```python
class Solution:
    def largestRectangleArea(self, heights):
        stack, max_area = [], 0
        heights.append(0)
        for i in range(len(heights)):
            while stack and heights[i] < heights[stack[-1]]:
                popped_idx = stack.pop()
                height = heights[popped_idx]
                left = stack[-1] + 1 if stack else 0
                right = max(i-1, 0)
                width = right - left + 1
                max_area = max(max_area, height*width)
            stack.append(i)
        return max_area

    def maximalRectangle(self, matrix: List[List[str]]) -> int:
        n_row, n_col = len(matrix), len(matrix[0])
        max_area = 0
        heights = [0]*n_col
        for i in range(n_row):
            for j in range(n_col):
                if matrix[i][j] == "1":
                    heights[j] += 1
                else:
                    heights[j] = 0
            max_area = max(max_area, self.largestRectangleArea(heights))
        return max_area
```

Notes
1. Actually this is not a dp solution
