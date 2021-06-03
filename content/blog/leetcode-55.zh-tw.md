---
title: "LeetCode 55 . Jump Game"
date: 2021-06-04T07:23:28+08:00
Tags: ["leetcode", "medium"]
Categories: ["algorithm"]
---
# LeetCode 55. Jump Game

<font color="#ef6c00">Medium</font>

Given an array of non-negative integers `nums`, you are initially positioned at the **first index** of the array.

Each element in the array represents your maximum jump length at that position.

Determine if you are able to reach the last index.

**Example 1:**
```
Input: nums = [2,3,1,1,4]
Output: true
Explanation: Jump 1 step from index 0 to 1, then 3 steps to the last index.
```
**Example 2:**
```
Input: nums = [3,2,1,0,4]
Output: false
Explanation: You will always arrive at index 3 no matter what. Its maximum jump length is 0, which makes it impossible to reach the last index.
```
**Constraints:**

-   `1 <= nums.length <= 104`
-   `0 <= nums[i] <= 105`

#### 解法 1
根據題目的說明，我們從第 1 個 index 出發，其數字為可走的步數，所以以反過來如果我們從最後一個 index 開始往前，只要前 1 個 index 的元素也就是可走的步數大於或等於兩個 index 間的距離就表示可以有辦法走到下一個 index，我們就可以再往前走。
以 Example 1 為例：
1. index 3 的步數為 1，等於 index 3 到 index 4 (同時也是終點)的距離 1 表示有辦法走到終點，所以我們可以再往前一個 index 檢查。
2. index 2 的步數為 1，等於 index 2 到 index 3 的距離 1，再往前一個 index。
3. index 1 的步數為 3 大於 index 1 到 index 2 的距離 1，再往前一個 index。
4. index 0 的步數為 2 大於 index 0 到 index 1 的距離 1，因為已經走到起點了，所以 Example 1 的陣列可以從起點走到終點。

同樣的邏輯套用到 Exmaple 2 上:
1. index 3 的步數為 0 小於 index 3 到 index 4 的距離 1，但因為在 index 3 之前還是可能有能走到 index 4 的步數，所以我們繼續往前檢查。
2. index 2 的步數為 1 小於 index 2 到 index 4 的距離 2
3. 再往前 index 1 的步數為 2 小於 index 1 到index 4 的距離 3
4. 再往 index 0 的步數為 3 小於 index 0 到 index 4 的距離 4，因為已經檢查到起點了，所以 Example 2 的陣列沒辦法從起點走到終點。

###### Java Code
```java
class Solution {
    public boolean canJump(int[] nums) {
        // 處理陣列長度為 1 時的特殊情況
        if (nums.length == 1) {
            return true;
        }

        int i = nums.length - 2;
        // 兩個 index 間的距離
        int distance = 1;
        // 從倒數第 2 個 index 開始檢查
        for (; i >= 0; i--) {
            if (nums[i] >= distance) {
                // 如果走到起點表示此陣列可以從起點走到終點
                if (i == 0) {
                    return true;
                }
                // 當要檢查下一個 index 時，檢查距離重設為 1
                distance = 1;
                continue;
            }
            // 如果此 index 的步數不滿足條件，則檢查下一個 index 時，檢查距離要加 1
            distance++;
        }

        return i == 0;
    }
}
```

- 時間複雜度：因為是遍歷陣列，所以時間複雜度為 $O(N)$

#### 解法 2 (LeetCode Forum Solution)
解法方式和解法 1 相同，但程式碼簡潔更多。基本上是將解法 1 中的 index 的元素是否大於或等於 `steps` 的計算直接用 `index + 該 index 的元素` 是否大於或等於後一個 index 取代，同時會用 1 個 pointer `last` 指向可以成功走到後一個 index 的 index 為何，如果最後 `last` 指到 index 0 表示可以從起點走到終點。

###### Java Code
```java
class Solution {
    public boolean canJump(int[] nums) {
        int last = nums.length - 1;
        for (int i = nums.length - 2; i >= 0; i--) {
            // index + index 的元素如果 >= 後 1 個 index, 表示可以從這邊走到後一個 index 的地方, 所以我們可以繼續往前檢查
            if (i + nums[i] >= last) {
                last = i;
            }
        }

        return last == 0;
    }
}
```

- 時間複雜度：因為需要遍歷陣列，所以同樣為 $O(N)$

#### 解法 3 (LeetCode Forum Solution)
這個解法的思考方式是，我們從起點開始記錄該 index 的步數最遠可以到的 index (用變數 `max` 記錄)，然後開始往後遍歷，如果中間有 index 的步數可以把 `max` 再往更後面推進就更新 `max` 的值，但如果我們在遍歷的過程中所檢查的 index 已經超出 `max` 指向的 index，表示我們最遠只能到 `max` 所指向的 index 而無法到達終點。如果可以順利遍歷完表示 `max` 最終的數值一定是等於或大於終點的 index。
以 Example 1 為例的話：
1. 開始 `max = 0`，起點 index 0 為 2 所以 `max = 2` 表示此時我們最遠可以到 index 2。
2. index 1 為 3 大於 `max` 所以更新 `max = 3` 表示此時我們最遠可以到 index 3。
3. index 2 為 1 小於 `max` 所以我們可以不用使用它。
4. index 3 為 1 小於 `max` 所以我們可以不用使用它。
5. index 4 為 4 大於 `max`，同時我們也遍歷到終點了，所以我們可以成功地從起點走到終點。

以 Example 2 為例的話：
1. 開始 `max = 0`，起點 index 0 為 3 所以 `max = 3` 表示此時我們最遠可以到 index 3。
2. index 1 為 2 小於 `max` 所以我們可以不用使用它。
3. index 2 為 1 小於 `max` 所以我們可以不用使用它。
4. index 3 為 0 小於 `max` 所以我們可以不用使用它。
5. index 4 為 4 但因為此時我們遍歷到的 index 已經超過 `max` 指向的 index 了，表示我們最遠只能走到 `max` 指向的 index 3，因此不可能從起點走到終點。

###### Java Code
```java
class Solution {
    public boolean canJump(int[] nums) {
        int max = 0;
        for (int i = 0; i < nums.length; i++) {
            if (i > max) {
                return false;
            }
            max = Math.max(i + nums[i], max);
        }

        return true;
    }
}
```

- 時間複雜度：需要遍歷陣列，所以時間複雜度為 $O(N)$

