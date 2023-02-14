# Count Different Palindromic Subsequences


<!--more-->

[LeetCode 730](https://leetcode.com/problems/count-different-palindromic-subsequences/)

The problem seems to be similar to [Longest Palindromic Subsequence]({{< relref "longest-palindromic-subsequence.md" >}}), but tt's hard to enumerate all the subsequences since it goes to $O(2^n)$ complexity.

Actually, the distinct subsequence is similar to [Distinct Subsequences II](https://leetcode.com/problems/distinct-subsequences-ii/). We could use the method called Sequence Automata. Check: [序列自动机总结与例题](https://www.cnblogs.com/y2823774827y/p/10619179.html), [序列自动机](https://blog.csdn.net/pig_dog_baby/article/details/81145857)

```python
class Solution:
    def init_next(self, s: str, a: int) -> List[List[int]]:
        n = len(s) - 1
        get_next = [[0]*a for _ in range(n+1)]
        for i in range(n, 0, -1):
            for c in range(a):
                get_next[i-1][c] = get_next[i][c]
            get_next[i-1][ord(s[i])-ord('a')] = i
        return get_next

    def countPalindromicSubsequences(self, S: str) -> int:
        s1, s2, n, a, mod = ' '+S, ' '+S[::-1], len(S), 4, 10**9+7
        next1, next2 = self.init_next(s1, a), self.init_next(s2, a)
        
        def dfs(i, j) -> int:
            ans = 0
            for c in range(a):
                i_n, j_n = next1[i][c], next2[j][c]
                if i_n and j_n:
                    if i_n + j_n > n + 1: continue
                    if i_n + j_n < n + 1: ans += 1
                    ans = (ans + dfs(i_n, j_n)) % mod
            return ans + 1

        return dfs(0, 0) - 1
```

