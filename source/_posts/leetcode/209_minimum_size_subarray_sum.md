---
title: 最短子数组和
tags:
  - 算法
  - leetcode
  - 双指针
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
### 滑动窗口 
> 使用双指针, 时间复杂度: O(n), 空间复杂度O(1)
#### python
```python
class Solution:
    def minSubArrayLen(self, s: int, nums: List[int]) -> int:
        if nums is None: return 0
        # 由于s是正整数, 所以_sum取0即可
        left, right, _sum, res = 0, 0, 0, len(nums) + 1
        while right < len(nums):
            _sum += nums[right]
            right += 1
            # 当_sum >= s时, 向右滑动到_sum < s
            # 当left == right, 此时_sum = 0, s > 0所以不会越界
            while _sum >= s:
                res = min(res, right - left)
                _sum -= nums[left]
                left += 1
    
        return 0 if res == len(nums) + 1 else res

```

#### java
```java
class Solution {
    public int minSubArrayLen(int s, int[] nums) {
        if (nums == null) return 0;
        int left = 0, sum = 0, res = nums.length + 1;
        for (int right = 0; right < nums.length; right++) {
            sum += nums[right];
            while (sum >= s) {
                res = res <= right - left + 1 ? res : right - left + 1;
                sum -= nums[left++];
            }
        }
        return res == nums.length + 1 ? 0 : res;
    }
}
```

### 二分(TODO)


