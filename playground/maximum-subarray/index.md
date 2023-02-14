# Maximum Subarray


<!--more-->

## Description

Given an integer array nums, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.

Example:
```
Input: [-2,1,-3,4,-1,2,1,-5,4],
Output: 6
```
Explanation: [4,-1,2,1] has the largest sum = 6.
Follow up:

If you have figured out the O(n) solution, try coding another solution using the divide and conquer approach, which is more subtle.

## Dynamic Programming Solution

Let $dp[i]$ to be the maximum subarray value that **ends at** $nums[i]$, which is all the possible subarray sum which ends at $i$, i.e.

$$ \begin{aligned} dp[i] &= \max_{j \in [0, i] } \left\lbrace \sum\limits_{k=j}^{i} nums[k] \right\rbrace \\\ &= \max_{j \in [0, i] } \left\lbrace \sum\limits_{k=j}^{i-1} nums[k],0 \right\rbrace + nums[i] \\\ &= \max \left\lbrace dp[i-1], 0 \right\rbrace + nums[i] \end{aligned} $$

```python
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        dp, max_sum = [0]*len(nums), -float("inf")
        for i in range(len(nums)):
            dp[i] = (max(dp[i-1], 0) if i-1>=0 else 0) + nums[i]
            max_sum = max(max_sum, dp[i])
        return max_sum
```

Notes
1. When calculating `dp[i-1]`, watch out for the corner case.
2. Generate `max_sum` in each iter on the fly.

