# Valid Parentheses


<!--more-->

## Description

Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

An input string is valid if:

1. Open brackets must be closed by the same type of brackets.
2. Open brackets must be closed in the correct order.

Note that an empty string is also considered valid.

## Stack Solution

Keep a stack
1. When a `(` arrives, push it into the stack
2. When a `)` arrives, compare it with the top of stack, if it forms a pair, pop it, else push it.

```python
class Solution:
    def isValid(self, s: str) -> bool:
        left_set = {'(', '{', '['}
        right_set = {')', '}', ']'}
        get_pair = {
            ')': '(',
            '}': '{',
            ']': '['
        }
        stack = []
        for p in s:
            if p in left_set:
                stack.append(p)
            else:
                if stack and stack[-1] == get_pair[p]:
                    stack.pop()
                else:
                    stack.append(p)
        return len(stack) == 0
```

Notes
1. We use a dict to check if the pair is valid.
2. Watch out for corner case before indexig.
3. [Follow up]({{< relref "longest-valid-parentheses.md" >}})
