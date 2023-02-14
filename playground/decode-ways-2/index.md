# Decode Ways 2


<!--more-->

[Leetcode 639](https://leetcode.com/problems/decode-ways-ii/)

Same as [Decode Ways]({{< relref "decode-ways.md" >}}), we only need to add some cases on wildcard matching, in which we could match multiple patterns for `S[i]`

```python
class Solution:
    def numDecodings(self, s: str) -> int:
        one = {str(k):1 for k in range(1, 10)}
        one.update({'*':9})
        two = {str(k):1 for k in range(10, 27)}
        two.update({
            '*0': 2, '*1': 2, '*2': 2, '*3': 2, 
            '*4': 2, '*5': 2, '*6': 2, '*7': 1, 
            '*8': 1, '*9': 1, '1*': 9, '2*': 6, '**': 15
        })

        dp = [0]*len(s)
        for i in range(len(s)):
            match_one = one.get(s[i], 0)
            match_two = two.get(s[i-1:i+1], 0) if i-1>=0 else 0
            
            one_bit = dp[i-1]*match_one if i-1>=0 else match_one
            two_bit = dp[i-2]*match_two if i-2>=0 else match_two

            dp[i] = one_bit + two_bit
        
        return dp[-1] % (10**9+7)
```


