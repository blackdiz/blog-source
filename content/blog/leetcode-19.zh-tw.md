---
title: "LeetCode 19. Remove Nth Node From End of List"
date: 2021-03-20T22:20:55+08:00
Tags: ["leetcode", "medium"]
Categories: ["algorithm"]
DisableComments: false
---

# LeetCode 19. Remove Nth Node From End of List

 <span style="color: #ef6c00;">Medium</span>

Given the head of a linked list, remove the nth node from the end of the list and return its head.

**Follow up**: Could you do this in one pass?

Example 1:
```
Input: head = [1,2,3,4,5], n = 2
Output: [1,2,3,5]
```
Example 2:
```
Input: head = [1], n = 1
Output: []
```
Example 3:
```
Input: head = [1,2], n = 1
Output: [1]
```


Constraints:

   - The number of nodes in the list is sz.
   - 1 <= sz <= 30
   - 0 <= Node.val <= 100
   - 1 <= n <= sz

---
### 解決方法 1
因為我們不曉得輸入的 `Linked List` 有多長，所以我們一邊遍歷 `Linked List` 一邊用 1 個 `List` 記錄下遍歷過的 Node 和順序。
之後我們就可以運用 `List` 可以用 index 直接取得元素的特性，取出 第 `n - 1` 個 Node，將它的 `next` 指 `next.next` 來把第 `n` 個 Node 刪除。

#### Java Code
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        List<ListNode> copy = new ArrayList<>();
        // 按順序複製 ListNode 到 List 中
        while (head != null) {
            copy.add(head);
            head = head.next;
        }

        // 因為 n 是倒數的, 所以 n 指向的 index 為 List 的長度 - n, 而我們目標要取出 n - 1
        // 所以我們實際要取出的 Node 是第 List 的長度 - n - 1 個
        // 而如果算出來的 index < 0 表示我們要刪除的第 n 個 Node 本身是 head, 所以 n - 1 的 index 才會 < 0
        // 因此這時刪除原本的 head 就是回傳 head.next
        int previousOfN = copy.size() - n - 1;
        if (previousOfN < 0) {
            return copy.get(0).next;
        }

        ListNode previousNodeOfRemoved = copy.get(previousOfN);
        previousNodeOfRemoved.next = previousNodeOfRemoved.next.next;

        return copy.get(0);
    }
}
```

### 解決方法 2
先遍歷 `Linked List` 一次得到長度後，再遍歷第 2 次，等走到第 `n - 1` 個 Node 時，將它的 `next` 指向 `next.next` 來把第 `n` 個 Node 刪除。

#### Java Code
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        int count = 0;
        ListNode current = head;
        // 計算長度
        while (current != null) {
            current = current.next;
            count++;
        }

        int previousOfN = count - n - 1;
        if (previousOfN < 0) {
            return head.next;
        }

        count = 0;
        current = head;
        // 找到 n 前 1 個 Node
        while (count != previousOfN) {
            current = current.next;
            count++;
        }

        current.next = current.next.next;

        return head;
    }
}
```