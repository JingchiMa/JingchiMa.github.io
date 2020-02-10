---
title: Leetcode 572. Subtree of Another Tree
categories:
  - Leetcode
tags:
  - tree
  - leetcode
---

## 1. Problem Description
Given two non-empty binary trees `s` and `t`, check whether tree `t` has exactly the same structure and node values with a subtree of `s`. A subtree of `s` is a tree consists of a node in `s` and all of this node's descendants. The tree `s` could also be considered as a subtree of itself.

## 2. Solution Key Point
Traverse every tree node. For each tree node in `s`, check if the corresponding subtree rooted at `s` is equal to `t`.

## 3. Mistakes
- I originally thought this could be solved in $O(n)$ time. My mistake was that even if `s.val == t.val`, `s` may not be the root corresponding to `t` so you still need to check the cases where its left or right child is root.
- Code is cleaner if you use `isSame` as a separate function.
- Time complexity is $O(mn)$, where $m,n$ are the number of tree nodes. This is because for each node in `s`, you need to spend $O(n)$ time to check if it equals to `t`.
- Coding:  
  **Notice that when the function has side effects, you cannot directly use** `func(leftChild) && func(rightChild)` **because this may cause the right child to not be traversed!!!!**

## 4. Follow-up:
This problem asks if `t` is a *subtree* of `s`. "subtree" has to include leaf nodes. What if we change the problem to be "if `t` is a subpart of `s`"?

The difference is that we don't use `isSame`. Instead we use `contains(s, t)`, which returns if `s` contains `t` from root of `s`.
```java
// assumption: t is not null
boolean isSubpart(TreeNode s, TreeNode t) {
	if (s == null) {
		return false;
  }
  return contains(s, t) || isSubpart(s.left, t) || isSubpart(s.right, t);
}

// returns if s contains t as a subpart from root.
boolean contains(TreeNode s, TreeNode t) {
	if (t == null) { // check has been finished.
		return true;
  }
  if (s == null) {
  	return false;
  }
  if (s.val != t.val) {
  	return false;
  }
  return contains(s.left, t.left) && contains(s.right, t.right);
}
```
