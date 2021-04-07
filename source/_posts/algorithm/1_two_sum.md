---
title: 两数之和
tags:
  - 算法
  - leetcode
categories: leetcode
date: 2021-02-24 09:41:41
---
## 题目([leetcode-1](https://leetcode.com/problems/two-sum/))
给定整数数组nums, 目标值target, 返回数组中两数和为target的下标.  
假设每个输入有且仅有一个输出, 同一元素不可重复使用.  
例:
```
Example 1:
Input: nums = [2,7,11,15], target = 9
Output: [0,1]
Output: nums[0] + nums[1] == 9, return [0, 1].

Input: nums = [3,2,4], target = 6
Output: [1,2]
```

## 题解
暴力循环时间复杂度为O(n), 使用map以空间换时间.  

### python
```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        d = {}
        for i in range(nums):
            if nums[i] not in d:
                d[nums[i]] = i
            if target - nums[i] in d:
                return [i, d[nums[i]]
        return []
```

### java
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> m = new HashMap<>(nums.length);
        int[] res = new int[2];
        for (int i = 0; i < nums.length; i++) {
            int r = target - nums[i];
            if (m.containsKey(r)) { 
                return new int[]{i, m.get(r)};
            }
            m.put(nums[i], i);
        }
        return new int[0];
    }
}
```

