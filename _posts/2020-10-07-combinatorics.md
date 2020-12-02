---
layout: post
title: "Some remarks on combinatorics problems"
date: 2020-10-07
tags: "algorithm combinatorics dp"
---

I used to be good in Math (third prize at Vietnam Math Olympiad in 2012). Nevertheless, recently I stuck at a combinatorics problem for two hours, and it turned out the be just a tweak of [Stars and Bars](https://en.wikipedia.org/wiki/Stars_and_bars_(combinatorics)#Theorem_two) problem - which 9 years ago I can solve in less than five minutes. I also feel that I\'m quite weak when dealing with combinatorics programming questions, so I thought it is good to write some summary about it.

# Back to basic

The first thing when we learn combinatorics: given `n` balls, how to select `k` balls from the collection. Everyone would say: easy, $C^k_n$ (we will write `C(n,k)` from for convenience). We know $C(n,k) = \frac{n!}{k!(n-k)!}$. In the world of normal people, `n!` is just an integer; but in the world of (C++) programmers, most of the time `n!` is not an integer. So we cannot directly calculate `C(n,k)` with the given formula. We should have another way.

Assume that we don\'t know the formula, can we solve it by some programming technique? It turns out that yes, we can solve it with dynamic programming. The state will be `(n,k)`, and the transaction is simple. Assume we have `n` balls in front of us. Let me consider the first ball. We only have two choices:

- Select the ball and then select `k-1` balls from the remaining `n-1` balls
- Do not select the balls and select `k` balls from the remaining `n-1` balls

It\'s easy to see the solutions for two cases are mutually exclusive. Therefore, we can combine them to get the final answer:

<div align="center">
$C(n,k) = C(n-1,k-1) + C(n-1,k)$
</div>

The `C(n,k)` can be calculated in `O(nk)` time and `O(nk)` space as follow:

```cpp
const int mod = 1e9+7;
int comb(int n , int k) {
  int dp[n+1][k+1];
  memset(dp,0,sizeof(dp));
  for (int i = 0; i <= n; ++i) {
    for (int j = 0; j <= k; ++j) {
      if (j == 0) dp[i][j] = 1;
      else if (i < j) dp[i][j] = 0;
      else {
        dp[i][j] = dp[i-1][j-1] + dp[i-1][j];
      }
      dp[i][j] %= mod;
    }
  }
  return dp[n][k];
}
```
The `dp` table is the [Pascal triangle](https://en.wikipedia.org/wiki/Pascal%27s_triangle#:~:text=In%20mathematics%2C%20Pascal\'s%20triangle%20is,China%2C%20Germany%2C%20and%20Italy.). We can even use row compression to reduce space complexity to `O(n)`.

# The Stars and Bars problem

Consider the following math problem: Count the number of solution $(x_1,x_2,...,x_k)$ that:

<div align="center">
$x_1+x_2+...+x_k = n,\ x_i > 0$
</div>

We can turn the problem into a combinatorics problem by imagining we have `n` stars in a row, and we want to put `k-1` bars in the spaces between the stars. The number of ways to place the `k-1` bar is the number of solutions for the original problems. Because $x_i>0$, we cannot select the space before the first ball and space after the last ball; as well as we cannot reuse a space. We have total `n-1` spaces between stars, therefore, the number of ways to place `k-1` bars is `C(n-1,k-1)`!

Let consider another similar problem. Count the number of solution $(x_1,x_2,...,x_k)$ that:

<div align="center">
$x_1+x_2+...+x_k = n,\ x_i \ge 0$
</div>

We can first try it with a combinatorics approach. This time, we have `n` stars and `k-1` bars. But since $x_i$ can be zero, we will have `n+1` space, and we can even reuse the spaces. The solution is not simple as when $x_i > 0$. Of course, we can do some tricks to solve the combinatorics problem. However, there is a easier way to do it. Let $y_i = x_i + 1$, then $y_i >0$. Replace $x_i$ with $y_i-1$, the problem we want to solve can be rewrite:

<div align="center">
<p>$y_1-1+y_2-1+...+y_k-1 = n,\ y_i > 0$</p>
<p>or $y_1+y_2+...+y_k = n+k,\ y_i > 0$</p>
</div>

This turn out to be the first problem, and the solution is `C(n+k-1,k-1)` or `C(n+k-1,n)`.

# The line segments problem

This is the Leetcode 1621. Number of Sets of K Non-Overlapping Line Segments ([1621](https://leetcode.com/problems/number-of-sets-of-k-non-overlapping-line-segments/))

```
Given n points on a 1-D plane, where the ith point (from 0 to n-1) is at x = i, find the number of ways we can draw exactly k non-overlapping line segments such that each segment covers two or more points. The endpoints of each segment must have integral coordinates. The k line segments do not have to cover all n points, and they are allowed to share endpoints.

Return the number of ways we can draw k non-overlapping line segments. Since this number can be huge, return it modulo 10^9 + 7.

Example 1:

Input: n = 4, k = 2
Output: 5
Explanation: 
The 5 different ways {(0,2),(2,3)}, {(0,1),(1,3)}, {(0,1),(2,3)}, {(1,2),(2,3)}, {(0,1),(1,2)}.
Example 2:

Input: n = 3, k = 1
Output: 3
Explanation: The 3 ways are {(0,1)}, {(0,2)}, {(1,2)}.
Example 3:

Input: n = 30, k = 7
Output: 796297179
Explanation: The total number of possible ways to draw 7 line segments is 3796297200. Taking this number modulo 109 + 7 gives us 796297179.
Example 4:

Input: n = 5, k = 3
Output: 7
Example 5:

Input: n = 3, k = 2
Output: 1

Constraints:

2 <= n <= 1000
1 <= k <= n-1
```

I come up with the dp solution quickly. Consider the `ith > 0` points, we can place one line that ends here, and then we have `n-1` points left to place `k-1` lines. Note that we have `i` ways to place a line that ends at `ith` point. Therefore, if we call `S(n,k)` is our solution, we can have the following transition:

```cpp
for (int i = 1; i < n; ++i) {
  S(n,k) += i*S(n-i,k-1);
}
```

And it\'s correct. The only problem: it\'s `O(n^2k)`. One can imagine we have a table `nxk`. We need to fill the table, which will take us `O(nk)` time, and for each position in the table, we need a for loop of `O(n)`. Within the given constrain, it will give us TLE. I\'ve spent hours trying to write down the transition formula and try to simplify it to resolve the for loop but failed to do so. So, when I take a look at the discussion; somebody has the same idea as mine and they successfully eliminate the for loop. However, there is a far smarter way to do solve this problem. It is written in different forms, but the main idea is to reduce the problem to the Stars and Bars problem. I came up with the following approach, which is quite clear and easy to interpret to me (to you, are you serious?).

We have to to draw `k` non-overlapping lines. If we have `k` non-overlapping lines, this will divide the range `0-n` into `2*k+1` segments: 

- `k` segments are `k` line we draw, which must have length > 0; we call this $x_i$
- `k+1` segments are the distance between the lines we draw; we call this $y_i$; since lines are non-overlapped and can share endpoints, we have $y_i \ge 0$

The sum of all segments must be the range `0` to `n`, which is `n-1`. Then we have

<div align="center">
$x_1+x_2+...+x_k + y_1 + y_2 + ... + y_{k+1} = n-1,\ x_i > 0, y_i \ge 0$
</div>

Let $z_i = y_i + 1$, we can rewrite the above formula to:

<div align="center">
<p>$x_1+x_2+...+x_k + z_1 - 1 + z_2-1 + ... + z_{k+1}-1 = n-1,\ x_i > 0, z_i > 0$</p>
<p>or $x_1+x_2+...+x_k + z_1 + z_2 + ... + z_{k+1} = n+k,\ x_i > 0, z_i > 0$</p>
</div>

And this is the Stars and Bars problem with `K = 2*k+1` and `N = n+k`. So the answer is `C(n+k-1,2*k)` and we can calculate the answer in `O(nk)` time!