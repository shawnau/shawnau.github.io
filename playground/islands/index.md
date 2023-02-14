# 岛屿问题


总结了一些01矩阵组成的岛屿问题. 基本上可以通过dfs解决, 但也可以用并查集处理.
其中200是我做的第一道并查集题, 印象很深

 - [200 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/): 最基本的dfs框架, 不需要返回值
 - [1254 统计封闭岛屿的数目](https://leetcode.cn/problems/number-of-closed-islands/): 200的变体, 特判一下与边界接壤的岛屿, 把他们去掉
 - [463 岛屿的周长](https://leetcode-cn.com/problems/island-perimeter/): dfs需要返回周长
 - [695 岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island/): dfs需要返回面积
 - [827 最大人工岛](https://leetcode-cn.com/problems/making-a-large-island/): dfs在标记岛屿的同时返回面积

参考:
[岛屿类问题的通用解法、DFS 遍历框架](https://leetcode.cn/problems/number-of-islands/solution/dao-yu-lei-wen-ti-de-tong-yong-jie-fa-dfs-bian-li-/)
