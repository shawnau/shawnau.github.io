# Distinct Subsequences


<!--more-->

[Leetcode 115](https://leetcode.com/problems/distinct-subsequences/)

This problem is quite similar to [Interleaving String]({{< relref "interleaving-string.md" >}}), but the solution is almost the same as [Regular Expression Matching]({{< relref "regular-expression-matching.md" >}}): we build top-down solution by backtracking, then build bottom-up solution of dp. 

---

## Intro problem

Let's discuss a simplified problem: How to check if `t` is a subsequence of `s`? Using a greddy strategy, this would be quite straight forward:

> Iterate through `s`, match `t` in a greddy manner.

1. If `t` has been iterated over, we find a match.
2. If `s` has been iterated over, we need to iterate over `t` as well, or there's no match, search over.
3. If not 1 or 2, we need to continue searching, skip the characters in `s[0]` which don't match `t[0]`

```python
def is_subsequence(s: str, t: str) -> bool:
    def search(i: int, j: int) -> bool:
        while i<len(s) and j<len(t) and s[i] != t[j]:
            i += 1
        
        if j == len(t):
            return True
        elif i == len(s):
            return False
        else:
            return search(i+1, j+1)
    
    return search(0, 0)
```

## Backtracking
Borrowing the idea from last section, we need to count all of the possible match, so we can't stop when we find a character matches, we need to skip this match and explore if there're other possibilities to achieve a match.

We define `search(i, j)` to be the distinct subsequences for `s[i:]` and `t[j:]`, despite that we search for the current match, we still goes for further possibilities in `s`

```python
def numDistinct(self, s: str, t: str) -> int:
    def search(i, j):
        while i<len(s) and j<len(t) and s[i] != t[j]:
            i += 1
        
        if j == len(t):
            return 1
        elif i == len(s):
            return 0
        else:
            return search(i+1, j+1) + search(i+1, j)

    return search(0, 0)
```
Note:
1. We can easily optimize time complexity by using a cache to avoid duplicate calculation. See [Optimize Recursion]({{< relref "optimize-recursion.md" >}}).

## Dynamic Programming

WIP
