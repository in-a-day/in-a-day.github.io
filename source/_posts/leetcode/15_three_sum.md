---
title: 三数之和
tags:
  - 算法
  - leetcode
categories: leetcode
date: 2021-02-24 10:26:29
---
## 题目([leetcode-15](https://leetcode.com/problems/3sum/))
给定整数数组nums, 在数组中寻找a,b,c使得a+b+c=0. 返回全部的不重复的结果.  
例:
```
Example 1:
Input: nums = [-1,0,1,2,-1,-4]
Output: [[-1,-1,2],[-1,0,1]]

Input: nums = [0]
Output: []
```

## 题解
暴力循环需要$O(n^3)$时间复杂度, 首先排序便于过滤重复结果, 可以使用双指针进行数据筛选

### python
```python
```

### java
```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        // 排序便于筛选出
        Arrays.sort(nums);
        List<List<Integer>> res = new LinkedList<>();
        for (int i = 0; i < nums.length; i++) {
            // 排序后起始的值>0, 无结果, 跳过
            if (nums[i] > 0) break;
            // 去除重复结果, i > 0保证了 [-1, -1, 0]这种情况不会跳过
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            for (int left = i + 1, right = nums.length - 1; left < right;) {
                int s = nums[left] + nums[right];
                if (s == -nums[i]) {
                    // 添加结果, 同时left - 1与right + 1
                    res.add(Arrays.asList(nums[i], nums[left++], nums[right--]));
                    // 去除重复结果
                    while (left < right && nums[left] == nums[left - 1]) left++;
                    while (left < right && nums[right] == nums[right + 1]) right--;
                } else if (s < -nums[i]) {
                    left++;
                } else {
                    right--;
                }
            }
        }
        return res;
    }
}
```
