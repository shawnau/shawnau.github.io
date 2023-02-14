# Best Time to Buy and Sell Stock Iv


<!--more-->

[LeetCode 188](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)

Say you have an array for which the i-th element is the price of a given stock on day `i`.
Design an algorithm to find the maximum profit. You may complete at most `k` transactions.

### DP Solution

Let `dp[i][j]` to be the maximum profit to buy and sell the stock with at most `i` operations at `t=j`.

For any given time `t`, we have two options:
1. **Do nothing, or buy the stock**, the profit remains unchanged: `dp[i][j] = dp[i][j-1]`
2. **Sell the stock**. We must buy the stock first. Assume that we but the stock at $t \in [0, j-1]$
    - We have already gained profit `dp[i-1][t-1]`
    - Buy the stock: `-prices[t]`
    - Sell the stock: `prices[j]`
 Total profit: `dp[i-1][t-1] - prices[t] + prices[j]`
3. We need to iterate through all possible buying time:

$$ dp[i][j] = \max \left\lbrace dp[i][j-1], \max\limits_{t \in [0, j-1]} \left\lbrace dp[i-1][t-1] - p[t] \right\rbrace + p[j] \right\rbrace $$

Notice the 2nd term, we let $M_j=\max\limits_{t \in [0, j-1]} \left\lbrace dp[i-1][t-1] - p[t] \right\rbrace$ and $ m_{t} = dp[i-1][t-1] - p[t] $:

$$ M_{j} = \max\limits_{t \in [0, j-1]} \lbrace m_{t} \rbrace = \max \left\lbrace M_{j-1}, m_{j-1} \right\rbrace $$

At each iteration $i, j$, we could calculate $m_{j}=dp[i-1][j-1]-p[j]$ for the next iteration in advance. Because in current iteration we already have the data of `dp[i-1][x]`. Thus we can get rid of the inner loop and calculate $M_j$ in an iterative way.

### Corner Cases
We start from $i=1, j=1$ to avoid corner cases. For $i=0$ or $j=0$, $dp[i][j]=0$, so we don't need additional init steps.

```python
class Solution:
    def quickSolve(self, prices: List[int]) -> int:
        profit = 0
        for i in range(1, len(prices)):
            if prices[i] > prices[i - 1]:
                profit += prices[i] - prices[i-1]
        return profit
    
    def maxProfit(self, k: int, prices: List[int]) -> int:
        if k >= len(prices) // 2: return self.quickSolve(prices)
        
        dp = [[0]*len(prices) for _ in range(k+1)]
        for i in range(1, k+1):
            tmp_max = -prices[0]
            for j in range(1, len(prices)):
                dp[i][j] =  max(dp[i][j-1], tmp_max + prices[j])
                tmp_max = max(tmp_max, dp[i-1][j-1] - prices[j])
        
        return dp[k][-1]
```

