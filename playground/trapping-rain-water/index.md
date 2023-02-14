# Trapping Rain Water

<!--more-->

https://leetcode.com/problems/trapping-rain-water/

设第$i$个bar的高度为$h_i$, 其蓄水量为$v_i$, 一共有$n$个bar,

## 暴力解
对于每一个位置$i$, 考虑它头顶有多少水. 很容易发现它左边最高和右边最高的两个bar中比较低的那个决定了水位高度, 再减去当前bar的高度就是水的体积. 也就是:
$$v_i = \min (\max\limits_{0\leq l<i} h_l, \max\limits_{i<r<n} h_r) - h_i$$

对所有位置求和就得到了结果. 注意处理负数情况为0. 时间复杂度 $O(n^2)$.
可以发现如果提前维护好每一个位置左边和右边最大的bar, 就可以 $O(n)$ 时间复杂度解决本题.

## "水位"法
对此, 还有一个非常直观的的解法：

想象水从两边流向中间, 水位在逐渐上升, 水会逐渐淹没比较低的bar,并且在淹没其之后流向相邻的bar, 从两边逐渐向中间蔓延. 
设水位高度为$level$, 并且水流向了$h_j$.
1. $level > h_j$, 水位比bar高, 水会淹没$h_j$, 此时$j$位置可以拦截的水量即为$v_j = level - h_j$. 证明: 无论水是从左边还是右边流向位置$j$的, 它一定得先漫过左右两边最高的bar中比较低的bar, 否则不能流到此位置.
2. $level \leq h_j$, 那么水平面遇到了更高的bar, 它不会容纳水, 即$v_j = 0$. 因为水从某一侧来就说明$h_j$的高度比那一侧最高的bar更高.

### 优先队列解
维护水位高度和bar组成的优先队列, 初始时水位为0, 左右两边的bar入队列. 随着水位的上升, 有:

1. 最小值出队列, 设为`i`, 同时更新水位 `level = max(level, height[i])`
2. 探索与`i`相邻的bar, 设为`j`, 若未访问过, 压入队列, 累加该位置的蓄水量 `v += level - height[j]`
3. 重复1~2步骤直至队列为空.

可以发现, 这就是一个从边界开始的bfs, 对每个元素计算其蓄水量并累加得到结果.

```cpp
int trap(vector<int>& height) {
        int n = height.size(), level = 0; int volume = 0;
        vector<int> visited(n);
        priority_queue<pair<int, int>> q;
        
        // 首尾入栈
        q.push({-height[0], 0});
        q.push({-height[n-1], n-1});
        visited[0] = 1; visited[n-1] = 1;

        while (!q.empty()) {
            auto i = q.top(); q.pop();
            level = max(level, -i.first);
            for (int j : {i.second-1, i.second+1}) {
                if (j >= 0 && j < n && !visited[j]) {
                    visited[j] = 1;
                    if (height[j] < level) {
                        volume += level - height[j];
                    }
                    q.push({-height[j], j});
                }
            }
        }
        return volume;
    }
```

这里用了几个小技巧. 一个是优先队列比较`pair<int, int>`中的第一个元素, 所以把高度放在前, 把index放在后. 其次是默认的优先队列是最大堆, 为了变成最小堆, 我们压入负的高度.

### 双指针解
双指针$l, r$从两边向中间遍历, 双指针其实就是水位法中优先队列里的两个元素. 双指针的移动顺序, 其实就是水浸没bar的顺序. 所以逻辑是完全一致的.

```cpp
class Solution {
public:
    int trap(vector<int>& height) {
        int l = 0; int r = height.size() - 1; // l和r对应队列中的index
        int lmax = 0; int rmax = 0; int v = 0; // lmax和rmax代表了左右两边水位的高度, 对应level
        while (l <= r) {
            int hl = height[l]; int hr = height[r]; // hl和hr对应队列中的height
            if (hl <= hr) {  // 左边水位比较低, 先处理左边
                lmax = max(lmax, hl); // 更新水位
                if (lmax > hl) { 
                    v += lmax - hl; // 水位浸没hl
                }
                l++; // 对应hl入队列
            } else {  // 右边水位比较低, 先处理右边
                rmax = max(rmax, hr);
                if (rmax > hr) {
                    v += rmax - hr; // 水位浸没hr
                }
                r--; // 对应hr入队列
            }
        }
        return v;
    }
};
```

## 二维拓展
Extend: https://leetcode.com/problems/trapping-rain-water-ii/
这道题拓展到了2维，其实使用"水位法"依然可解。

想象水从四周流向中间，水位逐渐上升，每当水位升至与某一个$h_i$齐平的时候，它就会"溢出"并且蔓延到和$h_i$接壤的位置，此时需要检查水可不可以"漫"过去。如果接壤的$h_j < h_i$，$h_j$之上可以容纳水的体积就确定为$h_i - h_j$. 因为对$j$来说，在已经把它包围的拓扑里，最低的值是$h_i$

所以和上一题把两边的bar入栈类似的, 先把最外围一圈入栈, 然后和1维情况一样了. [演示动画](https://www.youtube.com/watch?v=cJayBq38VYw)

```cpp
// 顺时针遍历周围的元素
int drow[4] = {0, -1, 0, 1};
int dcol[4] = {-1, 0, 1, 0};

int trapRainWater(vector<vector<int>>& h) {
    int m = h.size();
    int n = h[0].size();        
    int level = 0; int volume = 0;
    priority_queue<pair<int, pair<int, int>>> q; // {-height, {row, col}}
    vector<vector<int>> visited(m, vector<int>(n));
    // 最外层一圈入堆
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (i == 0 || j == 0 || i == m-1 || j == n-1) {
                q.push({-h[i][j], {i, j}});
                visited[i][j] = 1;
            }
        }
    }

    while (!q.empty()) {
        auto cur = q.top(); q.pop();
        level = max(level, -cur.first);
        
        for (int i = 0; i < 4; i++) {  // 遍历邻居
            int r2 = cur.second.first  + drow[i];
            int c2 = cur.second.second + dcol[i];
            
            if (r2 >= 0 && r2 < m && c2 >=0 && c2 < n && !visited[r2][c2]) {
                int h2 = h[r2][c2];
                if (h2 < level) { volume += level - h2; }
                q.push({-h2, {r2, c2}});
                visited[r2][c2] = 1;
            }
        }
    }
    return volume;
}
```
