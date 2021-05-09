---
title: 编辑距离
tags:
  - 算法
  - leetcode
categories: 算法
date: 2021-05-10 00:03:09
---

## 题目([leetcode-72](https://leetcode.com/problems/edit-distance/))

给定两个单词 `word1` 和`word2`, 返回将 `word1` 转换为`word2`需要的最少操作数.

一共可以使用以下操作:

- 插入一个字符
- 删除一个字符
- 替换一个字符

**Example 1:**

```
Input: word1 = "horse", word2 = "ros"
Output: 3
Explanation: 
horse -> rorse (替换 'h' 为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
```

**Example 2:**

```
Input: word1 = "intention", word2 = "execution"
Output: 5
Explanation: 
intention -> inention (删除 't')
inention -> enention (替换 'i' 为 'e')
enention -> exention (替换 'n' 为 'x')
exention -> exection (替换 'n' 为 'c')
exection -> execution (插入 'u')
```

**提示：**

- `0 <= word1.length, word2.length <= 500`
- `word1` 和 `word2` 由小写英文字母组成

## 题解

首先定义一个函数dp(i, j)为将word1[i:]转换为word2[j:]所需要的最少操作数. 

那么现在的问题就是如何得到dp(i, j):

- 如果word1[i] == word2[j], 那么代表当前字符已经相同, 则dp(i,j) = dp(i-1, j-1)

- 如果word1[i] != word2[j], 那么根据题目所给的操作, 我们有以下三个操作可以使用, 并使用三个步骤中需要的最少的操作数:
  - 删除word1[i], 此时比较word1[i - 1]与word2[j]
  - 替换word1[i], 此时比较word1[i - 1]与word2[j - 1]
  - 插入word2[j], 此时word1中新插入的元素与word2[j]已经相同, 所以需要比较word1[i]与word2[j - 1]
- 接着是基本情况:
  - 如果word1为空, 那么需要插入所有word2的字符
  - 如果word2为空, 那么需要删除所有word1的字符

### 递归解法

首先使用递归的解法

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int m = word1.length();
        int n = word2.length();
        int[][] memo = new int[m][n];
        for (int[] me : memo) {
            Arrays.fill(me, -1);
        }
        return recursion(word1, word2, m, n, memo);
    }
  
  	private int recursion(String s1, String s2, int i, int j, int[][] memo) {
        if (i == 0) return j;
        if (j == 0) return i;
        if (memo[i][j] != -1) return memo[i][j];
        if (s1.charAt(i) == s2.charAt(j)) {
          memo[i][j] = recursion(s1, s2, i - 1, j - 1, memo);
        } else {
            memo[i][j] =  min(
                recursion(s1, s2, i - 1, j) + 1,
                recursion(s1, s2, i - 1, j - 1) + 1,
                recursion(s1, s2, i, j - 1) + 1
            );
        }
        return memo[i][j];
    }
  	
  	private int min(int a, int b, int c) {
      return  Math.min(Math.min(a, b), c);
    }
}
```

### 迭代解法

```java

```

