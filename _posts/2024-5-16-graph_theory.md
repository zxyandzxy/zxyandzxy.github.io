---
layout: post
title: "图论"
date: 2024-5-16
tags: [algorithm]
comments: true
author: zxy
---

### 深度优先搜索理论基础

```
深搜三部曲：
1.确认递归函数，参数
vector<vector<int>> result; // 保存符合条件的所有路径
vector<int> path; // 起点到终点的路径
void dfs (图，目前搜索的节点)

2.确认终止条件
if (终止条件) {
    存放结果;
    return;
}

3.处理目前搜索节点出发的路径
for (选择：本节点所连接的其他节点) {
    处理节点;
    dfs(图，选择的节点); // 递归
    回溯，撤销处理结果
}
```

### 797.所有可能的路径

```
给你一个有 n 个节点的 有向无环图（DAG），请你找出所有从节点 0 到节点 n-1 的路径并输出（不要求按特定顺序）
graph[i] 是一个从节点 i 可以访问的所有节点的列表（即从节点 i 到节点 graph[i][j]存在一条有向边）。

n == graph.length
2 <= n <= 15
0 <= graph[i][j] < n
graph[i][j] != i（即不存在自环）
graph[i] 中的所有元素 互不相同
保证输入为 有向无环图（DAG）
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20221203135439.png)

```
其实回溯算法就是深度优先搜索算法的一种
1.确认递归函数，参数
参数：当前访问到的节点，图

2.确认终止条件
当当前访问节点是n - 1号节点时，就终止了

3.处理目前搜索节点出发的路径
遍历当前节点的所有后续节点，继续探索
```

```cpp
class Solution {
private:
    vector<vector<int>> result; // 收集符合条件的路径
    vector<int> path; // 0节点到终点的路径
    // x：目前遍历的节点
    // graph：存当前的图
    void dfs (vector<vector<int>>& graph, int x) {
        // 要求从节点 0 到节点 n-1 的路径并输出，所以是 graph.size() - 1
        if (x == graph.size() - 1) { // 找到符合条件的一条路径
            result.push_back(path);
            return;
        }
        for (int i = 0; i < graph[x].size(); i++) { // 遍历节点n链接的所有节点
            path.push_back(graph[x][i]); // 遍历到的节点加入到路径中来
            dfs(graph, graph[x][i]); // 进入下一层递归
            path.pop_back(); // 回溯，撤销本节点
        }
    }
public:
    vector<vector<int>> allPathsSourceTarget(vector<vector<int>>& graph) {
        path.push_back(0); // 无论什么路径已经是从0节点出发
        dfs(graph, 0); // 开始遍历
        return result;
    }
};
```

### 200. 岛屿数量

```
给你一个由 '1'（陆地）和 '0'（水）组成的的二维网格，请你计算网格中岛屿的数量。
岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。
此外，你可以假设该网格的四条边均被水包围。
提示：
m == grid.length
n == grid[i].length
1 <= m, n <= 300
grid[i][j] 的值为 '0' 或 '1'
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20220726093256.png)

```cpp
深搜版：
class Solution {
private:
    int dir[4][2] = {0, 1, 1, 0, -1, 0, 0, -1}; // 四个方向
    void dfs(vector<vector<char>>& grid, vector<vector<bool>>& visited, int x, int y) {
        for (int i = 0; i < 4; i++) {
            int nextx = x + dir[i][0];
            int nexty = y + dir[i][1];
            if (nextx < 0 || nextx >= grid.size() || nexty < 0 || nexty >= grid[0].size()) continue;  // 越界了，直接跳过
            if (!visited[nextx][nexty] && grid[nextx][nexty] == '1') { // 没有访问过的 同时 是陆地的

                visited[nextx][nexty] = true;
                dfs(grid, visited, nextx, nexty);
            }
        }
    }
public:
    int numIslands(vector<vector<char>>& grid) {
        int n = grid.size(), m = grid[0].size();
        vector<vector<bool>> visited = vector<vector<bool>>(n, vector<bool>(m, false));

        int result = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (!visited[i][j] && grid[i][j] == '1') {
                    visited[i][j] = true;
                    result++; // 遇到没访问过的陆地，+1
                    dfs(grid, visited, i, j); // 将与其链接的陆地都标记上 true
                }
            }
        }
        return result;
    }
};
```

```cpp
广度优先搜索
class Solution {
private:
int dir[4][2] = {0, 1, 1, 0, -1, 0, 0, -1}; // 四个方向
void bfs(vector<vector<char>>& grid, vector<vector<bool>>& visited, int x, int y) {
    queue<pair<int, int>> que;
    que.push({x, y});
    visited[x][y] = true; // 只要加入队列，立刻标记
    while(!que.empty()) {
        pair<int ,int> cur = que.front(); que.pop();
        int curx = cur.first;
        int cury = cur.second;
        for (int i = 0; i < 4; i++) {
            int nextx = curx + dir[i][0];
            int nexty = cury + dir[i][1];
            if (nextx < 0 || nextx >= grid.size() || nexty < 0 || nexty >= grid[0].size()) continue;  // 越界了，直接跳过
            if (!visited[nextx][nexty] && grid[nextx][nexty] == '1') {
                que.push({nextx, nexty});
                visited[nextx][nexty] = true; // 只要加入队列立刻标记
            }
        }
    }
}
public:
    int numIslands(vector<vector<char>>& grid) {
        int n = grid.size(), m = grid[0].size();
        vector<vector<bool>> visited = vector<vector<bool>>(n, vector<bool>(m, false));

        int result = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (!visited[i][j] && grid[i][j] == '1') {
                    result++; // 遇到没访问过的陆地，+1
                    bfs(grid, visited, i, j); // 将与其链接的陆地都标记上 true
                }
            }
        }
        return result;
    }
};

```

### 695. 岛屿的最大面积

```
给你一个大小为 m x n 的二进制矩阵 grid 。
岛屿 是由一些相邻的 1 (代表土地) 构成的组合，这里的「相邻」要求两个 1 必须在 水平或者竖直的四个方向上 相邻。你可以假设 grid 的四个边缘都被 0（代表水）包围着。
岛屿的面积是岛上值为 1 的单元格的数目。
计算并返回 grid 中最大的岛屿面积。如果没有岛屿，则返回面积为 0 。

输入：grid = [[0,0,1,0,0,0,0,1,0,0,0,0,0],[0,0,0,0,0,0,0,1,1,1,0,0,0],[0,1,1,0,1,0,0,0,0,0,0,0,0],[0,1,0,0,1,1,0,0,1,0,1,0,0],[0,1,0,0,1,1,0,0,1,1,1,0,0],[0,0,0,0,0,0,0,0,0,0,1,0,0],[0,0,0,0,0,0,0,1,1,1,0,0,0],[0,0,0,0,0,0,0,1,1,0,0,0,0]]
输出：6
解释：答案不应该是 11 ，因为岛屿只能包含水平或垂直这四个方向上的 1 。
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20220729111528.png)

```cpp
class Solution {
private:
    int count;
    int dir[4][2] = {0, 1, 1, 0, -1, 0, 0, -1}; // 四个方向
    void bfs(vector<vector<int>>& grid, vector<vector<bool>>& visited, int x, int y) {
        queue<int> que;
        que.push(x);
        que.push(y);
        visited[x][y] = true; // 加入队列就意味节点是陆地可到达的点
        count++;
        while(!que.empty()) {
            int xx = que.front();que.pop();
            int yy = que.front();que.pop();
            for (int i = 0 ;i < 4; i++) {
                int nextx = xx + dir[i][0];
                int nexty = yy + dir[i][1];
                if (nextx < 0 || nextx >= grid.size() || nexty < 0 || nexty >= grid[0].size()) continue; // 越界
                if (!visited[nextx][nexty] && grid[nextx][nexty] == 1) { // 节点没有被访问过且是陆地
                    visited[nextx][nexty] = true;
                    count++;
                    que.push(nextx);
                    que.push(nexty);
                }
            }
        }
    }

public:
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        vector<vector<bool>> visited = vector<vector<bool>>(n, vector<bool>(m, false));
        int result = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (!visited[i][j] && grid[i][j] == 1) {
                    count = 0;
                    bfs(grid, visited, i, j); // 将与其链接的陆地都标记上 true
                    result = max(result, count);
                }
            }
        }
        return result;
    }
};
```

### 1020. 飞地的数量

```
给你一个大小为 m x n 的二进制矩阵 grid ，其中 0 表示一个海洋单元格、1 表示一个陆地单元格。
一次 移动 是指从一个陆地单元格走到另一个相邻（上、下、左、右）的陆地单元格或跨过 grid 的边界。
返回网格中 无法 在任意次数的移动中离开网格边界的陆地单元格的数量。

示例：
输入：grid = [[0,0,0,0],[1,0,1,0],[0,1,1,0],[0,0,0,0]]
输出：3
解释：有三个 1 被 0 包围。一个 1 没有被包围，因为它在边界上。
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20220830100710.png)

```cpp
先遍历四周边界，如果是1，就去广度优先搜索，将连接的1全置为0
最后遍历1遍整个数组，统计1的个数就是答案
class Solution {
private:
int count = 0;
int dir[4][2] = {0, 1, 1, 0, -1, 0, 0, -1}; // 四个方向
void bfs(vector<vector<int>>& grid, int x, int y) {
    queue<pair<int, int>> que;
    que.push({x, y});
    grid[x][y] = 0; // 只要加入队列，立刻标记
    count++;
    while(!que.empty()) {
        pair<int ,int> cur = que.front(); que.pop();
        int curx = cur.first;
        int cury = cur.second;
        for (int i = 0; i < 4; i++) {
            int nextx = curx + dir[i][0];
            int nexty = cury + dir[i][1];
            if (nextx < 0 || nextx >= grid.size() || nexty < 0 || nexty >= grid[0].size()) continue;  // 越界了，直接跳过
            if (grid[nextx][nexty] == 1) {
                que.push({nextx, nexty});
                count++;
                grid[nextx][nexty] = 0; // 只要加入队列立刻标记
            }
        }
    }

}

public:
    int numEnclaves(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        // 从左侧边，和右侧边 向中间遍历
        for (int i = 0; i < n; i++) {
            if (grid[i][0] == 1) bfs(grid, i, 0);
            if (grid[i][m - 1] == 1) bfs(grid, i, m - 1);
        }
        // 从上边和下边 向中间遍历
        for (int j = 0; j < m; j++) {
            if (grid[0][j] == 1) bfs(grid, 0, j);
            if (grid[n - 1][j] == 1) bfs(grid, n - 1, j);
        }
        count = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (grid[i][j] == 1) bfs(grid, i, j);
            }
        }
        return count;
    }
};
```

### 130. 被围绕的区域

```
给你一个 m x n 的矩阵 board ，由若干字符 'X' 和 'O' ，找到所有被 'X' 围绕的区域，并将这些区域里所有的 'O' 用 'X' 填充。

输入：board = [["X","X","X","X"],["X","O","O","X"],["X","X","O","X"],["X","O","X","X"]]
输出：[["X","X","X","X"],["X","X","X","X"],["X","X","X","X"],["X","O","X","X"]]
解释：被围绕的区间不会存在于边界上，换句话说，任何边界上的 'O' 都不会被填充为 'X'。 任何不在边界上，或不与边界上的 'O' 相连的 'O' 最终都会被填充为 'X'。如果两个元素在水平或垂直方向相邻，则称它们是“相连”的。
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20220901104745.png)

```
从地图周边出发，将周边空格相邻的'O'都做上标记，然后在遍历一遍地图，遇到 'O' 且没做过标记的，那么都是地图中间的'O'，全部改成'X'就行。
```

```cpp
class Solution {
private:
    int dir[4][2] = {-1, 0, 0, -1, 1, 0, 0, 1}; // 保存四个方向
    void dfs(vector<vector<char>>& board, int x, int y) {
        board[x][y] = 'A';
        for (int i = 0; i < 4; i++) { // 向四个方向遍历
            int nextx = x + dir[i][0];
            int nexty = y + dir[i][1];
            // 超过边界
            if (nextx < 0 || nextx >= board.size() || nexty < 0 || nexty >= board[0].size()) continue;
            // 不符合条件，不继续遍历
            if (board[nextx][nexty] == 'X' || board[nextx][nexty] == 'A') continue;
            dfs (board, nextx, nexty);
        }
        return;
    }

public:
    void solve(vector<vector<char>>& board) {
        int n = board.size(), m = board[0].size();
        // 步骤一：
        // 从左侧边，和右侧边 向中间遍历
        for (int i = 0; i < n; i++) {
            if (board[i][0] == 'O') dfs(board, i, 0);
            if (board[i][m - 1] == 'O') dfs(board, i, m - 1);
        }

        // 从上边和下边 向中间遍历
        for (int j = 0; j < m; j++) {
            if (board[0][j] == 'O') dfs(board, 0, j);
            if (board[n - 1][j] == 'O') dfs(board, n - 1, j);
        }
        // 步骤二：
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (board[i][j] == 'O') board[i][j] = 'X';
                if (board[i][j] == 'A') board[i][j] = 'O';
            }
        }
    }
};
```

### 417. 太平洋大西洋水流问题

```
有一个 m × n 的矩形岛屿，与 太平洋 和 大西洋 相邻。 “太平洋” 处于大陆的左边界和上边界，而 “大西洋” 处于大陆的右边界和下边界。

这个岛被分割成一个由若干方形单元格组成的网格。给定一个 m x n 的整数矩阵 heights ， heights[r][c] 表示坐标 (r, c) 上单元格 高于海平面的高度 。

岛上雨水较多，如果相邻单元格的高度 小于或等于 当前单元格的高度，雨水可以直接向北、南、东、西流向相邻单元格。水可以从海洋附近的任何单元格流入海洋。

返回网格坐标 result 的 2D 列表 ，其中 result[i] = [ri, ci] 表示雨水从单元格 (ri, ci) 流动 既可流向太平洋也可流向大西洋 。
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20230129103212.png)

```
输入: heights = [[1,2,2,3,5],[3,2,3,4,4],[2,4,5,3,1],[6,7,1,4,5],[5,1,1,2,4]]
输出: [[0,4],[1,3],[1,4],[2,2],[3,0],[3,1],[4,0]]

m == heights.length
n == heights[r].length
1 <= m, n <= 200
0 <= heights[r][c] <= 10^5
```

```cpp
最直接的暴力，遍历每一个元素，通过广度优先搜索判断它能不能到两片海洋
时间复杂度是 (n * m) * (n * m)  会超时

反过来想，从两片海洋边上的节点逆流而上，将能到的所有节点标记为1
如果从两片海洋边上开始走，都能标记为1，那就能流到两片海洋
时间复杂度 2 * n * m + n * m
class Solution {
private:
    int dir[4][2] = {-1, 0, 0, -1, 1, 0, 0, 1}; // 保存四个方向

    // 从低向高遍历，注意这里visited是引用，即可以改变传入的pacific和atlantic的值
    void dfs(vector<vector<int>>& heights, vector<vector<bool>>& visited, int x, int y) {
        if (visited[x][y]) return;
        visited[x][y] = true;
        for (int i = 0; i < 4; i++) { // 向四个方向遍历
            int nextx = x + dir[i][0];
            int nexty = y + dir[i][1];
            // 超过边界
            if (nextx < 0 || nextx >= heights.size() || nexty < 0 || nexty >= heights[0].size()) continue;
            // 高度不合适，注意这里是从低向高判断
            if (heights[x][y] > heights[nextx][nexty]) continue;

            dfs (heights, visited, nextx, nexty);
        }
        return;

    }

public:

    vector<vector<int>> pacificAtlantic(vector<vector<int>>& heights) {
        vector<vector<int>> result;
        int n = heights.size();
        int m = heights[0].size(); // 这里不用担心空指针，题目要求说了长宽都大于1

        // 记录从太平洋边出发，可以遍历的节点
        vector<vector<bool>> pacific = vector<vector<bool>>(n, vector<bool>(m, false));

        // 记录从大西洋出发，可以遍历的节点
        vector<vector<bool>> atlantic = vector<vector<bool>>(n, vector<bool>(m, false));

        // 从最上最下行的节点出发，向高处遍历
        for (int i = 0; i < n; i++) {
            dfs (heights, pacific, i, 0); // 遍历最左列，接触太平洋
            dfs (heights, atlantic, i, m - 1); // 遍历最右列，接触大西
        }

        // 从最左最右列的节点出发，向高处遍历
        for (int j = 0; j < m; j++) {
            dfs (heights, pacific, 0, j); // 遍历最上行，接触太平洋
            dfs (heights, atlantic, n - 1, j); // 遍历最下行，接触大西洋
        }
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                // 如果这个节点，从太平洋和大西洋出发都遍历过，就是结果
                if (pacific[i][j] && atlantic[i][j]) result.push_back({i, j});
            }
        }
        return result;
    }
};

```

### 827.最大人工岛

```
给你一个大小为 n x n 二进制矩阵 grid 。最多 只能将一格 0 变成 1 。
返回执行此操作后，grid 中最大的岛屿面积是多少？
岛屿 由一组上、下、左、右四个方向相连的 1 形成。

示例 1:
输入: grid = [[1, 0], [0, 1]]
输出: 3
解释: 将一格0变成1，最终连通两个小岛得到面积为 3 的岛屿。
```

```
首先通过一次深搜，将所有岛屿标记出来，并记录岛屿的大小
然后尝试将每一个0 变成 1，同时看一下它周围4个区域是否是已经标记过的岛屿，是就可以添加进来了
通过先记录所有岛屿，就能在O(1)时间内发现某个位置能不能将已有岛屿连接起来
```

```cpp
class Solution {
private:
    int count;
    int dir[4][2] = {0, 1, 1, 0, -1, 0, 0, -1}; // 四个方向
    void dfs(vector<vector<int>>& grid, vector<vector<bool>>& visited, int x, int y, int mark) {
        if (visited[x][y] || grid[x][y] == 0) return; // 终止条件：访问过的节点 或者 遇到海水
        visited[x][y] = true; // 标记访问过
        grid[x][y] = mark; // 给陆地标记新标签
        count++;
        for (int i = 0; i < 4; i++) {
            int nextx = x + dir[i][0];
            int nexty = y + dir[i][1];
            if (nextx < 0 || nextx >= grid.size() || nexty < 0 || nexty >= grid[0].size()) continue;  // 越界了，直接跳过
            dfs(grid, visited, nextx, nexty, mark);
        }
    }

public:
    int largestIsland(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        vector<vector<bool>> visited = vector<vector<bool>>(n, vector<bool>(m, false)); // 标记访问过的点
        unordered_map<int ,int> gridNum;
        int mark = 2; // 记录每个岛屿的编号
        bool isAllGrid = true; // 标记是否整个地图都是陆地
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (grid[i][j] == 0) isAllGrid = false;
                if (!visited[i][j] && grid[i][j] == 1) {
                    count = 0;
                    dfs(grid, visited, i, j, mark); // 将与其链接的陆地都标记上 true
                    gridNum[mark] = count; // 记录每一个岛屿的面积
                    mark++; // 记录下一个岛屿编号
                }
            }
        }
        if (isAllGrid) return n * m; // 如果都是陆地，返回全面积

        // 以下逻辑是根据添加陆地的位置，计算周边岛屿面积之和
        int result = 0; // 记录最后结果
        unordered_set<int> visitedGrid; // 标记访问过的岛屿
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                int count = 1; // 记录连接之后的岛屿数量
                visitedGrid.clear(); // 每次使用时，清空
                if (grid[i][j] == 0) {
                    for (int k = 0; k < 4; k++) {
                        int neari = i + dir[k][1]; // 计算相邻坐标
                        int nearj = j + dir[k][0];
                        if (neari < 0 || neari >= grid.size() || nearj < 0 || nearj >= grid[0].size()) continue;
                        if (visitedGrid.count(grid[neari][nearj])) continue; // 添加过的岛屿不要重复添加
                        // 把相邻四面的岛屿数量加起来
                        count += gridNum[grid[neari][nearj]];
                        visitedGrid.insert(grid[neari][nearj]); // 标记该岛屿已经添加过
                    }
                }
                result = max(result, count);
            }
        }
        return result;
    }
};
```

### 127. 单词接龙

```
字典 wordList 中从单词 beginWord 和 endWord 的 转换序列 是一个按下述规格形成的序列：
	序列中第一个单词是 beginWord 。
	序列中最后一个单词是 endWord 。
	每次转换只能改变一个字母。
	转换过程中的中间单词必须是字典 wordList 中的单词。
给你两个单词 beginWord 和 endWord 和一个字典 wordList ，找到从 beginWord 到 endWord 的 最短转换序列 中的 单词数目 。如果不存在这样的转换序列，返回 0。

输入：beginWord = "hit", endWord = "cog", wordList = ["hot","dot","dog","lot","log","cog"]
输出：5
解释：一个最短转换序列是 "hit" -> "hot" -> "dot" -> "dog" -> "cog", 返回它的长度 5。

1 <= beginWord.length <= 10
endWord.length == beginWord.length
1 <= wordList.length <= 5000
wordList[i].length == beginWord.length
beginWord、endWord 和 wordList[i] 由小写英文字母组成
beginWord != endWord
wordList 中的所有字符串 互不相同
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20210827175432.png)

```
本题只需要求出最短路径的长度就可以了，不用找出路径。
所以这道题要解决两个问题：
	图中的线是如何连在一起的
	起点和终点的最短路径长度
首先题目中并没有给出点与点之间的连线，而是要我们自己去连，条件是字符只能差一个，所以判断点与点之间的关系，要自己判断是不是差一个字符，如果差一个字符，那就是有链接。

然后就是求起点和终点的最短路径长度，这里无向图求最短路，广搜最为合适，广搜只要搜到了终点，那么一定是最短的路径。因为广搜就是以起点中心向四周扩散的搜索。
本题如果用深搜，会比较麻烦，要在到达终点的不同路径中选则一条最短路。 而广搜只要达到终点，一定是最短路。

另外需要有一个注意点：
	本题是一个无向图，需要用标记位，标记着节点是否走过，否则就会死循环！
	本题给出集合是数组型的，可以转成set结构，查找更快一些
```

```cpp
class Solution {
public:
    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
        // 将vector转成unordered_set，提高查询速度
        unordered_set<string> wordSet(wordList.begin(), wordList.end());
        // 如果endWord没有在wordSet出现，直接返回0
        if (wordSet.find(endWord) == wordSet.end()) return 0;
        // 记录word是否访问过
        unordered_map<string, int> visitMap; // <word, 查询到这个word路径长度>
        // 初始化队列
        queue<string> que;
        que.push(beginWord);
        // 初始化visitMap
        visitMap.insert(pair<string, int>(beginWord, 1));

        while(!que.empty()) {
            string word = que.front();
            que.pop();
            int path = visitMap[word]; // 这个word的路径长度
            for (int i = 0; i < word.size(); i++) {
                string newWord = word; // 用一个新单词替换word，因为每次置换一个字母
                for (int j = 0 ; j < 26; j++) {
                    newWord[i] = j + 'a';
                    if (newWord == endWord) return path + 1; // 找到了end，返回path+1
                    // wordSet出现了newWord，并且newWord没有被访问过
                    if (wordSet.find(newWord) != wordSet.end()
                            && visitMap.find(newWord) == visitMap.end()) {
                        // 添加访问信息
                        visitMap.insert(pair<string, int>(newWord, path + 1));
                        que.push(newWord);
                    }
                }
            }
        }
        return 0;
    }
};
```

### 841.钥匙和房间

```
有 N 个房间，开始时你位于 0 号房间。每个房间有不同的号码：0，1，2，...，N-1，并且房间里可能有一些钥匙能使你进入下一个房间。
在形式上，对于每个房间 i 都有一个钥匙列表 rooms[i]，每个钥匙 rooms[i][j] 由 [0,1，...，N-1] 中的一个整数表示，其中 N = rooms.length。 钥匙 rooms[i][j] = v 可以打开编号为 v 的房间。
最初，除 0 号房间外的其余所有房间都被锁住。
你可以自由地在房间之间来回走动。
如果能进入每个房间返回 true，否则返回 false。

示例 1：
输入: [[1],[2],[3],[]]
输出: true
解释: 我们从 0 号房间开始，拿到钥匙 1。 之后我们去 1 号房间，拿到钥匙 2。 然后我们去 2 号房间，拿到钥匙 3。 最后我们去了 3 号房间。 由于我们能够进入每个房间，我们返回 true。

示例 2：
输入：[[1,3],[3,0,1],[2],[0]]
输出：false
解释：我们不能进入 2 号房间。
```

```
拿一个数组所有房间是否访问过
从0号房间开始，去到所有能去的房间，通过bfs，将所有能去的房间访问1遍
最后判断是不是所有房间都访问过即可
```

```cpp
class Solution {
bool bfs(const vector<vector<int>>& rooms) {
    vector<int> visited(rooms.size(), 0); // 标记房间是否被访问过
    visited[0] = 1; //  0 号房间开始
    queue<int> que;
    que.push(0); //  0 号房间开始

    // 广度优先搜索的过程
    while (!que.empty()) {
        int key = que.front(); que.pop();
         vector<int> keys = rooms[key];
         for (int key : keys) {
             if (!visited[key]) {
                 que.push(key);
                 visited[key] = 1;
             }
         }
    }
    // 检查房间是不是都遍历过了
    for (int i : visited) {
        if (i == 0) return false;
    }
    return true;

}
public:
    bool canVisitAllRooms(vector<vector<int>>& rooms) {
        return bfs(rooms);
    }
};
```

### 463. 岛屿的周长

```
给定一个 row x col 的二维网格地图 grid ，其中：grid[i][j] = 1 表示陆地， grid[i][j] = 0 表示水域。
网格中的格子 水平和垂直 方向相连（对角线方向不相连）。整个网格被水完全包围，但其中恰好有一个岛屿（或者说，一个或多个表示陆地的格子相连组成的岛屿）。
岛屿中没有“湖”（“湖” 指水域在岛屿内部且不和岛屿周围的水相连）。格子是边长为 1 的正方形。网格为长方形，且宽度和高度均不超过 100 。计算这个岛屿的周长。
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20230829180848.png)

```
输入：grid = [[0,1,0,0],[1,1,1,0],[0,1,0,0],[1,1,0,0]]
输出：16
解释：它的周长是上面图片中的 16 个黄色的边

提示：
row == grid.length
col == grid[i].length
1 <= row, col <= 100
grid[i][j] 为 0 或 1
```

```cpp
遍历每一个空格，遇到岛屿，计算其上下左右的情况，遇到水域或者出界的情况，就可以计算边了。
class Solution {
public:
    int direction[4][2] = {0, 1, 1, 0, -1, 0, 0, -1};
    int islandPerimeter(vector<vector<int>>& grid) {
        int result = 0;
        for (int i = 0; i < grid.size(); i++) {
            for (int j = 0; j < grid[0].size(); j++) {
                if (grid[i][j] == 1) {
                    for (int k = 0; k < 4; k++) {       // 上下左右四个方向
                        int x = i + direction[k][0];
                        int y = j + direction[k][1];    // 计算周边坐标x,y
                        if (x < 0                       // i在边界上
                                || x >= grid.size()     // i在边界上
                                || y < 0                // j在边界上
                                || y >= grid[0].size()  // j在边界上
                                || grid[x][y] == 0) {   // x,y位置是水域
                            result++;
                        }
                    }
                }
            }
        }
        return result;
    }
};
```

### 并查集理论基础

```cpp
nt n = 1005; // n根据题目中节点数量而定，一般比节点数量大一点就好
vector<int> father = vector<int> (n, 0); // C++里的一种数组结构

// 并查集初始化
void init() {
    for (int i = 0; i < n; ++i) {
        father[i] = i;
    }
}
// 并查集里寻根的过程
int find(int u) {
    return u == father[u] ? u : father[u] = find(father[u]); // 路径压缩
}

// 判断 u 和 v是否找到同一个根
bool isSame(int u, int v) {
    u = find(u);
    v = find(v);
    return u == v;
}

// 将v->u 这条边加入并查集
void join(int u, int v) {
    u = find(u); // 寻找u的根
    v = find(v); // 寻找v的根
    if (u == v) return ; // 如果发现根相同，则说明在一个集合，不用两个节点相连直接返回
    father[v] = u;
}
```

1. 寻找根节点，函数：find(int u)，也就是判断这个节点的祖先节点是哪个
2. 将两个节点接入到同一个集合，函数：join(int u, int v)，将两个节点连在同一个根节点上
3. 判断两个节点是否在同一个集合，函数：isSame(int u, int v)，就是判断两个节点是不是同一个根节点

### 1971. 寻找图中是否存在路径

```
有一个具有 n 个顶点的 双向 图，其中每个顶点标记从 0 到 n - 1（包含 0 和 n - 1）。图中的边用一个二维整数数组 edges 表示，其中 edges[i] = [ui, vi] 表示顶点 ui 和顶点 vi 之间的双向边。 每个顶点对由 最多一条 边连接，并且没有顶点存在与自身相连的边。
请你确定是否存在从顶点 start 开始，到顶点 end 结束的 有效路径 。
给你数组 edges 和整数 n、start 和 end，如果从 start 到 end 存在 有效路径 ，则返回 true，否则返回 false 。

1 <= n <= 2 * 10^5
0 <= edges.length <= 2 * 10^5
edges[i].length == 2
0 <= ui, vi <= n - 1
ui != vi
0 <= start, end <= n - 1
不存在双向边
不存在指向顶点自身的边
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20220705101442.png)

```cpp
并查集：根据边构建并查集，最后判断start和end是不是有共同 祖先即可
class Solution {
private:
    int n = 200005; // 节点数量 20000
    vector<int> father = vector<int> (n, 0); // C++里的一种数组结构

    // 并查集初始化
    void init() {
        for (int i = 0; i < n; ++i) { father[i] = i;
        }
    }
    // 并查集里寻根的过程
    int find(int u) {
        return u == father[u] ? u : father[u] = find(father[u]);
    }

    // 判断 u 和 v是否找到同一个根
    bool isSame(int u, int v) {
        u = find(u);
        v = find(v);
        return u == v;
    }

    // 将v->u 这条边加入并查集
    void join(int u, int v) {
        u = find(u); // 寻找u的根
        v = find(v); // 寻找v的根
        if (u == v) return ; // 如果发现根相同，则说明在一个集合，不用两个节点相连直接返回
        father[v] = u;
    }

public:
    bool validPath(int n, vector<vector<int>>& edges, int source, int destination) {
        init();
        for (int i = 0; i < edges.size(); i++) {
            join(edges[i][0], edges[i][1]);
        }
        return isSame(source, destination);

    }
};
```

### 684.冗余连接

```
树可以看成是一个连通且 无环 的 无向 图。
给定往一棵 n 个节点 (节点值 1～n) 的树中添加一条边后的图。添加的边的两个顶点包含在 1 到 n 中间，且这条附加的边不属于树中已存在的边。图的信息记录于长度为 n 的二维数组 edges ，edges[i] = [ai, bi] 表示图中在 ai 和 bi 之间存在一条边。
请找出一条可以删去的边，删除后可使得剩余部分是一个有着 n 个节点的树。如果有多个答案，则返回数组 edges 中最后出现的边。

n == edges.length
3 <= n <= 1000
edges[i].length == 2
1 <= ai < bi <= edges.length
ai != bi
edges 中无重复元素
给定的图是连通的
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20210727150215.png)

```
可以从前向后遍历每一条边（因为优先让前面的边连上），边的两个节点如果不在同一个集合，就加入集合（即：同一个根节点）。
如果边的两个节点已经出现在同一个集合里，说明着边的两个节点已经连在一起了，再加入这条边一定就出现环了。
```

```cpp
class Solution {
private:
    int n = 1005; // 节点数量3 到 1000
    vector<int> father = vector<int> (n, 0); // C++里的一种数组结构

    // 并查集初始化
    void init() {
        for (int i = 0; i < n; ++i) {
            father[i] = i;
        }
    }
    // 并查集里寻根的过程
    int find(int u) {
        return u == father[u] ? u : father[u] = find(father[u]);
    }
    // 判断 u 和 v是否找到同一个根
    bool isSame(int u, int v) {
        u = find(u);
        v = find(v);
        return u == v;
    }
    // 将v->u 这条边加入并查集
    void join(int u, int v) {
        u = find(u); // 寻找u的根
        v = find(v); // 寻找v的根
        if (u == v) return ; // 如果发现根相同，则说明在一个集合，不用两个节点相连直接返回
        father[v] = u;
}
public:
    vector<int> findRedundantConnection(vector<vector<int>>& edges) {
        init();
        for (int i = 0; i < edges.size(); i++) {
            if (isSame(edges[i][0], edges[i][1])) return edges[i];
            else join(edges[i][0], edges[i][1]);
        }
        return {};
    }
};
```

### 685.冗余连接 II

```
在本问题中，有根树指满足以下条件的 有向 图。该树只有一个根节点，所有其他节点都是该根节点的后继。该树除了根节点之外的每一个节点都有且只有一个父节点，而根节点没有父节点。

输入一个有向图，该图由一个有着 n 个节点（节点值不重复，从 1 到 n）的树及一条附加的有向边构成。附加的边包含在 1 到 n 中的两个不同顶点间，这条附加的边不属于树中已存在的边。

结果图是一个以边组成的二维数组 edges 。 每个元素是一对 [ui, vi]，用以表示 有向 图中连接顶点 ui 和顶点 vi 的边，其中 ui 是 vi 的一个父节点。

返回一条能删除的边，使得剩下的图是有 n 个节点的有根树。若有多个答案，返回最后出现在给定二维数组的答案。

n == edges.length
3 <= n <= 1000
edges[i].length == 2
1 <= ui, vi <= n
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20210727151057.png)

那么有如下三种情况，前两种情况是出现入度为 2 的点，且只有一个节点入度为 2，为什么不看出度呢，出度没有意义，一棵树中随便一个父节点就有多个出度。如图：

![img](https://code-thinking.cdn.bcebos.com/pics/685.%E5%86%97%E4%BD%99%E8%BF%9E%E6%8E%A5II1.png)

第三种情况是没有入度为 2 的点，那么图中一定出现了有向环（**注意这里强调是有向环！**）

![img](https://code-thinking.cdn.bcebos.com/pics/685.%E5%86%97%E4%BD%99%E8%BF%9E%E6%8E%A5II2.png)

统计入度的代码如下：

```cpp
int inDegree[N] = {0}; // 记录节点入度
n = edges.size(); // 边的数量
for (int i = 0; i < n; i++) {
    inDegree[edges[i][1]]++; // 统计入度
}
```

前两种入度为 2 的情况，一定是删除指向入度为 2 的节点的两条边其中的一条，如果删了一条，判断这个图是一个树，那么这条边就是答案，同时注意要从后向前遍历，因为如果两条边删哪一条都可以成为树，就删最后那一条。

```cpp
vector<int> vec; // 记录入度为2的边（如果有的话就两条边）
// 找入度为2的节点所对应的边，注意要倒序，因为优先返回最后出现在二维数组中的答案
for (int i = n - 1; i >= 0; i--) {
    if (inDegree[edges[i][1]] == 2) {
        vec.push_back(i);
    }
}
// 处理图中情况1 和 情况2
// 如果有入度为2的节点，那么一定是两条边里删一个，看删哪个可以构成树
if (vec.size() > 0) {
    if (isTreeAfterRemoveEdge(edges, vec[0])) {
        return edges[vec[0]];
    } else {
        return edges[vec[1]];
    }
}
```

在来看情况三，明确没有入度为 2 的情况，那么一定有向环，找到构成环的边就是要删除的边。

```cpp
// 在有向图里找到删除的那条边，使其变成树，返回值就是要删除的边
vector<int> getRemoveEdge(const vector<vector<int>>& edges)
```

```cpp
class Solution {
private:
    static const int N = 1010; // 如题：二维数组大小的在3到1000范围内
    int father[N];
    int n; // 节点的数量，同时也是边的数量
    // 并查集初始化
    void init() {
        for (int i = 1; i <= n; ++i) {
            father[i] = i;
        }
    }
    // 并查集里寻根的过程
    int find(int u) {
        return u == father[u] ? u : father[u] = find(father[u]);
    }
    // 将v->u 这条边加入并查集
    void join(int u, int v) {
        u = find(u);
        v = find(v);
        if (u == v) return ;
        father[v] = u;
    }
    // 判断 u 和 v是否找到同一个根
    bool same(int u, int v) {
        u = find(u);
        v = find(v);
        return u == v;
    }
    // 在有向图里找到删除的那条边，使其变成树
    vector<int> getRemoveEdge(const vector<vector<int>>& edges) {
        init(); // 初始化并查集
        for (int i = 0; i < n; i++) { // 遍历所有的边
            if (same(edges[i][0], edges[i][1])) { // 构成有向环了，就是要删除的边
                return edges[i];
            }
            join(edges[i][0], edges[i][1]);
        }
        return {};
    }

    // 删一条边之后判断是不是树
    bool isTreeAfterRemoveEdge(const vector<vector<int>>& edges, int deleteEdge) {
        init(); // 初始化并查集
        for (int i = 0; i < n; i++) {
            if (i == deleteEdge) continue;
            if (same(edges[i][0], edges[i][1])) { // 构成有向环了，一定不是树
                return false;
            }
            join(edges[i][0], edges[i][1]);
        }
        return true;
    }
public:

    vector<int> findRedundantDirectedConnection(vector<vector<int>>& edges) {
        int inDegree[N] = {0}; // 记录节点入度
        n = edges.size(); // 边的数量
        for (int i = 0; i < n; i++) {
            inDegree[edges[i][1]]++; // 统计入度
        }
        vector<int> vec; // 记录入度为2的边（如果有的话就两条边）
        // 找入度为2的节点所对应的边，注意要倒序，因为优先返回最后出现在二维数组中的答案
        for (int i = n - 1; i >= 0; i--) {
            if (inDegree[edges[i][1]] == 2) {
                vec.push_back(i);
            }
        }
        // 处理图中情况1 和 情况2
        // 如果有入度为2的节点，那么一定是两条边里删一个，看删哪个可以构成树
        if (vec.size() > 0) {
            if (isTreeAfterRemoveEdge(edges, vec[0])) {
                return edges[vec[0]];
            } else {
                return edges[vec[1]];
            }
        }
        // 处理图中情况3
        // 明确没有入度为2的情况，那么一定有有向环，找到构成环的边返回就可以了
        return getRemoveEdge(edges);

    }
};
```
