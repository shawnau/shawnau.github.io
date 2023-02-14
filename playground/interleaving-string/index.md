# Interleaving String


<!--more-->

[Leetcode 97](https://leetcode.com/problems/interleaving-string/)

## Description

Given s1, s2, s3, find whether s3 is formed by the interleaving of s1 and s2.

Example:
```
Input: s1 = "aabcc", s2 = "dbbca", s3 = "aadbbcbcac"
Output: true
```
Explanation: aa<mark>dbbc</mark>bc<mark>a</mark>c, aa<mark>db</mark>bc<mark>bca</mark>c are all valid interpretations for the interleaving.

Notice:
1. The order of the strings cannot be changed, which indicates that we can construct `s3` using `s1` and `s2` from their substrings.
2. Different ways of interleaving can led to same results.

### Example of constructing a string

We define $dp[i][j]$ as:  If $a3[0, i+j]$ can be constructed by $a1[0, i]$ and $a2[0, j]$
We have 2 choices to construct $a3[0, i+j+1]$: using $a1[i+1]$ or $a2[j+1]$

```python
if dp[i][j] and s1[i+1] == s3[i+j+1]: dp[i+1][j] = True # Use s1[i+1] to construct s3[i+j+1]
if dp[i][j] and s2[j+1] == s3[i+j+1]: dp[i][j+1] = True # Use s2[j+1] to construct s3[i+j+1]
```
So we have 2 ways to get the needed `dp[i][j]`

```python
use_s1 = dp[i-1][j] and s1[i] == s3[i+j]
use_s2 = dp[i][j-1] and s2[j] == s3[i+j]
```
i.e.

```python
dp[i][j] = use_s1 or use_s2
```
---

### Corner Cases

If $i=0, j=5$, it represents that if $a3[0, 5]$ can be constructed by $s1=\varnothing$ and $s2[0,5]$, we only need to check that if $s2[0,5] = s3[0,5]$

If we use 0-based index, this situation corresponding to `dp[-1][4]`, which is not quite convenience. Using 1-based indexing we could initialize the first row and col and then calculating the dp table from `dp[1][1]`

|   | 0 | d | b | b | c | a |
|---|---|---|---|---|---|---|
| 0 | T | F | F | F | F | F |
| a | T | F | F | F | F | F |
| a | T | T | T | T | F | F |
| b | F | T | T | F | F | F |
| c | F | F | T | T | T | T |
| c | F | F | F | T | F | T |

```python
class Solution:
    def isInterleave(self, s1: str, s2: str, s3: str) -> bool:
        if len(s1) + len(s2) != len(s3): return False
        
        m, n = len(s1), len(s2)
        dp = [[False]*(n+1) for _ in range(m+1)]
        
        for i in range(n+1):  # s1 is empty
            dp[0][i] = s2[:i] == s3[:i]
        for i in range(m+1):  # s2 is empty
            dp[i][0] = s1[:i] == s3[:i]
        
        for i in range(1, m+1):
            for j in range(1, n+1):
                use_s1 = dp[i-1][j] and s1[i-1] == s3[i+j-1]
                use_s2 = dp[i][j-1] and s2[j-1] == s3[i+j-1]
                dp[i][j] = use_s1 or use_s2
        
        return dp[-1][-1]
```

Note:
1. Under 1-based indexing, take care of the indexing on `s1`, `s2` and `s3`.

