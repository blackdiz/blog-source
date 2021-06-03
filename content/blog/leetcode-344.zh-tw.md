---
title: "Leetcode 344. Reverse String"
date: 2021-03-26T10:56:51+08:00
Tags: ["leetcode", "easy"]
Categories: ["algorithm"]
---

## LeetCode 344. Reverse String

<span style="color: #43a047;">Easy</span>

#### 題目
Write a function that reverses a string. The input string is given as an array of characters `char[]`.

Do not allocate extra space for another array, you must do this by **modifying the input array [in-place](https://en.wikipedia.org/wiki/In-place_algorithm)** with O(1) extra memory.

You may assume all the characters consist of [printable ascii characters](https://en.wikipedia.org/wiki/ASCII#Printable_characters).

**Example 1:**
```
Input: ["h","e","l","l","o"]
Output: ["o","l","l","e","h"]
```
**Example 2:**
```
Input: ["H","a","n","n","a","h"]
Output: ["h","a","n","n","a","H"]
```

#### 解決方法1
用兩個指標 `start` 和 `end` 分別由陣列第一個和最後一個元素開始交換位置，並且彼此慢慢靠近，因為字串長度可能為奇數或偶數，所以直到 `start >= end` 為止 (當為奇數時 `start == end`，當為偶數時兩者會交錯所以用 `start > end` 判斷)。

###### Java Code
```java
class Solution {
    public void reverseString(char[] s) {
        int end = s.length - 1;
        for (int start = 0; start < s.length; start++) {
              if (start >= end) {
                  return;
              }
              char temp = s[start];
              s[start] = s[end];
              s[end] = temp;
              end--;
        }
    }
}
```

