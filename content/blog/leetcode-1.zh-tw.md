---
title: "LeetCode 1. Two Sum"
date: 2020-12-01T23:10:57+08:00
Tags: ["leetcode", "easy"]
Categories: ["algorithm"]
---
## LeetCode 1. Two Sum

<span class="easy">Easy</span>

Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

Example:

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

#### 解決方法 1 (My Own Solution)
直覺的話就是暴力解法，用所有元素組合兩兩相加直到找出加總等於 `target` 為止。

###### Java Code
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++) {
            int num1 = nums[i];
            for (int j = i + 1; j < nums.length; j++) {
                if (num1 + nums[j] == target) {
                    return new int[]{i, j};
                }
            }
        }

        return new int[0];
    }
}
```

時間複雜度：每個元素都要遍歷陣列一次，所以有 $N$ 個元素時為 $O(N^2)$。

#### 解決方法 2 (My Own Solution)
使用 `Map` 將 `{數字 : index}` 儲存起來，在遍歷陣列時直接從 `Map` 取差值，如果有取到則回傳兩者的 index。

###### Java Code
```java
class Solution {
    public int twoSum(int[] nums, int target) {
        Map<Integer, Integer> records = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            records.put(nums[i], i);
        }

        for (int i = 0; i < nums.length; i++) {
            int num1 = nums[i];
            int num2 = target - num1;

            // 第 2 個條件避免找到自己，例如 6 - 3 = 3 的情況
            if (records.get(num2) != null && records.get(num2) != i) {
                return new int[]{i, records.get(num2)};
            }
        }

        return new int[0];
    }
}
```

時間複雜度：遍歷 2 次，所以 $N$ 個元素時為 $O(N)$
空間複雜度：需要用 `Map` 記錄陣列中的數字和 index，所以為 $O(N)$

#### 解決方法 3 (LeetCode Solution)
解決方法 2 可以再進一步優化，因為我們是用差值去 `Map` 中確認有無和差值相同的元素存在，所以如果 `num1 + num2 = target` 在遇到 `num1` 時，我們即使用差值 `num2` 取不到也將 `num1` 和它的 index 存入 `Map` 中，在遇到 `num2` 時用差值 `num1` 就可以取到 `num1` 的 index，所以我們可以不用先將陣列中的元素存入 `Map` 中，而是一邊遍歷一邊存入 `Map`。
例如 `[2, 7, 3, 4]` 而 `target = 9`，遍歷時先遇到 `2`，差值為 `7`，此時 `Map` 中沒有 `7` 但我們先把 `{2: 0}` 存入 `Map`，而繼續遍歷下去遇到 `7`，差值為 `2`，此時我們可以在 `Map` 取到 `{2:0}`，所以答案就是 `[1, 0]`。

###### Java Code
```java
class Solution {
    public int twoSum(int[] nums, int target) {
        Map<Integer, Integer> records = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int num1 = nums[i];
            int num2 = target - num1;
            if (records.get(num2) != null) {
                return new int[]{i, records.get(num2)};
            }

            records.put(num1, i);
        }

        return new int[0];
    }
}
```

###### Go Code
```go
func twoSum(nums []int, target int) []int {
    record := make(map[int]int)
    for i := 0; i < len(nums); i++ {
        n1, ok := record[target - nums[i]];
        if (ok) {
            return []int{i, n1}
        }
        record[nums[i]] = i
    }

    return []int{}
}
```
時間複雜度：需要遍歷 $N$ 個元素，所以為 $O(N)$
空間複雜度：需要建立儲存 $N$ 個元素的 `Map`，所以為 $O(N)$

如果有什麼想法或需要指正的地方，歡迎您留言或來信 😄