---
title: "Leetcode 1109. Corporate Flight Bookings"
categories:
  - Leetcode
tags:
  - intervals
  - leetcode
---

This problem also belongs to the series mentioned [here]({% post_url 2020-02-06-Number-of-Concurrent-Events %}). The only difference is that the intervals in this problem is of format $[a, b]$, in other words both ends are **inclusive**. Fortunately the intervals in this problem only have integer ends. Therefore we can easily convert $[a,b]$ to $[a, b+1)$ and apply the same technique.

```java
public int[] corpFlightBookings(int[][] bookings, int n) {
    int[] seatDeltas = new int[n + 2]; // mistake: flights are numbered from 1 to n.
    for (int[] booking : bookings) {
        seatDeltas[booking[0]] += booking[2];
        seatDeltas[booking[1] + 1] -= booking[2];
    }
    int[] res = new int[n];
    int count = 0;
    for (int i = 0; i < res.length; i++) {
        count += seatDeltas[i + 1];
        res[i] = count;
    }
    return res;
}
```
