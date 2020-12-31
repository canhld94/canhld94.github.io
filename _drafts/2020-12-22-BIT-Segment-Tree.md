---
layout: post
title: "Binary Index Tree and Segment Tree"
date: 2020-12-15
tags: "algorithm"
---

Binary index tree and segment tree sounds magical to me at first. I don't use segment tree very frequently, but I sometimes use binary index tree (BIT) in Leetcode contest. However, to be honest, I don't really understand BIT thoroughly (I get the idea, but still very confused why do we do `p += p&-p`). Today, I face a problem of range minimum query that I cannot solved. So, I take some study on both segment tree and BIT. So, I summary it here to ... my understanding.

# Binary Index Tree

BIT (or someone call it Fenwick Tree) is used to solve the range-sum query problem. Assume we have an array `arr` of size `n` (index `1`) and we want to do the two following operations on the array: 

- `sum(int p)`: get the sum of all elements from ``` to `p`
- `update(int p, int x)`: update the element `arr[p] += x`

If we store the plain array, then we can do `update` int `O(1)` but `sum` take `O(n)`. If we store the prefix sum array, then we can do `sum` in `O(1)` but `update` will take `O(n)` in the worst case. If we have, say, millions of mixed queries, then neither of above solutions work. BIT allow us to do both of operations in `O(logN)`. To get the idea of BIT, let take an example. Assume the array is `arr = [1,2,3,4,5,6,7,8,9,10,11,12`. Then we want to compute the `sum(11)`. Instead of compute it directly, let consider the binary representation of `11`. We have `11 = 8 + 2 + 1 = 2^3 + 2^1 + 2^0`. So, we can compute the sum of the first `11` elements compute the sum of first `8` elements, then next `2` elements, and finally the last `1` element. So, let store those sums as follow:

```
1   2   3   4   5   6   7   8   9   10    11    12     x
  1   3       10              36                  
          3       5   11          9    19           42
                          7                   11    
```

In the above example, the top row (0-th row) is the plain array and the below rows is how we store it in BIT. Start with index `1`, in the first row we will store the sum of sub-array with length `2^i` starting with `1`. Then in the second row, with the same rules, we fill all the gaps in the first row. We repeat the process until all position is filled. Now, let compute `sum(11)`. As we have, `11 = 8 + 2 + 1 = 2^3 + 2^1 + 2^0`, the sum of the first element (backward) is `11`, then the sum of the next two elements is `19`, then the last `8` elements is `36`, so `sum(11) = 11 + 19 + 36 = 66`. So what if we see the array as a tree, we start at `11` and then keep going `up` then `left`: that's `p -= p&-p`. The update is similar, but we will go `right` and `up`, which is `p += p&-p`.

# Segment Tree



<!-- 
There are N different packages. the ith package is of X[i] days and the price of that package is Y[i]. There are M customers. the jth customer wants the package of at least A[j] days and he doesn’t want to spend more than B[j] for any package. One package can accommodate at most one customer (Social Distancing) and a customer can buy at most one package. You have to find the maximum number of packages, you can sell. -->
