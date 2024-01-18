---
layout: post
title:  "Congruence Modulo"
date:   2024-01-18 09:49 +1100
categories: algorithm
---

## Keyword

- Divisible by `k`
- 
## Concept

Congruence modulo, often denoted as `a ≡ b (mod m)`, is a mathematical concept that deals with remainders after division. In the context of algorithms and programming, it is commonly used to simplify and compare values based on their remainders when divided by a given modulus.

The notation `a ≡ b (mod m)` means that the integers `a` and `b` have the same remainder when divided by `m`.

## Problems Suitable for Congruence Modulo

1. **Continuous Subarray Sum**
   - *Problem:* Given an integer array nums and an integer `k`, determine if there exists a continuous subarray such that the sum is divisible by `k`.
   - *Solution:*
     - Utilize the concept of congruence modulo (`≡`) to check if the prefix sums have the same remainder when divided by `k`.
     - Define the sum of elements from index `a` to `b` as `Sum(a, b) = PrefixSum(0, b) - PrefixSum(0, a)`.
     - To ensure `Sum(a, b) % k == 0`, it's required that `PrefixSum(0, b) ≡ PrefixSum(0, a) (mod k)`.

2. **Counting Pairs**
   - *Problem:* Count all pairs of numbers in an array such that their sum is exactly divisible by `k`.
   - *Solution:*
     - Employ the principle of congruence modulo to determine pairs with a sum divisible by `k`.
     - Calculate the remainder of each element when divided by `k` and find the complement needed for the sum to be divisible.
     - Maintain a count of such pairs using a dictionary to store the frequency of remainders.
     - Update the count by adding the frequency of pairs with the same complement.

    ```python
    from collections import defaultdict

    def count_pairs_with_sum_divisible_by_k(nums, k):
        remainder_count = defaultdict(int)
        count = 0

        for num in nums:
            remainder = num % k
            complement = (k - remainder) % k
            count += remainder_count[complement]
            remainder_count[remainder] += 1

        return count

    # Example usage:
    nums = [9, 7, 5, 3]
    k = 6

    result = count_pairs_with_sum_divisible_by_k(nums, k)
    print(result)
    ```

