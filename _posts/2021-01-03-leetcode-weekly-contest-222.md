---
layout: post
title: "Leetcode weekly contest 222 reviews"
date: 2021-01-03
tags: "C++ algorithm leetcode competitive-programming"
---

# Some thoughts on contests

I have been taken 23 contests in Leetcode and achieve a rating of `1964`, which is around the top `3%` of all users. Though that is a modest achievement, I feel happy with it. That's something I would never think I can do just half a year ago. Besides, practicing in contests help me a lot during the interview process. I passed almost all coding assessments (except for Naver, as they give me an OA in Korean). In my last interviews with Bytedance and SAP, I ACE all the coding sections. Just one year ago, writing a heap from scratch sounds impossible to me, but in my last interview with Bytedance, I did it in less than 20 minutes. That's awesome, at least for me.

I didn't take part in the weekly contest 222. However, I did a virtual contest and found that it was very fun. I did solve 3 questions. The last question is just a tweak of the Longest Commons Subsequence, and if you know the trick to convert Longest Common Subsequence, I would have been aced it. Perhaps, Leetcode is increasing the difficulty of the problem, and it's a good thing. I'm not yet a professional competitive programmer, but I hope I can be one day. 

# Question 1: Maximum Units on a Truck ([leetcode](https://leetcode.com/problems/maximum-units-on-a-truck/))

```
You are assigned to put some amount of boxes onto one truck. You are given a 2D array boxTypes, where boxTypes[i] = [numberOfBoxesi, numberOfUnitsPerBoxi]:

    - numberOfBoxes[i] is the number of boxes of type i.
    - numberOfUnitsPerBox[i] is the number of units in each box of type i.

You are also given an integer truckSize, which is the maximum number of boxes that can be put on the truck. You can choose any boxes to put on the truck as long as the number of boxes does not exceed truckSize.

Return the maximum total number of units that can be put on the truck.
```

Quite a tricky question. The description is long and hard to understand. If you don't read the question carefully, you will struggle to come with the proper idea. Surprisingly, many newbies (including myself) will fall into the trap. So, the first thing to learn is to calm down and read the question carefully. Make sure you understand every aspect of the problem.

Here, we have `n` type of boxes. For each type `i`, we have `numberOfBoxes[i]` boxes, each boxes have volume `numberOfUnitsPerBox[i]` units. We can select `truckSize` boxes, and the task is to maximize the volume of selected boxes. Naturally, we will come with the idea of greedy: select the boxes with bigger volume first. The algorithm is:

- Sort the `boxTypes` array in decreasing order of volume
- Iterate the array:
    - At each index, if we can select all boxes, select all boxes and go to the next index.
    - Otherwise, select as much as box as we can and return

``` cpp
int maximuUnits(vector<vector<int>>& boxTypes, int truckSize) {
  sort(boxTypes.begin(),boxTypes.end(),[](vector<int>& a, vector<int>& b) {
    return a[1] > b[1];
  });
  int ret = 0;
  for (auto &v : boxTypes) {
    int nums = v[0], units = v[1];
    int max_box = min(nums,truckSize);
    ret += max_box * units;
    truckSize -= max_box;
    if (truckSize == 0) break;
  }
  return ret;
}
```

# Question 2: Count Good Meals ([leetcode](https://leetcode.com/problems/count-good-meals/))

```
A good meal is a meal that contains exactly two different food items with a sum of deliciousness equal to a power of two.

You can pick any two different foods to make a good meal.

Given an array of integers deliciousness where deliciousness[i] is the deliciousness of the i​​​​​​th​​​​​​​​ item of food, return the number of different good meals you can make from this list modulo 10^9 + 7.

Note that items with different indices are considered different even if they have the same deliciousness value.

Constraints:

1 <= deliciousness.length <= 10^5
0 <= deliciousness[i] <= 2^20
```

The problem statement is clear. It's also trivial to come with the brute-force solution quickly: Check every pair `i < j` if `deliciousness[i] + deliciousness[j]` is power of `1`. This will give us an `O(N^2)` solution and look at the constrain of `N`, we will get TLE. We need a better approach.

The first thing that came to my mind is some kind of bit-manipulation. Then I thoughts this is only the second problem, so there must be an easier way. So I look at the constrain again and found that all number in the array `deliciousness` is no bigger than `2^20`, so the sum of any two elements will not excess `2^21`. So, we can brute force for all values from `2^0` to `2^21`. Because we can do two sum in `O(N)`, this will give us an `O(20*N)` solution.

I quickly code the solution below:

```cpp
int mod = 1e9+7;
int twoSum(vector<int>& nums, int target) {
  unordered_map<int,int> seen;
  int ret = 0;
  for (auto &v : nums) {
    auto it = seen.find(target-v);
    if (it != seen.end()) ret += it->second;
    seen[v]++;
  }
  return ret%mod;
}
int countPairs(vector<int>& nums) {
  int target = 1;
  int ret = 0;
  for (int i = 0; i <= 21; ++i) {
    ret += twoSum(nums,target);
    ret %= mod;
    target = target << 1;
  }
  return ret;
}
```

This solution passes all test cases but gives TLE. I thought once again and convinced myself that my approach is correct, but I should optimize it. I did two optimizations:

- First of all, we don't need to check all sum from `2^0` to `2^21`, we can find the maximum of the array check from `2^0` to the double of that value.
- Secondly, I shouldn't create the function `twoSum` for each target sum. I can do `twoSum` one time, but with multiple targets.

The final solution is as bellow, and it was accepted.

```cpp
int mod = 1e9+7;
int countPairs(vector<int>& nums) {
  unordered_map<int,int> seen;
  int max_log = log2(*max_element(nums.begin(),nums.end())) + 1;
  int ret = 0;
  for (auto &v : nums) {
    int target = 1;
    for (int i = 0; i <= max_log; ++i) {
      if (target >= v) {
        auto it = seen.find(target-v);
        if (it != seen.end()) ret += it->second;   
      }
      target = target << 1;
    }
    ret %= mod;
    seen[v]++;
  }
  return ret;
}
```

# Question 3: Ways to Split Array Into Three Subarrays ([leetcode](https://leetcode.com/problems/ways-to-split-array-into-three-subarrays/))

```
A split of an integer array is good if:

The array is split into three non-empty contiguous subarrays - named left, mid, right respectively from left to right.

The sum of the elements in left is less than or equal to the sum of the elements in mid, and the sum of the elements in mid is less than or equal to the sum of the elements in right.

Given nums, an array of non-negative integers, return the number of good ways to split nums. As the number may be too large, return it modulo 10^9 + 7.

3 <= nums.length <= 10^5
0 <= nums[i] <= 10^4
```

Again, the question is clear. The brute-force is also easy. We need three sub-array or two indices. Try every index `i < j` with `i` is the end of `left` and `j` is the end of `mid`. Using prefix sum, we can calculate any sub-array sum in `O(1)`. So the brute-force will give us an `O(N^2)` solution. Again, within the constrain of `N`, brute-force will make it TLE.

Even brute-force give us TLE, it provide a very good start. Assume we have the prefix sum array `p` with `p[i] = sum(arr[0],...,arr[i])` Let say, assume we have `left` end at `i`, we will need to find an index `j` from `i+1` to `n-2` so that:

- Condition A: `p[j]-p[i] >= p[i]`, or `p[j] >= 2*p[i]`: `p[i]` is sum of `left`, and `p[j]-p[i]` is sum of `mid`
- Condition B: `p[n-1]-p[j] >= p[j]-p[i]` of `2*p[j] <= p[i]+p[n-1]`: `p[n-1]-p[j]` is sum of `mid`, and `p[n-1]-p[j]` is sum of `right`

It turn out that we can find the ranges of `j` with binary seach from the above conditions:

- `it0 = lower_bound(2*p[i])` will give us the first position of `j` that sastifies the condition A.
- `it1 = upper_bound((p[i]+p[n-1])/2)` will give us the first position of `j` that **does not** sastify the condition B.

Therefore, any position in `[it0,it1)` will give us a valid `j`. There is only one edge case: if `it1` is the end of the array, we need to decrease `it1` by one, because we cannot have an empty subarray.

```cpp
int mod = 1e9+7;
int waysToSplit(vector<int>& nums) {
  const int n = nums.size();
  vector<int> p(n,0);
  partial_sum(nums.begin(),nums.end(),p.begin());
  int ret = 0;
  for (int i = 0; i < n; ++i) {
    auto it0 = lower_bound(p.begin()+i+1, p.end(), 2*psum[i]);
    int x = (p[i] + p[n-1]) / 2;
    auto it1 = upper_bound(p.begin()+i+1, p.end(), x);
    if (it1 == p.end()) it1--;
    if (it0 < it1) ret += it1 - it0;
    ret %= mod;
  }
  return ret;
}
```

# Question 4: Minimum Operations to Make a Subsequence ([leetcode](https://leetcode.com/problems/minimum-operations-to-make-a-subsequence/))

```
You are given an array target that consists of distinct integers and another integer array arr that can have duplicates.

In one operation, you can insert any integer at any position in arr. For example, if arr = [1,4,1,2], you can add 3 in the middle and make it [1,4,3,1,2]. Note that you can insert the integer at the very beginning or end of the array.

Return the minimum number of operations needed to make target a subsequence of arr.

A subsequence of an array is a new array generated from the original array by deleting some elements (possibly none) without changing the remaining elements' relative order. For example, [2,7,4] is a subsequence of [4,2,3,7,2,1,4] (the underlined elements), while [2,4,2] is not.

1 <= target.length, arr.length <= 10^5
1 <= target[i], arr[i] <= 10^9
target contains no duplicates.
```

If you are doing competitive programming for a while, you can quickly recognize the pattern: take the length of the longest common subsequence between `arr` and `target`, the size of `target` minus the size of the LCS will give us the answer. The problem is, the typical LCS is `O(N^2)`, and again, given the constraints, it's TLE. The idea of this problem is we convert the LCS problem to the LIS - longest increasing subsequence - so that we can solve it in `O(NlogN)`. Before getting to the solution, let's review both LCS and LIS problems.

## Longest Common Subsequence

In the LCS problem, we are given two arrays of integer `A` and `B`, and our task is to find the longest common subsequence of two arrays. A subsequence of the array is a collection of elements in the array in which the order of elements in the original array is preserved. For examples, `arr = [1,3,4,5]` will have subsequences `[1,3,5]`, `[1,4]`... and so on.

LCS has been study thoroughly in CS. The optimal solution is using dynamic programming. Let's `dp[i][j]` is the LCS if array `A` ends at `i` and array `B` ends at `j`. Then we will have:

- if `A[i] == B[j]`: we can include `A[i]` and `B[j]` in the LCS at `(i-1,j-1)` to create the LCS at `(i,j)`, i.e. `dp[i][j] = 1 + dp[i-1][j-1]`
- if `A[i] == B[j]`: LCS at `(i,j)` is either the LCS at `(i-1,j)` or `(i,j-1)`

```cpp
int longestCommonSubSequence(vector<int>& A, vector<int>& B) {
  const int m = A.size(), n = B.size();
  vector<vector<int>> dp(m+1,vector<int>(n+1,0));
  for (int i = 1; i < m; ++i) {
    for (int j = 1; j < n; ++j) {
      if (A[i-1]==B[j-1]) dp[i][j] = 1+dp[i-1][j-1];
      else dp[i][j] = max(dp[i-1][j],dp[i][j-1]);
    }
  }
  return dp[m][n];
}
```

## Longest Increasing Subsequence

In the LIS problem, we have an array of integers `arr` and our task is to find the longest increasing (assume operator `>`) subsequence. The problem can also be solved with dynamic programming. Let `dp[i]` is the length of the LIS that **ends** at `i`. Then for each index `j` from `0` to `i-1`, if `arr[i] >= arr[j]` then `dp[j]+1` can be the candidate for `dp[i]`. This will give us an `O(N^2)` solution as follow:

```cpp
int longestIncreasingSubSequence(vector<int>& A) {
  const int n = A.size();
  vector<int> dp(n,1);
  int ret = 0;
  for (int i = 0; i < n; ++i) {
    for (int j = 0; j < i; ++j) {
     if (arr[i] > arr[j]) 
        dp[i] = max(dp[i],1+dp[j]);
    }
    ret = max(ret,dp[i]);
  }
  return ret; 
}
```

The above code is easy. However, it's not yet the optimal solution. The following solution is from Leetcode. That's smart, and I could not come with it myself, so I will write the code first, and explain later.

```cpp
int longestIncreasingSubSequence(vector<int>& arr) {
  vector<int> res;
  for (auto &v : arr) {
    auto it = lower_bound(res.begin(),res.end(),v);
    if (it == arr.end()) res.push_back(v);
    else *it = v;
  }
  return res.size();
}
```

The above algorithm works greedily. Here, we try to construct the LIS with the array `res`. For each value in the array, there are two choices:

- If that is the largest value we found, then include it in the sequence, because it obviously makes a longer sequence.
- If that is not, then it can be a good start or continuation of a longer sequence. We don't need to start again but can update `res` in-place. 

In this way, the elements in `res` are not necessarily the LIS, but the length of res will give us the length of LIS. The intuition is quite close to a `monotonic stack`.

## LCS and LIS

So, can we convert LCS to LIS? Well, in general, no. But in this problem, yes. Recall that the array `target` consists of only distinct integers, therefore, an element is identical to its index. So, we can replace the elements of `arr` by their index in `target` (do nothing if an element is not in target) and call the new array `new_arr`. The LCS of `target` and `arr` is the LIS of `new_arr`. Note that the approach works only because of all integer in `target` is unique.

```cpp
int minOperations(vector<int>& target, vector<int>& arr) {
  const int n = target.size();
  unordered_map<int,int> pos;
  for (int i = 0; i < n; ++i) {
    pos[target[i]] = i;
  }
  vector<int> new_arr;
  for (auto &v : arr) {
    auto it = pos.find(v);
    if (it != pos.end()) new_arr.push_back(pos[v]);
  }
  vector<int> res;
  for (auto &v : new_arr) {
    auto it = lower_bound(res.begin(),res.end(),v);
    if (it == res.end()) res.push_back(v);
    else *it = v;
  }
  return n - res.size();
}
```

# Afterthoughts

Contests are fun, but also give me a headache. I need to work even harder to get better in the contest as well as the problem-solving game. I hope in 2021, I can claim the "Guardian" badge on Leetcode (which is only Master in Codeforce I think). There is a long way to go, and I hope I will not give up.