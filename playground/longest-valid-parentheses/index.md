# Longest Valid Parentheses


<!--more-->

## Description

Given a string containing just the characters '(' and ')', find the length of the longest valid (well-formed) parentheses substring.

Example 1:
```
Input: "(()"
Output: 2
Explanation: The longest valid parentheses substring is "()"
```
Example 2:
```
Input: ")()())"
Output: 4
Explanation: The longest valid parentheses substring is "()()"
```

## Dynamic Programming Solution

Let $dp[i]$ to be the **length** of the longest valid parentheses which **ends at** $i$'th position.

> $s[i]$ must be `)` to have a valid parentheses, else $dp[i] = 0$

Depends on `s[i-1]`, we have 2 situations:

1. $s[i-1] = ($

| ... | i-2     | i-1 | i     |
|-----|---------|-----|-------|
| ... | .       | (   | )     |
|     | dp[i-2] | 0   | dp[i] |

```
dp[i] = dp[i-2] + 2
```

2. $s[i-1] = )$
we have 2 situations, only needs to consider $s[i-dp[i-1]-1] = ($

|  i-dp[i-1]-2  | i-dp[i-1]-1 | i-dp[i-1] | ... | i-1     | i     |
|---------------|-------------|-----------|-----|---------|-------|
|       ?       |      (      | (         | ... | )       | )     |
|dp[i-dp[i-1]-2]|      0      | 0         | ... | dp[i-1] | dp[i] |

```
dp[i] = dp[i-dp[i-1]-2] + dp[i-1] + 2
```


Combining all the situations above, we have:

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        n = len(s)
        dp = [0]*n
        _max = 0
        for i in range(n):
            if s[i] == ')' and (i-1>=0):
                if s[i-1] == '(':
                    dp[i] = (dp[i-2] if (i-2>=0) else 0) + 2
                    _max = max(_max, dp[i])
                else:
                    if (i-dp[i-1]-1>=0) and s[i-dp[i-1]-1] == '(':
                        dp[i] = dp[i-1] + 2 + (dp[i-dp[i-1]-2] if (i-dp[i-1]-2>=0) else 0)
                        _max = max(_max, dp[i])

        return _max
```
Though the DP solution is not quite intuitive for me.

## Stack Solution

Maintain a stack, which borrows the idea from [Valid Parentheses]({{< relref "valid-parentheses.md" >}})

1. When a `(` arrives, push it into the stack
2. When a `)` arrives, compare it with the top of stack, if it forms a pair, pop it, else push it.
3. After the whole process, the items left in the stack are those not having valid pairs. On the other side, other elements are valid pairs. We just need to find longest space between those elements in the stack. So we push **index of the elements**, instead of the element itself.

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        stack: List[int] = []
        max_len: int = 0
        for i in range(len(s)):
            if s[i] == '(':
                stack.append(i)
            elif s[i] == ')':
                if stack and s[stack[-1]] == '(':
                    stack.pop()
                    last_not_paired = stack[-1] if stack else -1
                    max_len = max(max_len, i-last_not_paired)
                else:
                    stack.append(i)
        return max_len
```

Notes
1. We generate the max length **on the fly**. Each time a pair generated, we calculate space between current element and last element remains in the stack, which is the **last index that has not been paired** yet, which is the current length of the generated valid substring.
2. Watch out for corner case: if the stack is empty, we cannot find last element that has not been pairs, just put -1 to it shoud be fine.
