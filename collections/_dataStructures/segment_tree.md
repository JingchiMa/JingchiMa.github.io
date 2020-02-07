---
title: "Segment Tree"
layout: single
name: segment tree
---
## Goal
Segment tree can handle the following problems:  
1. split an interval into multiple intervals to aggregate an attribute. (Canonical Covering Aggregation)
2. find all the intervals that contain a given point.

## Structure:
Each node has the following data:
- interval: interval corresponding to this node.
- key: the number that splits the interval of the node. (seems unnecessary)
- left, right: pointing to left and right child
- auxiliary data structure: contains the attribute corresponding to the interval.

<img src="/assets/images/dataStructures/segment_tree" style="float: left; margin-right: 10px;" />


## Construction
### Step 1: build the segment tree based on *elementary intervals*.

1. Elementary intervals are consecutive, and usually are the smallest unit. Firstly one needs to find all the elementary intervals, and sort them. If there are $n$ intervals in total, then the time complexity of this step is $O(n\log{n})$.


2. Build a *balanced* binary tree based on the elementary intervals. The algorithm is as follows:
```
func build(elementary_intervals):
  split the elementary intervals into two halves, left_intervals and right_intervals
  create a node, whose key is the separator for the two halves
  node.left = build(left_intervals)
  node.right = build(right_intervals)
  return node
```
Note that the tree constructed by this algorithm is guaranteed to be balanced. The time complexity of this step is $O(n)$.

### Step 2: insert all intervals into the segment tree
Each interval is placed into the nodes whose intervals are *canonically covered* by it. We first introduce *Canonical Covering*.

#### Definition: *Canonical Covering*:
A node $N$ is in the **canonical covering** of an interval $I$ iff the interval $I_N$ of the node $N$ satisfies $I_N \subseteq I$ and its parent does not.

#### Theorem:
For $\forall$ interval $I$, on each level, there are at most two nodes that are canonically covered by $I$.

<img src="/assets/images/dataStructures/segment_tree_number_of_canonical_covering" style="float: left; margin-right: 10px;" />

So for each interval, we insert it into *all* the nodes that are canonically covered by it. The algorithm is as follows:

```
func interval_insertion(root, interval):
  if root.interval is a subset of interval:
    add interval to the auxiliary data structure of root
    return
  if interval intersects with root.left.interval:
    interval_insertion(root.left, interval)
  if interval intersects with root.right.interval:
    interval_insertion(root.right, interval)
```
What's the time complexity fo this algorithm? The key point to analyze this is to check the time usage *level by level*.

- Which nodes can be visited?  
  *Answer*: node $N$ that is canonically covered by *the* interval, or whose interval intersects with *the* interval.
- How many nodes can be visited at most on each level?
  Answer: Four.
  - At most 2 nodes are canonically covered by it.
  - How many nodes have interval that overlap with *the* interval? At most $2$.
  > Suppose the interval is [x, x']. There is at most one node whose interval contains x and one contains x'.

    Recall that [x, x'] does not canonically cover the parent!

<img src="/assets/images/dataStructures/segment_tree_time_complexity_insert_interval" style="float: left; margin-right: 10px;" />

## Mumblings
