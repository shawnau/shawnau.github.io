# Wildcard Matching

<!--more-->

## Description
Given an input string (s) and a pattern (p), implement wildcard pattern matching with support for '?' and '*'.

 - '?' Matches any single character.
 - '*' Matches any sequence of characters (including the empty sequence).
The matching should cover the entire input string (not partial).

Note:

 - s could be empty and contains only lowercase letters a-z.
 - p could be empty and contains only lowercase letters a-z, and characters like ? or *.

Example 1:
```
Input:
s = "aa"
p = "a"
Output: false
Explanation: "a" does not match the entire string "aa".
Example 2:
```
```
Input:
s = "aa"
p = "*"
Output: true
Explanation: '*' matches any sequence.
```

## Backtracking Solution

This is an easier version of `Regular Expression Matching`, since we don't need to check character matching when we get a wildcard. The logic is:

We define `search(si, pi)` as the search function, which means if `s[si:]` is matched by `p[pi:]`.

First check if it is a wildcard: `pi < len(p) and p[pi] == "*"`
1. If it is, match this or not: `search(si+1, pi) or search(si, pi+1)`
2. If not, just match current character: `si < len(s) and p[pi] in {s[si], "?"}`
    - Search next level: `search(si+1, pi+1)`

```python
class Solution:
    def isMatch(self, s: str, p: str) -> bool:
        def search(si: int, pi: int) -> bool:
            if pi == len(p): 
                return si == len(s)
            wildcard: bool = pi < len(p) and p[pi] == "*"
            if wildcard:
                return (si < len(s) and search(si+1, pi)) or search(si, pi+1)
            else:
                match: bool = si < len(s) and p[pi] in {s[si], "?"}
                return match and search(si+1, pi+1)
        return search(0, 0)
```

Again, it could be optimized through cache.

## Dynamic Programming

First we can construct state transision from the backtracking method. Jut let `search(si, pi)` to be `dp[si][pi]`

```text
dp[si][pi] <--dp[si][pi+1]
    ^      \
    |       \  
dp[si+1][pi]  dp[si+1][pi+1]
```

 - We have a `(len(s) + 1, len(p) +1)` matrix, the iteration order is described below. 
 - Starting from `dp[len(s)][len(p)-1]`, we need to get `dp[0][0]`
 - Set `dp[len(s)][len(p)] = True`, which indicates our backtracking terminal condition(find a match).


|s\p|\*|a|\*|b||
|:-:|:-:|:-:|:-:|:-:|:-:|
|a|end||||F|
|b|||||F|
|c|||||F|
|e|||||F|
|b|||||F|
|||||start|T|

```python
class Solution:
    def isMatch(self, s: str, p: str) -> bool:
        dp: List[List[bool]] = [[False]*(len(p)+1) for _ in range(len(s)+1)]
        dp[len(s)][len(p)] = True
        for si in range(len(s), -1, -1):
            for pi in range(len(p)-1, -1, -1):
                wildcard: bool = pi < len(p) and p[pi] == "*"
                if wildcard:
                    dp[si][pi] = (si < len(s) and dp[si+1][pi]) or dp[si][pi+1]
                else:
                    match: bool = si < len(s) and p[pi] in {s[si], "?"}
                    dp[si][pi] = match and dp[si+1][pi+1]
        return dp[0][0]

```
