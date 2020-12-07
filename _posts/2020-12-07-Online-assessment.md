---
layout: post
title: "Online Assessment Review"
date: 2020-11-29
tags: "coding-interview"
---

Recently, I have two online assessments, both from great tech companies. To be more accurate, this is an online assessment and a take home task. I did quite well for the take-home task, but the online assessment is not that good (at least, not as I expected). I want to review both of them here, hopefully, I will not make the same mistakes for the next interview.

# Online Assessment

Very standard Hackerrank online assessment. 120 mins with 4 questions. I finish 2 and was almost done with the 3rd question.

## Question 1: Counting rabbit pairs

```
1 pair of rabbits will give birth to another pair of rabbits (one male, one female) every day. The newborn pair will start giving birth on the second day after they were born. Formally, pair born on day k will start giving birth in the day `k+2` Assume you have a pair of rabbits, count the number of pairs you will have after N days.

Example 1: n = 1
Answer: 1

Example 2: n = 2
Answer: 2

Example 3: n = 3
Answer: 3

Example 4: n = 4
Answer: 5

```

Sound pretty horrible, but if you are familiar with competitive programming, you will smell the dynamic programming. And if you have some math background, you can even realize after a few minutes: this is just the Fibonacci sequence! Nevertheless, it takes me more than 20 mins to realize it\'s a Fibonacci sequence. I don\'t take the math approach, but I draw the sequence like below. Assume the first pair you have is `a1`, it will give birth to `a11, a12...` and so on. The sequence of giving birth will look like this:

```
1         2         3         4         5
a1        a11       a12       a13       a14
                              a111      a112
                                        a121
```

Consider when `n == 5`, we will have `8` pairs. In this `8` pair, `5` pair come from the day `4` and `3` pairs are newborn pair on that day. Why do we have `3` pair on that day? Because when `n == 3`, we have `3` pairs, and all of them will give birth on day `5`. That is, if we call our answer `f[n]`, then `f[5] = f[4] +f[3]`. With similar logic, you can see the total pair on the day `n` will be the sum of the total pair on the previous day `f[n-1]`, plus the number of newly born pair on the day, which is `f[n-2]`. So, we have the Fibonacci sequence here: `f[n] = f[n-1] + f[n-2]`.

## Question 2: Maximum product sum to a number

```
Given an integer N. N can be written in sum of at least two number as `N = a0 + a1 + ... + ak`. Find the maximum product of `a0,a1,...,ak`.

Example: n = 2
Answer: 1 (1+1)

Example: n = 3
Answer: 2 (1+2)

Example: n = 6
Answer: 9 (3+3)
```

It\'s first look like a math problem, but turn out to be dynamic programming. If we call our answer `f[N]`, then it\'s very likely `f[n] = max(f[a]*f[b])`. So I quickly implement it. But then I got some WAs. There is something I miss in the logic. Assume if `N = 4`. If we apply the above logic, it will give us `2` as `f[4] = max(f[1]*f[3],f[2]*f[2]) = max(1,2) = 2`. But it should be `4`, because `4 = 2+2` and `2*2 = 4`. Then I realize the above logic will only work when `f[n] >= n`! That is, `n >= 4`. When `n < 4`, we should initilize `f[n] = n`. What about the return value when `n < 4`? Well, just hardcode it.

```cpp
int maxProductSumToNumber(int n) {
  if (n == 2) return 1;
  if (n == 3) return 2;
  vector<int> dp(n+1,0);
  dp[1] = 1, dp[2] = 2, dp[3] = 3;
  for (int i = 4; i <= n; ++i) {
    for (int j = 1; j < i; ++j) {
      dp[i] = max(dp[j]*dp[i-j]);
    }
  }
  return dp[n];
}
```

## Question 3: Probability all tests is different

```
We recently upgrade an interface and we want to test it. After testing, we conclude that the interface is broken and both old and new interfaces will randomly return `N` different values. We decide to run the new interface `M` times, and the new interface `N` times, and compare them `M*M` times. Find the probability that all tests are different.

Example: N = 2, M = 1
Return: 0.5
Explanation: Let call the output of interface `A` and `B`. The possible result is `(A,B),(B,A),(A,A),(B,B)`. Only `(A,B),(B,A)` is sastisfied, so return 2/4 
```

The problem description is a little different. But the main idea is to calculate two values:

- Number of possible outcome `x`
- Number of outcome that all tests are different `y`

The problem description is quite ambiguous tbh. I cannot interpret it for a while. I think we can reduce the problem to a more simple one as:

```
Given an alphabet with `n` characters and two strings A and B with length `M`. Calculate:

1. Number of ways we can construct the two strings -> x
2. Number string pairs such as: if a character is in `A`, it will not be in `B` and vice versa -> y
```

Then the thing is much easier to do. `x` is `n^m*n^m = n^(2m)`, because we have a total of `2m` characters and each character has `n` choices. `y` is more complicated. I tried different approaches but still cannot deliver it in time. And TBH, I still cannot solve it by now.


# Take Home Task - External Sort

```
Given a 200GB text file, one sentence in a line, and a Linux machine with 8GB of RAM. Sort the file in lexicographically increasing order and write to an output file.
```

A basic external sort problem. Just some notes:

- Character can be non-ASCII
- File can be very large

Quickly implement it in about 10 hours. The code and online demo can be found [here](https://repl.it/@CanhLe2/CocCoc-Take-Home-Test)

## Algorithm

Assume we have `N` bytes memory and the size of the file we want to sort is `M`. We also assume we can use `N` bytes exclusively to store the content of the files.  External sort does sorting in two-step:

1. Sort chunks: divide the file into chunks of no more than `N` bytes, and sort each chunk, and save to temporary files on disk
2. Merge chunks: merge the chunks 

After the first step, we will have `K = ceil(M/N)` temp files. In the next steps, we will create `K+1` equals size `P = N/(K+1)` buffers in memory. We fill the first `K` buffers with first lines from `K` temporary files, then merge the chunks to the last buffer. When the output buffer is full, write its contents to the disk. When an input buffer is empty, populate it with the next lines from corresponding files. We keep doing it until all intermediate files are processed.

## Remarks

### UTF8 string comparison

Linux file is uft-8, so we can use `std::string` to buffer it. Nevertheless, the default operator `<` of strings will not give us the lexicographical order on utf-8 string. We need rules to compares the utf-8 strings. Fortunately, `STL` has a class to do it: `std::locale`. Unfortunately, it\'s pretty slow compared to the operator `<`. But it gives us a more reasonable result. For example, given the following input:

```
Bi
Ca
Anh
GCC
Ân
DDay
Đảng
```

Sorting with default `<` will give us:

```
Anh
Bi
Ca
DDay
GCC
Ân
Đảng
```

Sorting with `std::locale("en_US.UTF-8")` will give us:

```
Anh
Ân
Bi
Ca
Đảng
DDay
GCC
```

The second sorting is indeed more reasonable. However, it\'s almost `10x`  slower. The program uses `<` for sorting by default. To use `std::locale`, add `-DENABLE_LOCALE` to compile command. 

Can we make it better? I don\'t know yet. UTF-8 and Unicode are not easy to understand in a few days. Perhaps, I think we can define our comparison with a specific language. Or, we can pre-compute transform UTF8-string to locale-aware string with `std::collage::transform` and pay the memory cost. In `C++20`, a new class of `std::u8string` is introduced to hold UTF-8 string, maybe it will provide better support for lexicographical comparison.

### Sort and merge algorithm

I don\'t think we can make better than `std::sort`. For merging, I avoid scanning the top elements of the buffers by using a `priority_queue` to hold the **top iterator** of the buffers (please check the code for details). The merging complexity is `O(NlogK)` with N is the total number and `K` is the number of chunks.

### IO optimization

I use stream IO in the standard library to read and write files. Writing is normal, but reading is asynchronous. We split our buffer into two half. While we process the first half, we read to the second half. At the end of processing, swap the first half and second half and continue until the file is completely read. The asynchronous IO is done with `std::async`.

```
--RD first chunks-- --Process chunks--        --Process chunks--
                    --RD next chunks-- -swap- --RD next chunks--

```

So, if we have `P` bytes for the reading buffer, only `P/2` bytes are processed in an iteration. This might increase the number of chunks.

By default, the program run in async mode. We can add the 4th argument to run in sync mode as follow:

```
argv[4]     sort chunks     merge chunks
0           sync            sync
1           async           sync
2           sync            async
3           async           async
```

Comparison between of sync and async:

- System: Unbuntu 16.4, Intel Xeon E-5 1.2GHz, 16GB RAM, HDD 1TB W@4k 613MB/s, R@4k 116MB/s w/o cache
- File: 1.4G
- Report with `time [command]` in second
- Memory set: 100000000 (~100MB)

```
argv[4]   sys      user     real
0         47.859   24.413   7.811  
1         45.547   25.063   6.109
2         49.168   29.966   8.531
3         32.056   27.829   6.628       
```

- Memory set: 10000000 (~10MB)

```
argv[4]   sys      user     real
0         47.207   22.672   5.702  
1         45.166   26.113   5.947
2         39.665   32.442   9.288
3         47.557   36.557   11.814       
```

> **UPDATE**: Later, I use both async read and write in `KwayMergeAsync`, which I found to provide better performance. 

## Notes

- If the input file is empty, the program creates an empty output file
- There is a low bound for the smallest memory, depends on both the size of the longest line and the size of the file. If the longest line size is `X`, file size is `M` and memory is `N`, then `N > X(M/N + 1)`
- In Linux, the maximum open files for a process is usually `1024`. If we have more than `1024` chunks, we should increase that limit.