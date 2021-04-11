---
title: 快速排序
date: 2021-04-07 23:41:37
tags: 
    - 排序
    - 算法
categories: 算法
---
## 快速排序
> 快速排序使用分治的思想: 将一个数组以给定元素分为两个数组, 一个数组存放小于该数的其他元素, 另一个数组存放大于该数的其他元素(等于该数的元素可以在任意一个数组中皆可). 接着递归排序这两个数组, 两个数组都有序了, 那么整个数组也就有序了.

**NB: 递归发生在排序之后**

### 快排实现
```java
public class QuickSort {
    public static void sort(int[] nums) {
        
    }

    private static void sort(int[] nums, int left, int right) {
        if (right <= left) return;
        // 以pivot为基准, 将小于pivot的数放在左侧, 大于的放在右侧
        int pivotIndex = partition(nums, left, right);
        // 排序小于pivot的数组
        sort(nums, left, pivotIndex - 1);
        // 排序大于pivot的数组
        sort(nusm, pivotIndex + 1, right);
    }

    private static void partition(int[] nums, int left, int right) {
        
    }
}
```
