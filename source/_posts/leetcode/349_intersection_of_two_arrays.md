---
title: 两个数组的交集
tags:
  - 算法
  - leetcode
categories: leetcode
date: 2021-02-20 17:06:46
---
## 题目([leetcode-349](https://leetcode.com/problems/intersection-of-two-arrays/))
给定两个数组, 计算他们的交集.  
结果中的元素必须是唯一的, 结果可以是任意顺序.
例:
```
Example 1:
Input: nums1 = [1,2,2,1], nums2 = [2,2]
Output: [2]

Example 2:
Input: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
Output: [9,4]
```

## 题解
利用set减少时间复杂度.

### python
```python
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        if nums1 is None or nums2 is None:
            return []
        s = set(nums1)

        return list({i for i in nums2 if i in s})

    def useSetApi(self, nums1: List[int], nums2: List[int]) -> List[int]:
        return list(set(nums1) & set(nums2))

    def useMap(self, nums1: List[int], nums2: List[int]) -> List[int]:
        if nums1 is None or nums2 is None:
            return []
        d = {}
        for i in nums1:
            if i not in d:
                d[i] += 1
        for i in nums2:
            if i in d:
                d[i] += 1

        return [i for k, v in d.items() if v >= 2]
```

