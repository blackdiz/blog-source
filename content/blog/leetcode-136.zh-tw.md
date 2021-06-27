---
title: "Leetcode 136. Single Number"
date: 2021-06-27T21:35:21+08:00
Tags: ["leetcode", "easy"]
Categories: ["algorithm"]
---

## LeetCode 136. Single Number

<span class="easy">Easy</span>

#### 題目
Given a **non-empty** array of integers, every element appears _twice_ except for one. Find that single one.

**Note:**

Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

**Example 1:**
```
Input: [2,2,1]
Output: 1
```
**Example 2:**
```
Input: [4,1,2,1,2]
Output: 4
```

#### 解決方法 1 (My Own Solution)
用最笨的暴力方式每次取 1 個元素和它以外的元素比對, 比到重複的就跳出該次迴圈直到找到沒重複的。
這題如果只是要求解的話很簡單，除了用巢狀回圈外，還可以遍歷陣列用 `Map` 記錄元素出現個次數後，再從 `Map` 中找出只現在一次的元素，但這些簡單的解法都不符合額外的要求，也就是必須在 $O(N)$ 的時間複雜度下不使用額外的空間，所以就不列出其他這類簡單解法了。

###### Java Code
```java
class Solution {
    public int singleNumber(int[] nums) {
        int singleNumber = nums[0];
        for (int i = 0; i < nums.length; i++) {
            boolean duplicated = false; // 標記此次比對是否有重複元素
            singleNumber = nums[i];
            for (int j = 0; j < nums.length; j++) {
                if (i == j) { // index 相同表示為同個元素, 不用比對
                    continue;
                } else if (nums[i] == nums[j]) {
                    // 比對到重複的就可以跳出減少時間消耗
                    duplicated = true;
                    break;
                }
            }
            if (duplicated) {
                continue;
            } else {
                break;
            }
        }
        return singleNumber;
    }
}
```

#### 解決方法2 (LeeCode Solution)
對不熟悉位元運算的我來說，這個解法真的是想破頭也想不到的方法，只能說真的透過這題了解到了位元運算的特殊性質。
此解法使用了 **XOR** 運算來得到答案，**XOR** 有下列性質：
- 如果用 0 和其他數字作 **XOR** 運算則得到原數字：
  $a ⊕ 0 = a$
- 如果用相同數字做 **XOR** 運算則得到 0：
  $a ⊕ a = 0$

因為 **XOR** 運算具有交換律，所以順序可以交換，因此可以把所有數字依序做 **XOR** 運算，最後留下的即為非重複數字：

$a ⊕ b ⊕ a = ( a ⊕ a ) ⊕ b = 0 ⊕ b = b$

  以題目 `[4,1,2,1,2]` 為例：

\begin{align}
&4 ⊕ 1 ⊕ 2 ⊕ 1 ⊕ 2 ⊕ 1\newline
&= 1 ⊕ 1 ⊕ 2 ⊕ 2 ⊕ 4\newline
&= 0 ⊕ 2 ⊕ 2 ⊕ 4\newline
&= 2 ⊕ 2 ⊕ 4\newline
&= 0 ⊕ 4\newline
&= 4\newline
\end{align}

###### Java Code
```java
class Solution {
    public int singleNumber(int[] nums) {
        int result = 0;
        for (int i : nums) {
            result ^= i;
        }

        return result;
    }
}
```

如果有什麼想法或需要指正的地方，歡迎您留言或來信 😄