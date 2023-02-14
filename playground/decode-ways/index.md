# Decode Ways


<!--more-->

[Leetcode 91](https://leetcode.com/problems/decode-ways/)

This is a 1-d DP. Set $dp[i]$ to be the decode ways for `s[:i+1]`, we have:

$$ dp[i] = dp[i-2] \cdot M^{1}\_{i} + dp[i-1] \cdot M^{2}\_{i} $$

In which,

$$ \begin{aligned} M^{1}\_{i} &= | \lbrace S[i] \in A^{1} \rbrace | \\\ M^{2}\_{i} &= | \lbrace S[i-1:i+1] \in A^{2} \rbrace \end{aligned} $$

 - $M^{1}\_{i}$ means how many ways to match the 1-character decoders $A^1$ with $S[i]$. In this problem we have either 1($S[i] \in A^{1}$) or 0(not match, or not exists)
 - $M^{2}\_{i}$ means how many ways to match the 2-character decoders $A^2$ with $S[i-1:i+1]$. In this problem we have either 1($S[i] \in A^{1}$) or 0(not match, or not exists)

```python
class Solution:
    def numDecodings(self, s: str) -> int:
        one = {str(k):1 for k in range(1, 10)}
        two = {str(k):1 for k in range(10, 27)}

        dp = [0]*len(s)
        for i in range(len(s)):
            match_one = one.get(s[i], 0)
            match_two = two.get(s[i-1:i+1], 0) if i-1>=0 else 0
            
            one_bit = dp[i-1]*match_one if i-1>=0 else match_one
            two_bit = dp[i-2]*match_two if i-2>=0 else match_two

            dp[i] = one_bit + two_bit
        
        return dp[-1]
```

Notes
1. Use `get()` to take care of cases when `s[i]` is not in `one`
2. Take care of the corner cases when $i <= 2$, in which we can't get dp values.
3. Follow Up: [Decode Ways 2]({{< relref "decode-ways-2.md" >}})
