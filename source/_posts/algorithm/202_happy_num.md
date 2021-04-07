---
title: 快乐数
tags:
  - 算法
  - leetcode
categories: leetcode
date: 2021-02-24 09:00:51
---
## 题目([leetcode-202](https://leetcode.com/problems/happy-number/))
给定一个数n判断是否是快乐数:  
快乐数定义:  
- 以任意正整数开始, 以数字的每位的平方的和替换原来的数字
- 重复以上步骤直到数字变为1, 如果数字不会变成1将会无线循环
- 如果过程中产生1, 则该数字是快乐数.

约束:
$ 1 <= n < 2^{31} - 1 $

例:
```
Input: n = 19
Output: true
Explanation:
12 + 92 = 82
82 + 22 = 68
62 + 82 = 100
12 + 02 + 02 = 1
```

## 题解
一个重要的信息是, 如果数字不是快乐数, 则会进入无线循环.  
所以可以使用set保存的到的数字, 如果新产生的结果存在于set中, 表示该数不是快乐数.  
使用判断列表是否有环的思想(Floyd's cycle-finding algorithm), 使用快慢指针, 如果相遇且不为1表示不是快乐数.
### python
使用set.
```python
class Solution:
    def calcSum(self, n: int) -> int:
        s = 0
        while n != 0:
            s += (n % 10)**2
            n = n // 10
        return s

    def isHappy(self, n: int) -> bool:
        if n <= 0: 
            return false
        s = set()
        while n != 1:
            n = self.calcSum(n)
            if n in s:
                return False
            s.add(n)
        return True
```
### java
使用双指针
```java
class Solution {
    public boolean isHappy(int n) {
        if (n <= 0) return false;
        int slow = calcSum(n);
        int fast = calcSum(calcSum(n));
        while (slow != fast) {
            if (slow == 1 || fast == 1) return true;
            slow = calcSum(slow);
            fast = calcSum(calcSum(fast));
        }
        return slow == 1;
    }

    private int calcSum(int n) {
        int sum = 0;
        while (n != 0) {
            sum += (n % 10) * (n % 10);
            n = n / 10;
        }
        return sum;
    }
}
```

