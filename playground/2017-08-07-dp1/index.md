# 某动态规划小题

好像是谷歌面试题...中比较弱智的

<!--more-->

 - 输入: 一个m*n矩阵ary, 矩阵元素只有0和1
 - 输出: 寻找从左上到右下的连续路径中最长的路径长度(也就是只能往右或往下或往右下走)
 - 定义: 只要1的周围8个点有1就定义为连续路径

1. 状态矩阵定义: $dp[i][j]$代表在$ary[:i][:j]$子矩阵中, 经过点$arr[i][j]$的路径的最大值
2. 状态转移方程:
 $$ dp[i][j] = \begin{cases} max \lbrace & dp[i-1][j], \\\
 & dp[i][j-1], \\\
 & dp[i-1][j-1] \rbrace + 1,  & ary[i][j] = 1 \\\
0, && ary[i][j] = 0
\end{cases}$$
3. 首先计算出dp第一行第一列的值, 再从左向右从上到下计算整个dp数组, 最后统计最大值即可

```cpp
#include <iostream>
#include <vector>
using namespace std;

class Solution
{
public:
    int dp_sol(vector<vector<int> > & ary)
    {
        int max_route = 0;
        auto m = ary.size();
        auto n = ary[0].size();
        vector<vector<int>> dp(m, vector<int>(n, 0));
        // init 1st row
        for (int i = 0; i < n; ++i) {
            if (ary[0][i] == 1) {
                if ((i != 0) && (ary[0][i-1] == 1)) {
                    dp[0][i] = dp[0][i-1] + 1;
                } else {dp[0][i] = 1;}
            }
        }
        // init 1st col
        for (int i = 0; i < m; ++i) {
            if (ary[i][0] == 1) {
                if ((i != 0) && (ary[i-1][0] == 1)) {
                    dp[i][0] = dp[i-1][0] + 1;
                } else {dp[i][0] = 1;}
            }
        }
        // dp
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                if (ary[i][j] == 1) {
                    dp[i][j] = max(max(dp[i-1][j], dp[i][j-1]), dp[i-1][j-1]) + 1;
                }
            }
        }
        // find max
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
            if (dp[i][j] > max_route) {max_route = dp[i][j];}
            }
        }
        return max_route;
    }
};

int main() {
    vector< vector<int> >
    a = {{0, 1, 1, 0},
         {1, 0, 0, 0},
         {1, 0, 1, 0},
         {1, 0, 0, 1}};
    Solution s = Solution();
    cout<<s.dp_sol(a)<<endl;
    return 0;
}
```

拓展:
如果允许往任意方向走, 那就成了一个搜索最大的连通分量的题
