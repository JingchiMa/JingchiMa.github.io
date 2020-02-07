---
title: "Interval Tree (1D)"
name: interval tree
summary: static tree capable of finding intervals that overlap with the given interval.
---

## Goal
What problems are interval trees able to solve (efficiently)?

**Enclosing Interval Searching Problem**: Given a set of intervals and a point, find all (or k) intervals that contains that point.  
**Overlapping Interval Searching Problem**: Given a set of intervals, and an interval, find all (or k) intervals that overlap with the given interval.

## Data Structure
This version of interval tree is a static data structure. For every node in the interval tree, the following data is stored:
- key: the median of end points of current intervals.  
  The key of current node splits the interval set $S$ into three parts:
  - $S_l$: all the intervals whose high endpoint $hi < key$
  - $S_m$: all intervals with $lo \leq key \leq hi$.
  - $S_r$: all the intervals whose low endpoint $lo > key$

- left, right pointers: pointing to left child and right child.

- augmented data structure: two sorted arrays containing intervals in $S_m$
  - the 1st sorted array is sorted according to low endpoint $lo$
  - the 2nd sorted array is sorted according to high endpoint $hi$

  Note that the intervals associated with the node are stored in this augmented data structure. And by using this two sorted array it's easy to get the median of the endpoints in linear time.

### Construction
The construction process is the same as the definition.

```java
public static Node constructIntervalTree(List<Interval> intervals) {
    List<Interval> sortedByLos = new ArrayList<>(intervals);
    sortedByLos.sort((i1, i2) -> i1.lo - i2.lo);
    List<Interval> sortedByHis = new ArrayList<>(intervals);
    sortedByHis.sort((i1, i2) -> i2.hi - i1.hi);
    return constructHelper(sortedByLos, sortedByHis);
}

// sortedByLos is sorted in ascending order, while sortedByHis is sorted in descending order.
private static Node constructHelper(List<Interval> sortedByLos, List<Interval> sortedByHis) {
    if (sortedByHis.isEmpty()) {
        return null;
    }
    int key = findMedian(sortedByLos, sortedByHis);
    List<List<Interval>> intervalSetsLo = getIntervalSets(sortedByLos, key);
    List<List<Interval>> intervalSetsHi = getIntervalSets(sortedByHis, key);
    Node root = new Node(key, intervalSetsLo.get(1), intervalSetsHi.get(1));
    root.left = constructHelper(intervalSetsLo.get(0), intervalSetsHi.get(0));
    root.right = constructHelper(intervalSetsLo.get(2), intervalSetsHi.get(2));
    return root;
}

private static List<List<Interval>> getIntervalSets(List<Interval> intervals, int key) {
    // contains left, mid, right parts
    List<List<Interval>> results = new ArrayList<>();
    List<Interval> left = new ArrayList<>();
    List<Interval> mid = new ArrayList<>();
    List<Interval> right = new ArrayList<>();
    for (Interval interval : intervals) {
        if (interval.hi < key) {
            left.add(interval);
        } else if (interval.lo > key) {
            right.add(interval);
        } else {
            mid.add(interval);
        }
    }
    results.add(left);
    results.add(mid);
    results.add(right);
    return results;
}

// sortedByLos and sortedByHis have the same size and >= 1
private static int findMedian(List<Interval> sortedByLos, List<Interval> sortedByHis) {
    int count = sortedByHis.size();
    int median = 0; // initial value should not be used
    int i = 0;
    int j = sortedByHis.size() - 1;
    while (count > 0) {
        if (sortedByLos.get(i).lo <= sortedByHis.get(j).hi) {
            median = sortedByLos.get(i).lo;
            i++;
        } else {
            median = sortedByHis.get(j).hi;
            j--;
        }
        count--;
    }
    return median;
}
```

## Complexity Analysis
- *The time complexity of construction is $O(n\log{n})$ where $n$ is the number of intervals.*  
- *The space complexity is $O(n)$.*

At the first glance one may think that the time complexity for building the tree is worst case $O(n^2)$ because this tree is not necessarily a *balanced* binary tree. Indeed it's true that the built tree may not be balanced. But the time complexity is still $O(n\log{n})$.

Notice that each time we pick the median of all endpoints, meaning that for the left or right subtree, the maximum number of endpoints is $n$, where $n$ is the number of intervals. Therefore, the maximum number of *intervals* on each subtree is $n/2$ so we have
\[ T(n) \leq 2T(n/2) + O(n) \]
which yields $T(n) = O(n\log{n})$.

Also by the same analysis we can show that **the depth of the tree is indeed $O(logn)$**.

As for the space complexity, notice that there are at most $O(n)$ tree nodes, and each interval is in one node. Therefore the space complexity is $O(n)$.

## Query
**Query: Given a set S of intervals, return k intervals that are overlapped with the target interval $I$.**

### Algorithm
Suppose $v$ is the current node, and $Q$ is the query interval.
```
if v.key is contained in Q then
  add v.intervals into results
  recursive_call_on_left_subtree
  recursive_call_on_right_subtree
else if Q.hi < v.key
  add all intervals in v.intervals whose lo <= Q.hi
  recursive_call_on_left_subtree
else
  add all intervals in v.intervals whose hi >= Q.lo
  recursive_call_on_right_subtree
```

### Complexity
time: $O(\log{n} + k)$
