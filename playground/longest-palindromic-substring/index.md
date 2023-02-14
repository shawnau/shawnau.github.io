# Longest Palindromic Substring


<!--more-->

## Description

Given a string `s`, find the longest palindromic substring in `s`. You may assume that the maximum length of `s` is 1000.

Example 1:
```
Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.
```
Example 2:
```
Input: "cbbd"
Output: "bb"
```

## Dynamic Programming Solution

Let `dp[i][j]` indicates that `s[i:j+1]` is a palindromic sequence, we have

$$ dp[i][j] = \begin{cases} True,  & \text{$i$ = $j$} \\\ s[i] = s[j], & \text{$j$ - $i$ = 1}  \\\ (s[i] = s[j]) \\ \text{and} \\ dp[i+1][j-1], & \text{$j$ - $i$ > 1} \end{cases} $$

```
              dp[i][j]
            /
dp[i+1][j-1]
```

Filling order is as described below:

| i\j | b | a | b | a | d  |
|-----|---|---|---|---|----|
| b   | 0 | 1 | 3 | 6 | 10 |
| a   |   | 2 | 4 | 7 | 11 |
| b   |   |   | 5 | 8 | 12 |
| a   |   |   |   | 9 | 13 |
| d   |   |   |   |   | 14 |

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        max_substr = ""
        dp = [[False]*len(s) for _ in range(len(s))]
        for j in range(len(s)):
            for i in range(j+1):
                if i == j:
                    dp[i][j] = True
                elif j-i == 1:
                    dp[i][j] = s[i] == s[j]
                else:
                    dp[i][j] = (s[i] == s[j]) and dp[i+1][j-1]
                
                if dp[i][j] and j-i+1 > len(max_substr):
                    max_substr = s[i:j+1]
        return max_substr
```

## Palindrome Ad-hoc Solution

Set $i$ to be the center of palindrome sunstring, $j$ to be the radius, we can expand from center to side to valid the substring:
1. Odd length substring:
    - Expand from $(i, i)$, in which $j=0$, $L=i-j, R=i+j$, length of the substring is $2j+1$, $r\_{max} = i$
2. Even length substring:
    - Expand from $(i-1, i)$, in which $j=0$, $L=i-1-j, R=i+j$, length of the substring is $2j+2$, $r\_{max} = i$

| i-j |   |   | i-1 |  i  | i+1 |   |   | i+r |   |
|:---:|:-:|:-:|:---:|:---:|:---:|:-:|:-:|:---:|:-:|
|  1  | 2 | 3 |  4  |  5  |  4  | 3 | 2 |  1  |   |
|     |   |   |     | i-1 |  i  |   |   |     |   |
|  1  | 2 | 3 |  4  |  5  |  5  | 4 | 3 |  2  | 1 |

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        n, max_len, max_str = len(s), 0, ""
        for i in range(n):
            for j in range(i+1):
                l, r = i-j, i+j
                if l>=0 and r<n and s[l]==s[r]:
                    if max_len < 2*j+1:
                        max_len = 2*j+1
                        max_str = s[l:r+1]
                else:
                    break

            for j in range(i+1):
                l, r = i-1-j, i+j
                if l>=0 and r<n and s[l]==s[r]:
                    if max_len < 2*j+2:
                        max_len = 2*j+2
                        max_str = s[l:r+1]
                else:
                    break
        
        return max_str
```

## Manacher Algorithm

Check wiki~ it is O(n)
