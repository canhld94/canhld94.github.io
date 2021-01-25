---
layout: post
title: "Leetcode weekly contest 225 reviews"
date: 2021-01-25
tags: "C++ algorithm leetcode competitive-programming"
---

Again, a review of a Leetcode contest. The problem is getting more interesting and also harder. I will not review all questions but only focus on Question II. 

```
You are given two strings a and b that consist of lowercase letters. In one operation, you can change any character in a or b to any lowercase letter.

Your goal is to satisfy one of the following three conditions:

1. Every letter in a is strictly less than every letter in b in the alphabet.
2. Every letter in b is strictly less than every letter in a in the alphabet.
3. Both a and b consist of only one distinct letter.

Return the minimum number of operations needed to achieve your goal.

 

Example 1:

Input: a = "aba", b = "caa"
Output: 2
Explanation: Consider the best way to make each condition true:
1) Change b to "ccc" in 2 operations, then every letter in a is less than every letter in b.
2) Change a to "bbb" and b to "aaa" in 3 operations, then every letter in b is less than every letter in a.
3) Change a to "aaa" and b to "aaa" in 2 operations, then a and b consist of one distinct letter.
The best way was done in 2 operations (either condition 1 or condition 3).
Example 2:

Input: a = "dabadd", b = "cda"
Output: 3
Explanation: The best way is to make condition 1 true by changing b to "eee".
 

Constraints:

1 <= a.length, b.length <= 10^5
a and b consist only of lowercase letters.
```

Some initial thoughts after reading the problem description:

- There are 3 conditions, but it is 2 because the first two conditions are the same, we only need to swap the two string
- The 3rd condition is easy, we only need to find the character with maximum frequency and count the other characters (do on both string)

Let now focus on the first condition: how to make every character in `a` strictly smaller than every character in `b`. I came up with a wrong greedy solution as follow:

- Find the maximum character in `a`, call it `ch`
- For every character `c` in `b`, if `c <= ch` then increase the answer by one

Of course, it's the wrong solution. We can change the character in both strings, so we can either make `c` bigger or `ch` smaller. If we make `ch` smaller, we change the string `a` and we cannot continue the logic. So, the naive greedy approach will not work. 

After realizing the greed was wrong, I made up my mind and try another approach. Since we need to make the smallest character in `b` greater than the largest character in `c`, we can use two heaps. The first min-heap for `b` and the second max-heap for `a`. Then we can try to update the top of the two heaps. However, this approach leads to the same question as to the greedy approach: which heap to update. It's a really hard (and probably unanswerable) question. I was frustrated and lost direction. 

Then I stop for one minute. Then I look at the question again, and ask myself the _game channging_ question: if every character in `a` is less than any character in `b`, what does it look like if we draw the histogram of characters in `a` (call `fa`) and `b` (call `fb`). There must be a point in the character range (call `ch`) where all columns in `fa` are on the left of `ch` and all columns in `fb` are on the right of `ch`. If we know `ch`, calculating the cost to move all columns of `fa` to the left of `ch` and all columns of `fb` to the right of `ch` is easy. That was the trick. We only have 26 characters, so we can brute-force for `ch`!

```cpp
class Solution {
public:
  int inf = 1e5+7;
  int make_smaller(int *fa, int *fb) {
    // make fa  < fb
    int ret = inf;
    if (fa[25] > 0) return ret;
    for (auto c = 0; c < 26; ++c) {
      int cnt = 0;
      for (int i = c+1; i < 26; ++i) {
        cnt += fa[i];
      }
      for (int i = 0; i <= c;++i) {
        cnt += fb[i];
      }
      ret = min(ret,cnt);
    }
    return ret;
  }
  int minCharacters(string a, string b) {
    int ret = inf;
    int fa[26] = {}, fb[26] = {};
    for (auto &c : a) {
      fa[c-'a']++;
    }
    for (auto &c : b) {
      fb[c-'a']++;
    }
    int ma = a.size() - *max_element(begin(fa),end(fa));
    int mb = b.size() - *max_element(begin(fb),end(fb));
    ret = min(ret,ma+mb); // condition 3
    ret = min(ret,make_smaller(fa,fb)); // condition 1
    ret = min(ret,make_smaller(fb,fa)); // condition 2
    return ret;
  }
};
```

A few months ago, Q2 was usually easy-medium, but now it's medium-hard. The acceptance rate for the question during the contest is only about `15%`. This is not a hard question IMO, but it's a really good question. 