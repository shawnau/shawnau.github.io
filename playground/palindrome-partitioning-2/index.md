# Palindrome Partitioning 2


<!--more-->

[LeetCode 132](https://leetcode.com/problems/palindrome-partitioning-ii/)

We need to borrow the idea from [Longest Palindromic Substring]({{< relref "longest-palindromic-substring.md" >}}), using the ad-hoc method, try to expand at each index as the center.

Let $dp[i]$ to be the minimum cuts for $s[0, i]$, initialize it to the maximum cuts, which is $i-1$ (in 0-based indexing, it is $i$)

By using the expanding idea, when we get a palindrome substring $s[l, r]$, the minimum cuts for $s[0, r]$ shrinks into $dp[l-1]+1$. Iterate through all the possible palindrome substrings.

| a | b | a | c | d | d | c | a | a | a |
|---|---|---|---|---|---|---|---|---|---|
| 0 | 1 | 0 | 1 | 2 | 2 | 1 | 2 | 2 | 2 |

### Corner Case
When `i-1<0`, i.e. we have a palindrome substring which starts from `0`, e.g. `s=abc`, `r=2`, then the `dp[0-1]=0`, means we don't need any cut for string `abc` cuz it itself is a palindrome substring.

```python
class Solution:
    def minCut(self, s: str) -> str:
        n = len(s)
        dp = [i for i in range(n)]
        for i in range(n):
            for j in range(i+1):
                l, r = i-j, i+j
                if l>=0 and r<n and s[l]==s[r]:
                    dp[r] = min(dp[r], dp[l-1]+1 if l-1>=0 else 0)
                else:
                    break

            for j in range(i+1):
                l, r = i-1-j, i+j
                if l>=0 and r<n and s[l]==s[r]:
                    dp[r] = min(dp[r], dp[l-1]+1 if l-1>=0 else 0)
                else:
                    break
        
        return dp[-1]
```
