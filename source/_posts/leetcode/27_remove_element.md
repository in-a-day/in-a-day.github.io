---
title: 移除元素
tags:
  - 算法
  - leetcode
  - 双指针
categories: leetcode
date: 2021-02-01 13:50:41
---
## 题目([leetcode-27](https://leetcode.com/problems/remove-element/))
给定一个数组和一个值val, 原地(in-place, 即不使用额外空间)移除数组中所有等于val的元素, 并返回新的长度. 数组元素的顺序可以改变, 数组剩余长度元素的值可以是任意的.
例:   
```
Input: nums = [3, 2, 2, 3], val = 3  
Output: 2, nums = [2, 2, x, x]  
Explanation: 函数只需要返回2即可, 代表数组前两个元素. 数组后两位的元素可以是任意值[2, 2, 3, 3], [2, 2, 1, 2]...  
```

## 题解
### python
```python
class Solution:
    def removeElement(self, nums: List[int], val: int) -> int:
        if nums is None or len(nums) == 0: return -1
        # 首尾指针
        left, right = 0, len(nums) - 1
        while left < right:
            if nums[left] == val:
                if nums[right] != val:
                    nums[left] = nums[right]
                right -= 1
            else:
                left += 1
        # 最后一次循环有两种情况:
        # 1. nums[left] == val, 此时判断循环后的nums[left]即可
        # 2. nums[left] != val, 循环后left已经+1, 此时也是直接判断left的值即可

        return left if nums[left] == val else left + 1

    def removeElement2(self, nums: List[int], val: int) -> int:
        if nums is None or len(nums) == 0: return -1
        # 快慢指针
        # curr指针用于记录不等于val的元素个数
        curr = 0
        # 每次i != val, 就增加curr, 并将nums[i]的值赋值给nums[curr]
        # 就相当于新建数组, 每次i != val, 就插入新数组, 新数组index+1
        for i in nums:
            if i != val:
                nums[curr] = i
                curr += 1
        return curr
```


### java
```java
class Solution {
    public int removeElement(int[] nums, int val) {
        if (nums == null || nums.length == 0) return -1;
        int curr = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] != val) nums[curr++] = nums[i]
        }

        return curr;
    }
}

```

