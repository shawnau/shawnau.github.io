# Unique Paths 3

Walk over all grids without duplicate, with obstacles

<!--more-->

## Description

See [LeetCode 980](https://leetcode.com/problems/unique-paths-iii/)

## DFS Backtracking Solution

Unlike the DP solutions to Unique path 1 and 2, we cannot reuse calculated paths cuz we can walk forth and back into the same arange (i, j). And the start and end position varies. Using a DFS.

```python
class Solution:
    def uniquePathsIII(self, grid: List[List[int]]) -> int:
        n_row, n_col = len(grid), len(grid[0])
        self.count, empty = 0, 1
        # find start and end, count empty grids
        for i in range(n_row):
            for j in range(n_col):
                if grid[i][j] == 0:
                    empty += 1
                if grid[i][j] == 1:
                    start = [i, j]
                if grid[i][j] == 2:
                    end = [i, j]
        
        def search(i, j, empty):
            if i in range(n_row) and j in range(n_col) and grid[i][j] >= 0:
                if ([i, j] == end) and (empty == 0):
                    self.count += 1
                    return
                
                grid[i][j] = -2
                search(i-1, j, empty-1)
                search(i+1, j, empty-1)
                search(i, j-1, empty-1)
                search(i, j+1, empty-1)
                grid[i][j] = 0
        
        search(start[0], start[1], empty)
        return self.count
```

Note
1. We initialize empty with 1, cuz the start of the search counts.
2. Search in a grid is frequently used, we need to restore the element after search.
3. The search function could be memorized to accelerate.
