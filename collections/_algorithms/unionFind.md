---
title: "Union Find"
---

## 1. Overview
Union find is a data structure on *undirected graph* that supports the following two operations:

```
find(node_id) # returns the root node_id of the given node_id

union(node_id1, node_id2) # connects node_id1 and node_id2
```

UnionFind is often used to:
- form connected components
- check connectivity for streaming updates

## 2. Algorithm
```
class UnionFind
  parents # parents[node_id] contains the parent node of the given node
  ranks   # if no path compression, ranks[node_id] is the depth of the tree rooted at node_id

  func find(node_id):
    traverse along the parent pointer until reaches the node whose parent is itself.
    as traversing, change the parent pointer to the root.

  func union(node1, node2):
    root1, root2 = find(node1), find(node2)
    if ranks[root1] < ranks[root2],
      parents[root1] = root2 # append the root1 tree to root2
    else if ranks[root1] > ranks[root2],
      parents[root2] = root1
    else, # same rank, arbitrarily appending
      parents[root1] = root2
      ranks[root2] += 1 # depth increased by 1
```

![find](/assets/images/algorithms/find.png)
![union](/assets/images/algorithms/union.png)

## 3. Implementations
```java
class UnionFind {
  int[] parents;
  int[] ranks;
  // n is the number of nodes. Node ids from 0 to n-1.
  public UnionFind(int n) {
    parents = new int[n];
    ranks = new int[n];
    for (int i = 0; i < n; i++) {
      parents[i] = i; // initially every node itself is a cluster
      ranks[i] = 0;
    }
  }

  public int find(int nodeId) {
    if (parents[nodeId] == nodeId) {
      return nodeId;
    }
    int root = find(parents[nodeId]);
    parents[nodeId] = root; // path compression
    return root;
  }

  public void union(int node1, int node2) {
    int root1 = find(node1);
    int root2 = find(node2);
    if (ranks[root1] < ranks[root2]) {
      parents[root1] = root2;
    } else if (ranks[root1] > ranks[root2]) {
      parents[root2] = root1;
    } else {
      parents[root1] = root2;
      ranks[root2]++;
    }
  }
}
```

## 4. Time and Space Complexity
If the number of nodes is n, clearly the space complexity is $O(n)$.

As for time complexity, with only "union by rank", "rank" is essentially the height of subtree. Therefore, one can prove that for every node, the heights of its subtrees can differ at most by 1. As a result the time complexity is $O(\log{n})$ for each `find` and `union` operation.

With path compression, the analysis is a little more tricky. You can check more on this [here](https://en.wikipedia.org/wiki/Disjoint-set_data_structure).
