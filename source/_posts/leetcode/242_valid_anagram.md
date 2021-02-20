---
title: 变位词
tags:
  - 算法
  - leetcode
categories: leetcode
date: 2021-02-20 12:24:29
---
## 题目([leetcode-242](https://leetcode.com/problems/valid-anagram/))
给定两个字符串s, t, 判断t是否是s的变位词(即t是由s中的字符重新排序组成).  
假定字符串中只包含小写字母.
例:
```
Example 1:
Input: s = "anagram", t = "nagaram"
Output: true

Example 2:
Input: s = "rat", t = "car"
Output: false
```

## 题解
由于只有小写字母, 可以使用数组记录每个字符出现的次数.  
也可以使用map记录字母出现的次数
### python
```python
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        if s is None or t is None or len(s) != len(t):
            return False
        lst = [0] * 26
        for i in range(len(s)):
            lst[ord(s[i]) - ord('a')] += 1
            lst[ord(t[i]) - ord('a')] -= 1
        for i in lst:
            if i != 0:
                return False

        return True

    def usingMap(self, s: str, t: str) -> bool:
        if s is None or t is None or len(s) != len(t):
            return False
        d = {}
        for i in range(len(s)):
            d[s[i]] = d.get(s[i], 0) + 1
            d[t[i]] = d.get(t[i], 0) - 1
        for v in d.values():
            if v != 0:
                return False
        return True
```

### java
```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if (s == null || t == null || s.length() != t.length()) return false;
        int[] lst = new int[26];
        for (int i = 0; i < s.length(); i++) {
            lst[s.charAt(i) - 'a'] += 1;
            lst[t.charAt(i) - 'a'] -= 1;
        }
        for (int i : lst) {
            if (i != 0) return false;
        }
        return true;
    }
}
```

