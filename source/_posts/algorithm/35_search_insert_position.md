---
title: 寻找插入位置
tags:
  - 算法
  - leetcode
  - 二分
categories: leetcode
date: 2021-02-01 09:45:03
---

## 题目([leetcode-35](https://leetcode.com/problems/search-insert-position/))
给定一个已排序的整数数组(数组中无重复元素)和目标值, 如果目标值存在于数组中, 返回索引, 如果不存在, 返回目标值应该在数组中插入的位置.  
例:
```
Input: nums = [1,3,5,6], target = 5
Output: 2
```

## 题解
由于数组有序可以使用二分查找.

### python
```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        if nums is None or len(nums) == 0: return -1
        low, high, mid = 0, len(nums) - 1, 0
        while(low <= high):
            mid = low + high >> 1
            if target == nums[mid]: return mid
            if target < nums[mid]: high = mid - 1
            else: low = mid + 1
        # 此处返回的条件是未在数组中找到与target相同的元素, 则最后一次循环(即low == high),有以下两种情况:
        # 1. target < nums[mid], 应返回mid, 此时low == mid
        # 2. target > nums[mid], 应返回mid + 1, 此时low == mid + 1
        return low
```

### java
```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        if (nums.length == 0) return -1;
        int low = 0, high = nums.length - 1, mid;
        while(low <= high) {
            mid = low + high >> 1;
            if (target == nums[mid]) return mid;
            else if (target < nums[mid]) high = mid - 1;
            else low = mid + 1;
        }
        return low;
    }
}
```

