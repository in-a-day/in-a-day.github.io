---
title: 螺旋矩阵2
tags:
  - 算法
  - leetcode
categories: leetcode
date: 2021-02-03 10:31:12
---
## 题目([leetcode-59](https://leetcode.com/problems/spiral-matrix-ii/))
给定正整数n, 以螺旋方式生成一个`n x n`矩阵, 矩阵元素为1至$n^2$.  
例:
```
Input: n = 3
Output: [[1,2,3],[8,9,4],[7,6,5]]

Input: n = 1
Output: [[1]]
```
限制: $1 <= n <= 20$

## 题解
顺时针插入: 左上->右上, 右上->右下, 右下->左下, 左下->左上

### python
```python
class Solution:
    def generateMatrix(self, n: int) -> List[List[int]]:
        if n <= 0: return [[]]
        res = [[0] * n for _ in range(n)]
        # 行列开始位置
        row_start, col_start = 0, 0
        # 行列结束位置
        row_end, col_end = n - 1, n - 1
        # 填充的数字
        count = 1
        # 顺时针进行填充
        while row_start <= row_end and col_start <= col_end:
            # 左->右
            for i in range(col_start, col_end + 1):
                res[row_start][i] = count
                count += 1
            row_start += 1

            # 上->下
            for i in range(row_start, row_end + 1):
                res[i][col_end] = count
                count += 1
            col_end -= 1

            # 右->左
            for i in range(col_end, col_start - 1, -1):
                res[row_end][i] = count
                count += 1
            row_end -= 1

            # 下->上
            for i in range(row_end, row_start - 1, -1):
                res[i][col_start] = count
                count += 1
            col_start += 1

        return res
```

### java
```java
class Solution {
    public int[][] generateMatrix(int n) {
        if (n <= 0) return new int[0][0];           
        int[][] res = new int[n][n];
        int row_start = 0, col_start = 0;
        int row_end = n - 1, col_end = n - 1;
        int count = 1;
        while (row_start <= row_end && col_start <= col_end) { 
            for (int i = col_start; i <= col_end; i++) {
                res[row_start][i] = count++;
            }
            row_start++;

            for (int i = row_start; i <= row_end; i++) {
                res[i][col_end] = count++;
            }
            col_end--;

            for (int i = col_end; i >= col_start; i--) {
                res[row_end][i] = count++;
            }
            row_end--;

            for (int i = row_end; i >= row_start; i--) {
                res[i][col_start] = count++;
            }
            col_start++;
        }
        return res;
    }
}
```
