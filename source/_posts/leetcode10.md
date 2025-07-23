---
title: leetcode刷题
date: 2024-12-21 08:52:26
categories:
    - leetcode
tags: 
    - 刷题
---
# [无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)
```python
class Solution:
    # 使用两个指针[i, j]记录下无重复字符的字串
    # 使用一个set记录子串，方便查找是否有重复
    def lengthOfLongestSubstring(self, s: str) -> int:
        # 边界条件
        if len(s) < 1:
            return 0
        # 起始位置0， 已经包含在不重复的指针区间
        i, j = 0, 0
        char_set = set()
        char_set.add(s[j])
        result_length = 1
        # 右指针向前探路，因为当前已经包含了右指针，所以要判断下一个指针会不会越界
        while j + 1 < len(s):
            # 如果下一个不重复，则把下一个放进区间
            if s[j + 1] not in char_set:
                j += 1
                char_set.add(s[j])
                # 计算最大的长度
                result_length = max(result_length, j - i + 1)
            # 如果重复了，要移动左指针
            else:
                char_set.discard(s[i])
                i += 1
        return result_length
```
# [数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)
```python
class Solution:
    # 使用一个大小为k的小根堆，那么根就是第k大的数
    # heapq默认是小根堆
    def findKthLargest(self, nums: List[int], k: int) -> int:
        # 先填充满大小为k的小根堆
        my_heap = nums[:k] 
        heapq.heapify(my_heap)
        for i in range(k, len(nums)):
            # 如果当前数大于heap的根，则把该元素加入，并弹出根部
            if nums[i] > my_heap[0]:
                heapq.heappushpop(my_heap, nums[i])
        # 返回根部
        return my_heap[0]
```
python 实现一个小根堆
```python


```