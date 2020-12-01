---
layout: post
title: "Graph advanced algorithm - Disjoint Set"
date: 2020-10-15
tags: "algorithm graph union-find C++"
---

Disjoint set, or UnionFind, is a magical data structure when we first hear about it. Disjoint set is the correct name of the data structure, but I usually call it UnionFind - which represents the two methods that make it so famous: `unite` and `find`. Assume we have `n` components. Two components is connected if there is an edge between them. The connection is transitive: if `x` connects to `y` and `y` connects to `z` then `x` connects to `z`. We call the connected components a group. The UnionFind allow us to do the following operation:

- `unite(a,b)`: connect two components `a` and `b`, thus unite the corresponding groups of `a` and `b`
- `find(a)`: find the group that `a` belong to, so if we have `find(a) == find(b)` then `a` and `b` is connected

So, UnionFind is useful when we don't care about the underlying connections but only want to know if two or some components is connected. The below codes is the simplest form of union-find.

```cpp
class UnionFind {
private:
    std::vector<int> index;
public:
    // init an UF with n elements
    UnionFind(int n) {
        index = std::vector<int>(n);
        rank = std::vector<int>(n,1);
        std::iota(index.begin(),index.end(),0);
    }
    // connect two components a and b
    bool unite(int a, int b) {
        int x = find(a); // find root of a
        int y = find(b); // find root of b
        if (x == y) return false; // if they are already in same set, return
        index[x] = y;
        return true;
    }
    int find(int a) {
        while (a != index[a]) {
            a = index[a];
        }
        return a;
    }
};
```

We assume that `n` components are labeled from `0` to `n-1`. Here, we use a tree to store the connected components. `index[i]` is the index of the root of components `i`. Originally, the root of a component is itself. This will keep for a root component, i.e. a component `x` is root if and only if `x == index[x]`. In the `find(int a)` method, we want to find the root index of node `a`, so we recursively assign `a` to `index[a]` until they have the same value. In the method `unite(int a, int b)`, we want to connect to a group of `a` and `b`. Assume `a`'s root is `x` and `b`'s root is `y`, we can connect two groups by making `y` become root node of `x`, i.e. `index[x] = y`.

The above implementation works so far, however, there are some flaws:

- In `unite`, we blindly made `y` become the root of `x`; in the worst case, when `x` always have more components than `y`, this will make the tree look like a linked list and make the complexity of `find` be `O(N)`
- We don't care about the connection, so we don't need too many intermediate root node, all components in one group have one root is enough

This can be addressed by two techniques: union by rank and path compression. In union by rank, we maintain an additional `rank` array which keeps the number of components of a group (if component `i` is not a root node, `rank[i] = 0`). When we do connect to the roots, we make the root with a higher rank to be the final root. Union by rank made the tree balance and thus, have height `log(N)`. Path compression does not change the height of the tree in the worst case, but it can make reduce the height of the tree by half for each time we call `find` by bypassing one intermediate root in each iteration while moving up. The two techniques make both `unite` and `find` now have `log(N)` time complexity. The below code is the union-find template that I usually use when doing leetcode

```cpp
class UnionFind {
private:
  std::vector<int> index;
  std::vector<int> rank;
  int num_groups;
public:
  // init an UF with n elements
  UnionFind(int n) {
    index = std::vector<int>(n);
    rank = std::vector<int>(n,1);
    std::iota(index.begin(),index.end(),0);
    num_groups = n;
  }
  // connect two components a and b
  bool unite(int a, int b) {
      a = find(a); // find root of a
      b = find(b); // find root of b
      if (a == b) return false; // if they are already in same set, return
      if (rank[a] > rank[b]) { // we want to connect small tree to big tree to make the tree balance
          std::swap(a,b);
      }
      index[a] = b;
      rank[b] += rank[a];
      rank[a] = 0;
      num_groups--;
      return true;
  }
  int find(int a) {
      // find root of the element id
      // remark that index[id] is the root of element id
      // therefore, we can recursively search until id == index[id]
      // then id will be the root node
      while (a != index[a]) {
          index[a] = index[index[a]]; // bypassing current intermediate root node
          a = index[a];
      }
      return a;
  }
  bool idConnected(int a, int b) {
      // find is there any connection between a and b
      // they are connected if they have same root
      return find(a) == find(b);
  }
  int count(int a) {
      return rank[find(a)];
  }
  int count() {
    return num_groups;
  }
};
```

If you want a more detailed explanation, you can check out [this slide](https://www.cs.princeton.edu/~rs/AlgsDS07/01UnionFind.pdf)