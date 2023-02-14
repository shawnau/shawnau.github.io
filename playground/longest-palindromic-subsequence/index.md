# Longest Palindromic Subsequence


<!--more-->

[LeetCode 516](https://leetcode.com/problems/longest-palindromic-subsequence/)

This is similar to [Longest Palindromic Substring]({{< relref "longest-palindromic-substring.md" >}}), but it is hard to use the expanding idea cuz the subsequence is not continuous. We can try to use the dp idea as an initial:

Let $dp[i][j]$ to be the longest palindromic substring for $s[i, j]$, then:

$$ dp[i][j] = \begin{cases} dp[i+1][j-1] + 2,  & \text{$s[i]=s[j]$} \\\ \max\lbrace dp[i][j-1], dp[i+1][j] \rbrace, & \text{$s[i] \neq s[j]$} \\\ 1, &\text{$i=j$} \\\ 0, &\text{$i>j$} \end{cases} $$

```

dp[i][j-1]<--dp[i][j]
            /    |
           /     |
dp[i+1][j-1] dp[i+1][j]
```
### Corner Cases
There are two corner cases as listed above, $i=j$ and $i<j$, we need to initialize them first.

| i\j | 0 | 1 | 2 | 3 | 4 |
|:---:|:-:|:-:|:-:|:-:|:-:|
|  0  | 1 | a | c | f | j |
|  1  | 0 | 1 | b | e | i |
|  2  |   | 0 | 1 | d | h |
|  3  |   |   | 0 | 1 | g |
|  4  |   |   |   | 0 | 1 |

```python
class Solution:
    def longestPalindromeSubseq(self, s: str) -> int:
        n = len(s)
        if n == 0: return 0
        dp = [[0]*n for _ in range(n)]
        # init two corner cases
        for i in range(n):
            dp[i][i] = 1
        for i in range(1, n):
            dp[i][i-1] = 0
        # filling the table
        for j in range(1, n):
            for i in range(j-1, -1, -1):
                if s[i] == s[j]:
                    dp[i][j] = dp[i+1][j-1] + 2
                else:
                    dp[i][j] = max(dp[i+1][j], dp[i][j-1])
        return dp[0][n-1]
```

## Backtracking Solution

WIP
