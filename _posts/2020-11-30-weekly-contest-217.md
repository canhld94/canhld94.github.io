---
layout: post
title: "Leetcode weekly contest 217 review"
date: 2020-11-29
tags: "C++ algorithm leetcode competitive-programming"
---

In Leetcode contests, I usually solve 3 problems, and sometimes 4. I don\'t make the last question in time quite often, but usually, I have the right direction and just don\'t have enough time to finish it due to (a little) weak coding fluency. Recently, I start learning more advanced data structures (Disjoint set, spare table, binary index tree...) and I found the hard question is becoming easier.

Leetcode 217 is different. It\'s pure logic contest, and doesn\'t require any complex data structure. I make the Q2 with stack but I was completely lost with Q3. I cannot think of any approach to solve the problem for almost 45 minutes. I came up with one hypothesis in the last 15 minutes and quickly implement it, but it turned out to be a wrong hypothesis. And of course, I didn\'t even have time to think about Q4. That was so embarrassing. Therefore, I decided to write a review for all problems (except Q1) for this contest.

# Question 2: Most competitive subsequence

```
Given an integer array nums and a positive integer k, return the most competitive subsequence of nums of size k.

An array\'s subsequence is a resulting sequence obtained by erasing some (possibly zero) elements from the array.

We define that a subsequence a is more competitive than a subsequence b (of the same length) if in the first position where a and b differ, subsequence a has a number less than the corresponding number in b. For example, [1,3,4] is more competitive than [1,3,5] because the first position they differ is at the final number, and 4 is less than 5.

Example 1:

Input: nums = [3,5,2,6], k = 2
Output: [2,6]
Explanation: Among the set of every possible subsequence: {[3,5], [3,2], [3,6], [5,2], [5,6], [2,6]}, [2,6] is the most competitive.

Example 2:

Input: nums = [2,4,3,3,5,4,9,6], k = 4
Output: [2,3,3,4]
 
Constraints:

1 <= nums.length <= 105
0 <= nums[i] <= 109
1 <= k <= nums.length
```

In short, we have an array with size `n`, and we should find the lexicographical smallest subsequence with size `k`. The first approach that came to my mind is some kind of sorting. To have the smallest subsequence, we should start with the smallest possible elements. Say, if we found the smallest element at index `i` and we find that we still have more than `k-1` elements after it, we should start with `arr[i]`. So I thought about sorting the index of the array and greedily select the element. Then I found it\'s too complicated and error-prone. So, I tried to think in another direction. Usually, when working with subsequence, stack is a very good choice. So I try with stack and come up with the following algorithm (that still, based on the idea of smaller elements should come first):

- Initialize an empty stack `stk`, this will keep the subsequence we want to find
- Iterate the array from begin to end, keep track of how many elements remains
- For each iteration, let call the current element `cur`, our stack size is `have` and the remaining elements `cnt`
  - If the stack is empty, or the remains element just enough for us to make exactly a `k-size` subsequence (that is `have + cnt == k`), we have no choice but include the current element in our answer
  - If we found that we have `have + cnt > k`, we know that we can remove some element in the stack
  - If `cur >= stk.top()` just include it in the stack
  - if `cur < stk.top()`, we know we can make a better subsequence; pop out the stack until `cur >= stk.top()`, or until we have just enough element to make `k-size` subsequence, then push `cur` to the stack.

In the end, we may have more than `k` element in the stack, but the first `k` element of the stack is the subsequence we want to find. Later, I fixed this by only push when `cur >= stk.top()` if we have less than `k` element in the `stk`. This will make my stack have only `k` elements.

```cpp
class Solution {
public:
    vector<int> mostCompetitive(vector<int>& nums, int k) {
        const int n = nums.size();
        vector<int> stk;
        int cnt = n;
        for (int  i = 0; i < n; ++i) {
            auto &v = nums[i];
            int have = stk.size(); // currently have
            if (have == 0 || have + cnt == k) { // only room left of k
                stk.push_back(v);
            }
            else { // have + cnt > k
                if (v >= stk.back()) {
                    if (have < k) { // only push if have less than k in stk
                        stk.push_back(v);
                    }
                }
                else {
                    // cout << have << " " << v << endl;
                    while (have > 0 && have + cnt > k && stk.back() > v) {
                        stk.pop_back();
                        have--;
                    }
                    stk.push_back(v);
                }
            }
            cnt--;
        }
        return stk;
    }
};
```

The complexity is `O(N)` times and `O(N)` space. When the contest ended, I check other solutions and I think I have come with an optimal solution during the contest.

# Question 3: Minimum Moves to Make Array Complementary

```
You are given an integer array nums of even length n and an integer limit. In one move, you can replace any integer from nums with another integer between 1 and limit, inclusive.

The array nums is complementary if for all indices i (0-indexed), nums[i] + nums[n - 1 - i] equals the same number. For example, the array [1,2,3,4] is complementary because for all indices i, nums[i] + nums[n - 1 - i] = 5.

Return the minimum number of moves required to make nums complementary.

Example 1:

Input: nums = [1,2,4,3], limit = 4
Output: 1
Explanation: In 1 move, you can change nums to [1,2,2,3] (underlined elements are changed).
nums[0] + nums[3] = 1 + 3 = 4.
nums[1] + nums[2] = 2 + 2 = 4.
nums[2] + nums[1] = 2 + 2 = 4.
nums[3] + nums[0] = 3 + 1 = 4.
Therefore, nums[i] + nums[n-1-i] = 4 for every i, so nums is complementary.
Example 2:

Input: nums = [1,2,2,1], limit = 2
Output: 2
Explanation: In 2 moves, you can change nums to [2,2,2,2]. You cannot change any number to 3 since 3 > limit.
Example 3:

Input: nums = [1,2,1,2], limit = 2
Output: 0
Explanation: nums is already complementary.
 
Constraints:

n == nums.length
2 <= n <= 105
1 <= nums[i] <= limit <= 105
n is even.
```

So you have an array, and you can replace some elements in the array with a number from `1` to `limit` to make the `arr[i]` and `arr[n-1-i]` sum to the same value to every `i`. Your task is to minimize the number of replacements. Sound pretty easy. But it\'s easy only if you go in the right direction. At the first look, we want `arr[i] + arr[n-1-i] == k` for every `i`, so it\'s nature to think about finding the value of `k` first. I made the mistake here, as I think `k` should be a specific value and try to find it. Then I come with a hypothesis that if we compute all value `arr[i] + arr[n-1-i]` and store it in an array, `k` the elements with the highest frequency in the array, or the median, or mean... But it turns out that they are all wrong. `k` can be anywhere in the range `2` to `2*limits`, and we need to consider all of them. It is possible because, for each index `i`, we can make at most two moves. Depending on the value of `k`, we can determine how many moves should we make. Let `a,b = minmax(arr[i], arr[n-1-i])`, there is five possible range of `k`

- `2 <= k < a+1`: we need to make two moves to make both `a` and `b` smaller
- `a+1 <= k < a+b`: we need to make one move to make `b` smaller
- `k == a+b`: we don\'t need any moves
- `a+b+1 <= k < b+limit+1`: we need one move to make a bigger
- `k >= b+limit+1`: we need two moves to make both `a` and `b` bigger

So, we will have an array `target` from `2` to `2*limit + 1`; and `target[k]` stores the number of moves needed to make all our given array complementary at `k`. But how to fill `target`. Naively, for each `k`, we need to iterate `arr` and for each element, find out where is `k` locate in one of five cases above. That will make an `O(kn)` solution, which is not sufficient. We can make it better. If we look carefully at the way `target[k]` is computed, we can see the pattern of interval sum here. And for the interval sum, there is a much better way to calculate `target`: calculate the adjacent difference, and then find the final answer with accumulation. That can be done easily by adding the corresponding increment or decrement amount at the left endpoints of each interval.

```cpp
class Solution {
  public:
  int minMoves(vector<int>& arr, int k) {
    const int n = arr.size();
    vector<int> delta(2*limit+2, 0); // this is the target array in form of adj diff
    for (int i = 0; i < n-i-1; ++i) {
      auto [a,b] = minmax(arr[i],arr[n-i-1]);
      // first interval 2 <= k < a+1, we need 2 move, cur_move = 2
      delta[2] += 2;
      // second interval a+1 <= k < a+b, we have only 1 moves, so decrease by 1, cur_move = 1
      delta[a+1] -= 1;
      // third interval k == a+b, we have 0 moves, so decrease by 1, cur_move = 0
      delta[a+b] -= 1;
      // forth intervals a+b+1 <= k < b+limit+1, we need 1 move, so increase by 1, cur_move = 1
      delta[a+b+1] += 1;
      // fifth intervals b+limit+1 <= k < 2*limit+1, we nee 2 move, so increase by 1, cur_move = 2
      delta[b+limit+1] += 1; 
    }
    // calculate the actual target array with accumulation
    int res = n, acc = 0;
    for (int i = 2; i < 2*limit+2; ++i) {
      acc += delta[i];
      res = max(res,acc);
    }
    return ret;
  }
}
```

# Question 4: Minimum deviation of an array

```
You are given an array nums of n positive integers.

You can perform two types of operations on any element of the array any number of times:

If the element is even, divide it by 2.
For example, if the array is [1,2,3,4], then you can do this operation on the last element, and the array will be [1,2,3,2].
If the element is odd, multiply it by 2.
For example, if the array is [1,2,3,4], then you can do this operation on the first element, and the array will be [2,2,3,4].
The deviation of the array is the maximum difference between any two elements in the array.

Return the minimum deviation the array can have after performing some number of operations.

Example 1:

Input: nums = [1,2,3,4]
Output: 1
Explanation: You can transform the array to [1,2,3,2], then to [2,2,3,2], then the deviation will be 3 - 2 = 1.
Example 2:

Input: nums = [4,1,5,20,3]
Output: 3
Explanation: You can transform the array after two operations to [4,2,5,5,3], then the deviation will be 5 - 2 = 3.
Example 3:

Input: nums = [2,10,8]
Output: 3
 
Constraints:

n == nums.length
2 <= n <= 105
1 <= nums[i] <= 109
Accepted
```

That is, again, some logical problem. I took a look at this during the contest and then just decided to go back to Q3 because it looks similar to Q3. The idea is whatever we\'ve done, within the given operators, each element will only have limited values. If an element is odds, we can only double it one time. If an element is even, we can only divide it by `2` until it\'s odd. So the naive solution will be finding all `bound` for each element and try all possible combinations. However, it\'s not necessary. We only want to maximize the `min` value and `minimize` the max value. So, if we currently have an even `max` value, we can only divide it by `2`. Since can start from any states, we can first make all element become even. We will divide the `max` of the array by `2` and keep track of the deviation until we cannot do so. The following code is from [Richard Liu](), also who inspires me to start this blog.

```cpp
class Solution {
  public:
  int minimumDeviation(vector<int>& nums) {
    priority_queue<int> snapshot;
    int mini = numeric_limits<int>::max(), res = mini;
    for (auto &v : nums) {
      if (v%2 == 1) {
        v *= 2;
      }
      mini = min(mini,v);
      snapshot.push(v);
    }
    while (1) {
      int maxi = snapshot.top();
      snapshot.pop();
      res = min(res,maxi - mini);
      if (maxi%2 == 1) break;
      snapshot.push(maxi/2);
      mini = min(mini,maxi / 2);
    }
    return res;
  }
};
```

# Afterthoughts

This contest is very good I think. The problem doesn\'t require any specific knowledge of data structure and algorithms but stresses logical thinking. But it\'s also embarrassing for me because it reveals that my logical thinking still has many weaknesses that must be addressed in some way. I read so many times that we need to review the problem, understand it, and recognize the pattern. I have a private repo to store the problem I did, but perhaps it\'s not enough. The summary is essential for recognizing the pattern and structure of your learning. I guess I should do it more often.
