# Word Break


<!--more-->

[leetcode 139](https://leetcode.com/problems/word-break/)
This is an easy version of [Longest Palindromic Substring]({{< relref "longest-palindromic-substring.md" >}}), cuz finding a word is much easier than finding a valid palindrome substring.

Let $dp[i]$ to be if $s[0, i]$ is a valid break, we have:

> If $s[0, n]$ is a valid break and $s[n+1, i]$ is in the dict $S$, then $s[0, i]$ is a valid break. i.e.
$$ dp[n] \land \lbrace s[n+1, i] \in S \rbrace \to dp[i] $$

It looks like we need to firstly iterate the $s$, and iterate substring at each position $i$, which makes it $O(n^2)$ complexity. When we find a match in the substring, we can stop the iteration.


### Corner Cases
If we check substring from $s[0, i]$, then we don't have $dp[-1]$, we can insert a `True` to the $dp$ table, and be careful of the index changes: 
 - When indexing $dp$, $[0, n-1] \to [1, n]$
 - when indexing $s$, we need to use $s[i-1]$

$$dp[i] = \bigcap_{n=0}^{i} \left\lbrace dp[n] \land \lbrace s[n+1, i] \in S \rbrace \right\rbrace $$

```python
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        n = len(s)
        dp = [False]*(n+1)
        dp[0] = True
        for i in range(1, n+1):
            for n in range(i+1):
                if dp[n] and s[n:(i-1)+1] in wordDict:
                    dp[i] = True
                    break
        return dp[n]
```
