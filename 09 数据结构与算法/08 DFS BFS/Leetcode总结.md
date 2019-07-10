# [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

给定一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，计算岛屿的数量。一个岛被水包围，并且它是通过水平方向或垂直方向上相邻的陆地连接而成的。你可以假设网格的四个边均被水包围。

**示例 1:**

```
输入:
11110
11010
11000
00000

输出: 1
```

**示例 2:**

```
输入:
11000
11000
00100
00011

输出: 3
```

**思路：**

- 典型的DFS，递归的作用是以当前的位置为起点，进行深度优先搜索，将上下左右为1的都置0
- 每执行完一次，意味着存在一座岛屿，则sum++

```java
class Solution {
    public int numIslands(char[][] grid) {
       if(grid == null || grid.length == 0 || grid[0].length == 0){
           return 0;
       }
        int numIslands = 0;
        int rows = grid.length;
        int cols = grid[0].length;
        for(int i = 0; i < rows; i++){
            for(int j = 0; j < cols; j++){
                if(grid[i][j] == '1'){
                    numIslands++;
                    dfs(grid, i, j, rows, cols);
                }
            }
        }
        return numIslands++;
    }
        private void dfs(char[][] grid, int i, int j, int rows, int cols){
            if(j < cols && i < rows && j>=0 && i>=0 && grid[i][j] == '1')
		{
			grid[i][j] = '0';
			dfs(grid,i+1,j,rows,cols);
			dfs(grid,i,j+1,rows,cols);
			dfs(grid,i-1,j,rows,cols);
			dfs(grid,i,j-1,rows,cols);
		}
        }
}
```

## 并查集

并行计算