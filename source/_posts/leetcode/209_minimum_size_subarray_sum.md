---
title: 最短子数组和
tags:
  - 算法
  - leetcode
  - 滑动窗口
categories: leetcode
date: 2021-02-01 16:54:27
---
## 题目([leetcode-209](https://leetcode.com/problems/minimum-size-subarray-sum/))
给定一个含有n个正整数的数组和一个正整数s, 在数组中寻找一个最小的连续子数组, 且该子数组的和>=s. 如果存在返回子数组的长度, 不存在返回0.

例:
```
Input: s = 7, nums = [2,3,1,2,4,3]
Output: 2
Explanation: [4,3] 为该子数组.
```

## 题解
### python
```python
class Solution:
    def minSubArrayLen(self, s: int, nums: List[int]) -> int:
```
