---
title: 272. Closest Binary Search Tree Value II
categories:
  - Leetcode
tags:
  - tree
  - tree traversal
---
## 1. Problem Description
Given a non-empty binary search tree and a target value, find k values in the BST that are closest to the target.

Note:
- Given target value is a floating point.
- You may assume k is always valid, that is: k ≤ total nodes.
- You are guaranteed to have only one unique set of k values in the BST that are closest to the target.

## 2. Solution Key Points
An intuitive way is to iterate all the nodes and find the top k. This can be solved by the typical TopK algorithms. The following methods are two that may not necessarily visit all the nodes in the tree.

### Method I: Sliding Window
If you convert the BST into an array, the k closest numbers will be continuous in the array. Therefore we can use inorder traversal + sliding window.

This is a *fixed-size* sliding window with *early termination*. At each step check if the current node should be added into the sliding window by checking
1. sliding window size
2. if the current node value is closer to the target than the *first* node.

If the 2nd condition is not satisfied, that means further increasing node value will not make it closer to target, so we stop.
```java
public List<Integer> closestKValues(TreeNode root, double target, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    Queue<TreeNode> queue = new ArrayDeque<>();
    TreeNode prev = root;
    while (!stack.isEmpty() || prev != null) {
        if (prev != null) {
            stack.offerFirst(prev);
            prev = prev.left;
        } else {
            TreeNode cur = stack.pollFirst();
            prev = cur.right;
            if (queue.size() < k) {
                queue.offer(cur);
            } else {
                if (Math.abs(cur.val - target) < Math.abs(queue.peek().val - target)) {
                    // should use the current node
                    queue.poll();
                    queue.offer(cur);
                } else {
                    break; // early termination
                }
            }
        }
    }
    List<Integer> results = new ArrayList<>();
    for (TreeNode node : queue) {
        results.add(node.val);
    }
    return results;
}
```
*Time complexity*: O(n) because of inorder traversal.  
*Space complexity*: O(h) because inorder traversal takes O(h) space.

### Method II: Two Stacks
The idea is that, create two iterators:
- one will offer the largest node value that is less than the target.
- the other one will offer the smallest node value that is greater than or equal to the target.

Maintain these two iterators will take O(h) time.

The function `initializeLeftStack` actually finds the "top-right" boundary for all the nodes with value less than the target.
```java
private void initializeLeftStack(Deque<TreeNode> leftStack, TreeNode root, double target) {
    TreeNode cur = root;
    while (cur != null) {
        if (cur.val < target) {
            leftStack.offerFirst(cur);
            cur = cur.right;
        } else {
            cur = cur.left;
        }
    }
}
```

![smallerElementsIterator](/assets/images/leetcodes/272/smallerElementsIterator.png)


## 3. Mistakes
- `getSuccessor` and `getPredecessor` can be defined on a BST w.r.t. a target.
- two stacks should not share same nodes.

## 4. Related Problems
1. Given a sorted array, find the k closest numbers w.r.t. a target.
