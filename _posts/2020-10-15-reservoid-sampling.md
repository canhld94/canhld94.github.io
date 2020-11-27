---
layout: post
title: "Reservoir sampling"
date: 2020-10-15
tags: "C++ algorithm"
---

# Random sampling from a stream

Random sampling is interesting. In the most simple form, we want to randomly sample from an array `arr` such as each element in the array has the same probability to be chosen. Assume we have a good enough random generator, we can do it easily by generating an index in range `[0 arr.length-1]` and select the element at that index. Now consider the same problem, but the input is a stream of elements. Of course, we can buffer the elements to an array and do the same thing as was done above. However, can we do it without additional space?

The answer is yes. There is more than one approach, but I find Reservoir sampling is the easiest way to do it. Let call our stream `S` with 1-index, the algorithm can be explained as follow.

- First, we have a `cnt` variable to count the number of elements in the stream and `x` to hold the element that we will select
- For each current element of the stream `cur`, increase `cnt` by `1` and replace `x` with `cur` with `1/cnt` probability. The replacement decision can be determined by randomizing a number in range `[0 1]` and do the replacement if the number is smaller than `1/cnt`

We can prove when the stream ends, each element in the stream have an equal probability to be selected. The element at index `i` is selected if and only if all the following events happen:

- A replacement happens at `cnt == i`, and
- No replacement happens when `cnt > i`

Assume the total number in the Stream is `N`. The probability a replacement happens at `cnt == i` is `1/i`. The probability no replacement happens at `cnt == j > i` is `(1 - 1/i)`. So the probability of the element `is` is selected is:

<div align="center">
$P = \frac{1}{i} \times (1-\frac{1}{i+1}) \times ... \times \frac{1}{N} = \frac{1}{i} \times \frac{i}{i+1} \times ... \times \frac{N-1}{N} = \frac{1}{N} $
</div>

So far so good. The following code is the solution when the stream is a linked list ([Leetcode 328](https://leetcode.com/problems/linked-list-random-node/)):

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
  /** @param head The linked list's head.
      Note that the head is guaranteed to be not null, so it contains at least one node. */
  Solution(ListNode* _head) {
    head = _head;
    static random_device rd;
    gen.seed(rd());
    dis = uniform_real_distribution(0.0,1.0);
  }
    
  /** Returns a random node's value. */
  int getRandom() {
    auto node = head;
    int x = -1, cnt = 0;
    while (node) {
      cnt++;
      // generate number in (0,1) and do replacement if 
      // got a number smaller than 1/cnt
      if (dis(gen) < (double) 1.0 / cnt) {
        x = node->val;
      }
      node = node->next;
    }
    return x;
  }
private:
  mt19937 gen; // mersenne twister
  uniform_real_distribution<> dis; // random distribution
  ListNode* head;
};
```
I use the Mersenne twister engine `mt19937` to generate a number instead of the old C way `rand()`. The random library in `C++` with Mersenne twister engine looks quite horrible at first, but it\'s much better than `rand()`, so I think I need to get used to it.

# Random sampling of more than one element

Now, what if we want to select more than one element? The formal statement of the problem is: giving a stream `S` with size `N` (unknown at first) and a bucket with size `K`, how to randomly fill the bucket with the elements of the stream such as each element have the same probability to be selected. We still can use the Reservoir sampling algorithm to solve the problem. The algorithm is:

- First, we have a `cnt` variable to count the number of elements in the stream and `B` is the bucket that will hold the element
- For each current element of the stream `cur`, increase `cnt` by `1` and randomly replace `cur` with one of an element in `B` with `K/cnt` probability. 
- Similar to the above section, we can determine to do a replacement or not by randomizing a number in range `[0 1]` and do the replacement if the number is smaller than `K/cnt`
- If we do replacement and `B` is full, randomly choose an element to be evicted.

We will show that the above algorithm will select the element of `S` uniformly. The approach is different for the first `K` elements and the rest. For the first `K` element, they will always be chosen. So an element `x` remains until the end of the stream if and only if the following events happen when `cnt == i > K`

- Replacement does not happen, or
- If replacement happens, `x` will not be evicted

The probability the replacement does not happen is `1-K/i`, while the probability of replacement happens and `x` is not not chosen is `K/i*(1-1/K)`. So the probability `x` will not be replaced by another element when `cnt == i > K` is:

<div align="center">
$P_i = 1-\frac{K}{i} + \frac{K}{i} \times (1-\frac{1}{K}) = 1-\frac{K}{i} + \frac{K-1}{i} = 1-\frac{1}{i} = \frac{i-1}{i}$
</div>

It must happen for all `i = k+1,k+2,...,N`. Therefore, the probability `x` remains until the stream ends is:

<div align="center">
$P_i = P_{k+1} \times P_{k+2} \times ... \times P_N = \frac{K}{K+1}\times\frac{K+1}{K+2}\times...\times\frac{N-1}{N} = \frac{K}{N}$
</div>

For the rest of the stream, the proof is quite similar the Section 1. An element at index `i` (call `y`) is in `B` if and only if:

- Replacement happens at `cnt == i`, and
- No replacement is happen when `cnt == j > i`, or if there is, `y` is not evicted

We have shown that the probability an element in `B` is not evicted by element at index `j` is `(j-1)/j`. The probability that a replacement happens at `cnt == i` is `K/i`. So the probability `y` is in `B` when the stream ends is:

<div align="center">
$P_i = \frac{K}{i} \times \frac{i}{i+1} \times ... \times \frac{N-1}{N} = \frac{K}{N}$
</div>

So, we have shown that the probability of any element is in `B` at the end of `S` is `K/N`, which means our random sampling algorithm is uniform. The following code is the same problem in Section 1 but we select `k` elements.

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
  /** @param head The linked list's head.
      Note that the head is guaranteed to be not null, so it contains at least one node. */
  Solution(ListNode* _head, int _k) {
    head = _head;
    k = _k;
    static random_device rd;
    gen.seed(rd());
    dis0 = uniform_real_distribution(0.0,1.0);
    dis1 = uniform_int_distribution(0,k-1);
  }
    
  /** Returns a random node's value. */
  vector<int> getRandom() {
    auto node = head;
    int cnt = 0;
    vector<int> B(k);
    while (node) {
      cnt++;
      // generate number in (0,1) and do replacement if 
      // got a number smaller than K/cnt
      if (cnt <= k) {
        B[k-1] = node->val;
      }
      else if (dis0(gen) < (double) k / cnt) {
        int id = dis1(gen);
        B[id] = node->val;
      }
      node = node->next;
    }
    return x;
  }
private:
  mt19937 gen; // mersenne twister
  uniform_real_distribution<> dis0; // random distribution for replacement
  uniform_int_distribution<> dis1; // random distribution for eviction
  ListNode* head;
  int k;
};
```