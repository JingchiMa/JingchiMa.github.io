---
title: "Leetcode 721. Accounts Merge"
categories:
  - Leetcode
tags:
  - graph
  - connected components
  - leetcode
---

## 1. Problem Description
Given a list accounts, each element `accounts[i]` is a list of strings, where the first element `accounts[i][0]` is a name, and the rest of the elements are emails representing emails of the account.

Now, we would like to merge these accounts. Two accounts definitely belong to the same person if there is some email that is common to both accounts. Note that even if two accounts have the same name, they may belong to different people as people could have the same name. A person can have any number of accounts initially, but all of their accounts definitely have the same name.

After merging the accounts, return the accounts in the following format: the first element of each account is the name, and the rest of the elements are emails in sorted order. The accounts themselves can be returned in any order.

## 2. Solution Key Points
### Method I: Build Graph + BFS
As long as there is one common emails between two email lists, all the emails in them should be connected. Each connected component represents a person. Therefore, a natural way to solve this problem is to build a graph to represent the connectivity, and find every connected components.

**We want to build a graph / use union find because every email only give partial information on connectivity**.

One problem is *What should be the nodes in the graph*. Naturally one would think about using *emails* as nodes. Scan all the emails and create adjacency list for each node. Note here we do not need to build a *complete* graph, where every email has an edge pointing to every other email in the cluster. Instead we only need to connect one email to another one because **we only need to guarantee that these emails are connected**.
![emailAsNode](/assets/images/leetcodes/721/emailAsNode.png)

But actually we do not necessarily use email as node. Instead we can directly use *accounts*. Two accounts are connected if they share the same email so we only need to maintain a map mapping from email to account. If an email is already in the map, we connect the current account with the previous account. During traversal we only need to add all the corresponding emails into results.
![accountAsNode](/assets/images/leetcodes/721/accountAsNode.png)

Need to choose a representative node for each connected component.
### Method II: Union Find
Why using union find?

For this problem I don't think union find is a better solution than BFS. Union find is suitable for connection check, especially when we have dynamic updates. For this problem though, we are trying to find all the connected components so we have to scan the whole graph anyway.

## Mistakes
1. Failed to identify this is actually a problem to find connected components.
2. No need to build a complete graph. Just need to make sure every email in the same list is connected.
3. forgot to create a node for every account. The following code will fail to create node when there is only 1 email in the account.
   ```java
   int accountId = 0;
   for (List<String> account : accounts) {
       // mistake: forget to create a node for every account here.
       for (String email : account.subList(1, account.size())) {
           Integer prevId = emailToAccount.get(email);
           if (prevId == null) {
               emailToAccount.put(email, accountId);
           } else {
               graph.putIfAbsent(accountId, new ArrayList<>());
               graph.putIfAbsent(prevId, new ArrayList<>());
               graph.get(accountId).add(prevId);
               graph.get(prevId).add(accountId);
           }
       }
       accountId++;
   }
   ```
