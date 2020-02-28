---
title: Automatically Refactoring Kubernetes Config Files into Kustomize Structure
---

## 1. Motivation
In practice people often have similar applications, or applications belonging to the same categories. They often share similar kubernetes config files. As the number of applications grows larger, the config files can have a lot of duplicates and become more difficult to manage. [Kustomize](https://github.com/kubernetes-sigs/kustomize) offers a decorative way to reuse kubernetes config files, which can help with this issue. But first, the old config files need to be refactored into "Kustomize structure". Given that number of old config files can be large, it is hard to manually refactor all of them. Therefore, an automatic script to to the refactoring will be ideal.

## 2. Overview
The rough idea can be described as:
- First extract the "common parts" from the config files as `base`.
- Then use the "complement" for every config file w.r.t. the `base` as `patch`.

One trick thing is how to define "common parts", which is discussed in the next section.

## 3. Define *Common*
I'll start from the naive way and gradually modify its definition. The major goal is to maximize `base` and minimize `patch`.

### 3.1 Naive definition
Given that all config files are `yaml` file, there are three types to consider: `list`, `dict` and `literal`. One intuitive way to define common parts is as follows:
- `literal`:
  Given `literal1` and `literal2`, the common part is `literal1` if `literal1 == literal2`, null otherwise.
- `list`:
  Given `list1` and `list2`, the common part is `list1` if `list1 == list2`, null otherwise.
- `dict`:
  Given `dict1` and `dict2`, the common part will be a `dict` with common keys whose value is determined by recursive call with  `dict1[common_key]` and `dict2[common_key]`.

This definition is actually the same as finding common path between two trees, and treat `literal` and `list` as leaf nodes.

The problem with this approach is that the common part can be very small due to the way to compare `list`. For example, `list1` and `list2` may not be exactly the same, but they share a lot of same or similar elements. As a result, instead of directly returning null, we would like to *further compare the elements in the lists*. This leads us to the next definition.

### 3.2 Unwrap list
As mentioned in 3.1, elements in the list also need to be compared. The tricky problem is, **how to pair elements in two lists**?

#### Method I: same index
A natural way is to pair elements on the same index. The problem is that in many cases, the order of elements in the list does not actually matter so there may not be many common parts between the two compared elements.

#### Method II: identifier
Some list objects have `identifier` as a field. This essentially can convert the list into a map. We can pair the elements with the same identifier. The problem with this approach is that sometimes the only difference between two elements is the identifier ... (YES). This is because people name elements differently in different applications.

#### Method III: dfs
One brute force way is to try all the possible ways to pair, and define a way to evaluate each. Two problems with this approach:
- may take a long time when lists are long.
- not easy to define a value function for each combination.

Although this can be a general way out, I did not choose it because of the complexity.

#### What should we do?
It turns out that it may be hard to figure out a general approach for all applications. 


## 4. Plugins
