---
title: "Leetcode 56: Merge Intervals"
categories:
  - Leetcode
tags:
  - intervals
  - leetcode
---
One of the typical ways to solve this problem is to sort the input intervals first. But this is not possible when the intervals are given as a stream. The following two methods apply to the streaming case as well, with time complexity $O(n)$ for every new interval.

## Solution I: Leetcode 57 Insert intervals
In streaming case, the problem can be described as follows:
> Given a sorted list of intervals, and a new interval, merge it into the sorted list such that it does not contain overlapping intervals.

This is exactly the same as Leetcode 57. The way to solve it is very similar to **merge two sorted list**.

## Solution II: Accumulative Value of Concurrent Events
The technique in [here]({% post_url 2020-02-06-Number-of-Concurrent-Events %}) can also be applied to this problem. Here we define the accumulative value for each timestamp ts as *the number of intervals $[a, b] s.t. a \leq ts < b$*.

Whenever the value reaches 0 at timestamp $ts$, we know that there is no intervals in range $(ts, next)$, which means there are no overlapping intervals anymore, and hence the current interval ends.

```java
public int[][] merge(int[][] intervals) {
    TreeMap<Integer, Integer> timestamps = new TreeMap<>();
    for (int[] interval : intervals) {
        timestamps.put(interval[0], timestamps.getOrDefault(interval[0], 0) + 1);
        timestamps.put(interval[1], timestamps.getOrDefault(interval[1], 0) - 1);
    }
    List<int[]> res = new ArrayList<>();
    int count = 0;
    Integer start = null;
    for (Map.Entry<Integer, Integer> entry : timestamps.entrySet()) {
        count += entry.getValue();
        if (start == null) {
            start = entry.getKey();
        }
        int end = entry.getKey();
        if (count == 0) {
            res.add(new int[] {start, end});
            start = null;
        }
    }
    return res.toArray(new int[res.size()][2]);
}
```
