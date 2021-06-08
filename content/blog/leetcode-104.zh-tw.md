---
title: "LeetCode 104. Maximum Depth of Binary Tree"
date: 2021-06-09T01:14:14+08:00
Tags: ["leetcode", "easy"]
Categories: ["algorithm"]
DisableComments: false
---

## LeetCode 104. Maximum Depth of Binary Tree

<span style="color: #43a047;">Easy</span>

#### 題目
Given a binary tree, find its maximum depth.

The maximum depth is the number of nodes along the longest path from the root node down to the farthest leaf node.

**Note:** A leaf is a node with no children.

**Example:**

Given binary tree `[3,9,20,null,null,15,7]`,
```
    3
   / \
  9  20
    /  \
   15   7
```
return its depth = 3.

#### 解決方法 1 (My Own Solution)
我們用 2 個變數分別記錄目前所在的層數和目前到過的最大層數，每到一層時如果目前所在的層數大於到過的最大層數就更新最大層數。

###### Java Code
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int maxDepth(TreeNode root) {
        int depth = 0; // 記錄目前所在 node 的深度
        int maxDepth = 0; // 記錄目前所到最深的深度
        int[] tuple = new int[]{depth, maxDepth};

        recTraverse(root, tuple);

        return tuple[1];
    }

    private void recTraverse(TreeNode root, int[] tuple) {
        if (root == null) {
            return;
        }

        // recTraverse 每呼叫一次表示到新的一層深度所以 depth + 1
        tuple[0]++;
        // 如果目前到達的 depth > maxDepth, maxDepth 用新的 depth 取代
        if (tuple[0] > tuple[1]) {
             tuple[1] = tuple[0];
        }

        if (root.left == null && root.right == null) {
            return;
        }

        if (root.left != null) {
            recTraverse(root.left, tuple);
            // 因為從下一層回來, 所以 depth - 1
            tuple[0]--;
        }

        if (root.right != null) {
            recTraverse(root.right, tuple);
            // 因為從下一層回來, 所以 depth - 1
            tuple[0]--;
        }
    }
}
```

###### Go Code
```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    maxDepth := recMaxDepth(root, 0, 0)

    return maxDepth
}

func recMaxDepth(node *TreeNode, currentDepth int, maxDepth int) int {
    currentDepth++
    if currentDepth > maxDepth {
        maxDepth = currentDepth
    }

    if (node.Left == nil && node.Right == nil) {
        return  maxDepth
    }

    if (node.Left != nil) {
        maxDepth = recMaxDepth(node.Left, currentDepth, maxDepth)
    }
    if (node.Right != nil) {
        maxDepth = recMaxDepth(node.Right, currentDepth, maxDepth)
    }

    return maxDepth
}
```