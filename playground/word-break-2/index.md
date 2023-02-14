# Word Break 2

<!--more-->

[leetcode 140](https://leetcode.com/problems/word-break-ii/)

This is pretty much an easier version of [Palindrome Partitioning]({{< relref "palindrome-partitioning.md" >}}), cuz finding a word is much easier than finding a valid palindrome substring.

Using a backtracking method:
1. When a substring is breakable, we iterate this substring and find all possible break positions.
2. Dfs search each possible breaks.
3. If search ends, we find a match.

### Corner Cases:
We can handle the situaion which search string is `''`, so we can put the terminal condition in the `self.breakable()`

```python
class Solution:
    def breakable(self, s: str, wordDict: List[str]) -> bool:
        n = len(s)
        dp = [False]*(n+1)
        dp[0] = True
        for i in range(1, n+1):
            for n in range(i+1):
                if dp[n] and s[n:i] in wordDict:
                    dp[i] = True
                    break
        return dp[n]

    def wordBreak(self, s: str, wordDict: List[str]) -> List[str]:
        n = len(s)
        results: List[str] = []
        def search(i:int, path: List[str]):
            if i == n:
                results.append(' '.join(path))
            if self.breakable(s[i:], wordDict):
                for j in range(i, n):
                    sub_str = s[i:j+1]
                    if sub_str in wordDict:
                        search(j+1, path+[sub_str])
        search(0, [])
        return results
```

