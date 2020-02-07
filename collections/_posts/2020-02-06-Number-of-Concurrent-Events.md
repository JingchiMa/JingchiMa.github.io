---
title: "Accumulated Value of Concurrent Events"
categories:
  - Leetcode
tags:
  - intervals
  - leetcode-collection
---
A set of `Interval` problems can be viewed as one single problem of calculating an accumulated value from a list of concurrent events.

## Overview
Leetcode `253 Meeting Rooms II`, `Calendar I,II,II`, and `1094 Car Pooling` can treated as the same problem: **Calculate the accumulated value of concurrent events**.

The problem can be described as follows:  
Given a list of intervals `[a, b)`, each associated with an *delta* on an accumulable variable, calculate the maximum accumulable values along the way.

## Solution Key Points
The intuition is that, for any point in an *elementary interval [a, b)*, the value of the accumulable variable is the same. Therefore, we only need to *record the deltas on every endpoint of the intervals* because the value of the accumulable variable will remain the same until you hit the next endpoint!

If the minimum time unit is known, and there is a maximum value for timestamp, we can use `Array` to record the deltas on each timestamp. Otherwise `TreeMap` can be used in the more general cases. The sketch of the algorithm is as follows:
```
for each interval,
  update the treemap using interval.left and interval.end and their deltas

scan all the endpoints stored in the treemap, and calculate the accumulative value along the way.
```

## Make Things Faster?
The solution mentioned seems to be somehow slow because each time a new interval comes, all the previous endpoints need to be scanned. This is because *intervals may not come in order according to the start time*. For example, `[3, 5)` may come before `[1, 2)`.

But if either
- we are given all the intervals beforehand

or

- the streaming intervals are guaranteed to come in the ascending order by `interval.start`

then we don't need to scan all the previous end points any more. We just need to maintain a variable recording *the current variable value so far*, and a `PriorityQueue` containing all the timestamps whose delta hasn't been added into the `value` yet.

Data structures:
```
int value        // variable value so far
PriorityQueue pq // all timestamps with deltas not added to value yet, sorted in ascending order.
```

Algorithm sketch:
```
for each new interval,
  pop timestamp out of pq until it > interval.start
  update value using the deltas
  print value
```
