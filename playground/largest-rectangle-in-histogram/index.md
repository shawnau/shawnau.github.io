# Largest Rectangle in Histogram


<!--more-->

[LeetCode 84](https://leetcode.com/problems/largest-rectangle-in-histogram)

The idea is somewhat similar to [Longest Valid Parentheses]({{< relref "longest-valid-parentheses.md" >}}), in which:

1. Maintain a stack which stores the indices
2. Calculate the target function each time an element pops out from the stack
3. Take care of the corner case with `-1` when we don't have the leftmost element

---

Maintain a stack of indices, in which:
1. All the elements in the stack follows ascending order.
2. The elements popped out between adjacent elements remains in the stack are all taller, i.e. 
$$height[stack[j]] <= height[k], \\ \forall k \in [stack[j-1], stack[j]]$$
3. So for any element $j$ popped out
    - leftmost element is $stack[-1] + 1$
    - rightmost element is current index $i-1$
    - the width is $(i-1)-(stack[-1]+1)+1=i-stack[-1]-1$

<table>
<tr><th>Before height[3] enter</th><th>After height[3] enter</th></tr>
<tr><td>

| 0 | 1 | 2 | 3 | <mark>4</mark> |
|---|---|---|---|---|
|   |   |   | x |   |
|   |   | x | x |   |
|   | x | x | x | <mark>x</mark> |
| x | x | x | x | <mark>x</mark> |

</td><td>

| 0 | 1 | <mark>4</mark> |
|---|---|---|
|   |   | o |
|   |   | o |
|   | x | <mark>x</mark> |
| x | x | <mark>x</mark> |

</td></tr> </table>

 - when 3 pops, the leftmost is 1+1=2, rightmost is 4-1=3

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
```
Notes:
1. To ensure that `left` and `right` is valid, watch out for the corner cases when calculating.
2. Append `0` to `heights` to ensure it pop out all the elements in the stack.
