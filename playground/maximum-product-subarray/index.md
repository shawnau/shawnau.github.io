# Maximum Product Subarray

<!--more-->

[leetcode 152](https://leetcode.com/problems/maximum-product-subarray/)

The idea is the same like [Maximum Subarray]({{< relref "maximum-subarray.md" >}}), let $dp[i]$ to be the maximum product subarray value that **ends at** $nums[i]$, but we have some issues here:

1. If $dp[i-1] = 3$, while the minimum product subarray value that ends at $i$ is $-4$ and $nums[i] = -1$, then we get $-4 \cdot -1 > 3 \cdot -1$
2. Apparently we need to record the minimum product subarray as well, and we compare all the values.

$$ dp[i] = \max \lbrace dp_{min}[i-1] \cdot nums[i], dp_{max}[i-1] \cdot nums[i], nums[i] \rbrace $$

```python
class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        n, max_prod = len(nums), nums[0]
        dp = [{'max':0, 'min':0} for _ in range(n)]
        dp[0]['max'], dp[0]['min'] = nums[0], nums[0]
        for i in range(1, n):
            dp[i]['max'] = max(dp[i-1]['max']*nums[i], dp[i-1]['min']*nums[i], nums[i])
            dp[i]['min'] = min(dp[i-1]['max']*nums[i], dp[i-1]['min']*nums[i], nums[i])
            max_prod = max(max_prod, dp[i]['max'])
        return max_prod
```
