# Palindrome Partitioning


<!--more-->

[LeetCode 131](https://leetcode.com/problems/palindrome-partitioning/)

Although this is similar to [Longest Palindromic Substring]({{< relref "longest-palindromic-substring.md" >}}), yet the idea is similar to [Word Break 2]({{< relref "word-break-2.md" >}}). We use backtracking at each place it could be partitioned.

```python
class Solution:
    def partition(self, s: str) -> List[List[str]]:
        n = len(s)
        results: List[List[str]] = []
        def search(i, path: List[str]):
            if i == n: results.append(path)
            
            for j in range(i, n):
                sub = s[i:j+1]
                if sub == sub[::-1]:
                    search(j+1, path+[sub])
        search(0, [])
        return results
```
