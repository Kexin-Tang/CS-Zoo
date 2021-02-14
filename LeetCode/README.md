# 刷题笔记总结
此处总结了一些可供参考的题目思考角度，包括某些问题常见的算法

## 堆

一般而言，堆算法都可以用来解决 <**TOP-K**> 问题，比如说如下题目:
* [23. Merge k Sorted Lists](./python/23.%20Merge%20k%20Sorted%20Lists.md)
* [703. Kth Largest Element in a Stream](./python/703.%20Kth%20Largest%20Element%20in%20a%20Stream.md)

---

## 数组

### 1≤a[i]≤n, n为数组长度， 求出现0/1/2次的元素

该问题一般被称为**抽屉问题**，解决此方法的最佳方式都是时间复杂度O(n)，空间复杂度O(1)的方法 —— **原数组实现哈希**

> 由条件1 ≤ a[i] ≤ n，可知出nums 中的所有值可以和其索引有对应关系
> 
> 对应关系为 nums[i]的正负值可表示 值为i + 1是否出现，若出现则将其变为加上负号，即 nums[i] *= -1,默认为正整数即未出现

* [442. Find All Duplicates in an Array](./python/442.%20Find%20All%20Duplicates%20in%20an%20Array.md)
* [448. Find All Numbers Disappeared in an Array](./python/448.%20Find%20All%20Numbers%20Disappeared%20in%20an%20Array.md)