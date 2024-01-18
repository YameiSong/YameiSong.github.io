---
layout: post
title:  "Sliding Window"
date:   2024-01-18 09:49 +1100
categories: algorithm
---

## Problems Suitable for Sliding Window Algorithm

The sliding window algorithm is particularly useful when dealing with problems that require tracking or analyzing subarrays or windows of elements within a larger array or sequence. Its efficiency stems from the fact that it avoids redundant computations and explores the solution space in a systematic and optimized manner.

1. **Fixed Size Subarray/Window Problems:**
   - Find the sum/average/product of subarrays/windows of a fixed size in an array.
   - Examples: Maximum Sum Subarray of Fixed Size, Smallest Subarray with a Given Sum, Maximum Product Subarray, etc.

2. **Variable Size Subarray/Window Problems:**
   - Find the sum/average/product of subarrays/windows where the size can vary.
   - Examples: Longest Subarray with at Most K Distinct Elements, Subarrays with Product Less than K, etc.

3. **Two-Pointer Technique:**
   - Use two pointers to efficiently solve problems involving subarrays or windows.
   - Examples: Container With Most Water, Trapping Rain Water, Three Sum, etc.

4. **String Problems:**
   - Solve problems related to strings and substrings.
   - Examples: Minimum Window Substring, Longest Substring Without Repeating Characters, Longest Repeating Character Replacement, etc.

5. **Counting/Mapping Problems:**
   - Maintain counts or mappings of elements within a sliding window.
   - Examples: Counting Elements in Two Arrays, Number of Anagrams in a String, etc.

6. **Frequency Counting:**
   - Count the frequency of elements within a window efficiently.
   - Examples: Longest Substring with Same Letters after Replacement, Fruit into Baskets, etc.

7. **Optimization Problems:**
   - Optimize brute-force solutions by avoiding redundant calculations using the sliding window technique.
   - Examples: Minimum Size Subarray Sum, Maximum Points You Can Obtain from Cards, etc.

8. **Monotonic Queues and Deques:**
   - Maintain monotonicity in a queue or deque within a sliding window for efficient problem-solving.
   - Examples: Sliding Window Maximum, Sliding Window Minimum, etc.

9. **Dynamic Programming Optimization:**
   - Optimize dynamic programming solutions by applying the sliding window technique.
   - Examples: Maximum Subarray Sum with One Deletion, Shortest Subarray with Sum at Least K, etc.

10. **Frequency Tracking in Arrays:**
    - Efficiently track frequencies of elements in an array using sliding windows.
    - Examples: Subarrays with K Different Integers, etc.

## Hash Table in Sliding Window

1. Count elements in the sliding window by type
   - leetcode 904. Fruit Into Baskets: a window must contain no more than 2 types of elements
   - leetcode 438. Find All Anagrams in a String: a window must contain the same types of elements with the same freqency

2. Check if a substring has appeared before
   - leetcode 187. Repeated DNA Sequences

```python
from collections import defaultdict

def slidingWindow(arr: List[int] or str):
   hash_table = defaultdict(int)
   ans = 0 or []
   left = 0
   for right in range(len(arr)):
      use arr[right] to update hash_table
      if hash_table triggers some required condition:
         update ans
      shrink left to restore hash_table status
   return ans
```

## Rolling Hash

A rolling hash is a technique used in **string matching** algorithms to efficiently compute the hash value of a window or substring of a larger sequence as it shifts or "rolls" over the sequence. This approach is particularly useful for algorithms that involve **sliding windows**, such as the **Rabin-Karp** string matching algorithm.

### 1. Initialization

Calculate the hash value for the initial substring (window) of the sequence using a hash function.

### 2. Rolling the Window

As the window slides one position to the right, update the hash value for the new substring efficiently without recalculating the entire hash from scratch.

### 3. Rolling Hash Formula

Use a rolling hash formula to update the hash value. The formula typically involves subtracting the contribution of the character leaving the window and adding the contribution of the new character entering the window.

```
hash_new = (hash_old - old_char * base^(window_size-1)) * base + new_char
```

```python
class RabinKarp:
    def __init__(self, seq):
      # seq: first string sequence in the sliding window
      k = len(seq) # size of window
      self.base = 131 # use a prime as base to reduce hash collision
      self.mod = int(1e10) + 7 # avoid int overflow
      self.p = pow(self.base, k - 1, self.mod)  # factor of the leading character: base^(k-1)
      self.hash = self.init_hash(seq)

    def init_hash(self, seq):
      # For example, the hash code of "abcd...xyz" is
      # h = 'a' * base^(k-1) + 'b' * base^(k-2) + ... + 'y' * base + 'z'
      h = 0
      for ch in seq:
         h = (h * self.base + ord(ch)) % self.mod
      return h

    def update_hash(self, ch_out, ch_in):
      h = self.hash
      h = ((h - ord(ch_out) * self.p) * self.base + ord(ch_in)) % self.mod
      self.hash = h

class Solution: # leetcode 187
    def findRepeatedDnaSequences(self, s: str) -> List[str]:
        cnt = defaultdict(int)
        k = 10
        n = len(s)
        ans = []
        hash_gen = RabinKarp(s[:k])
        h = hash_gen.hash
        cnt[h] += 1
        for i in range(k, n):
            seq = s[i - k + 1 : i + 1]
            hash_gen.update_hash(s[i - k], s[i])
            h = hash_gen.hash
            cnt[h] += 1
            if cnt[h] == 2:
                ans.append(seq)
        return ans
```