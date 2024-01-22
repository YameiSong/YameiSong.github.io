---
layout: post
title:  "Prefix Sum and Difference Array"
date:   2024-01-22 15:10 +1100
categories: algorithm
---

## Problems Suitable for Prefix Sum Algorithm

The prefix sum algorithm is particularly suitable for solving problems that involve repeatedly computing cumulative sums over subarrays or require efficient ways to answer range queries.

> Sum(a, b) = PreSum(0, b) - PreSum(0, a)

1. **Range Sum Queries:**
   - Given an array, efficiently compute the sum of elements in a specified range [i, j].

2. **Subarray Sum Equals K:**
   - Determine the total number of contiguous subarrays whose sum equals a given target value K.
   - Longest Subarray with Sum K
   - Smallest Subarray with Sum at Least K
   - Count Subarrays with Sum K

3. **Maximum Subarray Sum:**
   - Find the maximum sum of a contiguous subarray within an array.

4. **Frequency Counting:**
   - Keep track of the frequency of elements in a range.

5.  **Maximum Subarray Product:**
    - Find the maximum product of a contiguous subarray within an array.

```python
def subarrays_with_sum(nums, k):
    prefix_sum = {0: 1}  # Initialize a dictionary to store prefix sums and their frequencies
    current_sum = 0  # Initialize the current prefix sum
    count = 0  # Initialize the count of subarrays with sum k

    for num in nums:
        current_sum += num  # Update the current prefix sum
        # Check if there is a prefix sum such that current_sum - prefix_sum == k
        count += prefix_sum.get(current_sum - k, 0)
        # Update the frequency of the current prefix sum in the dictionary
        prefix_sum[current_sum] = prefix_sum.get(current_sum, 0) + 1

    return count

# Example usage:
nums = [1, 2, 3, 4, 5]
k = 8
result = subarrays_with_sum(nums, k)
print(result)

```

## Problems Suitable for Difference Array Algorithm

The primary use case for the difference array involves frequent updates of elements in a specific range of an array.

The range update on `[l, r)` is highly efficient because it only modifies on two points: `l` and `r`.

1. **Range Updates:**
   - Perform updates on a range of elements in the array, such as incrementing or decrementing values.

2. **Dynamic Array Manipulation:**
   - Efficiently handle dynamic array manipulation, such as incrementing or decrementing values in a range.

```python
def diff_array(nums):
    n = len(nums)
    
    # Initialize the difference array with zeros
    diff = [0] * n
    
    # Build the difference array (prefix sum)
    for i in range(1, n):
        diff[i] = nums[i] - nums[i - 1]
    
    return diff

def range_update(diff, l, r, val):
    # Perform range update on the difference array
    diff[l] += val
    if r + 1 < len(diff):
        diff[r + 1] -= val

def compute_updated_array(diff):
    # Compute the updated array based on the difference array
    n = len(diff)
    nums = [diff[0]] * n
    
    for i in range(1, n):
        nums[i] = nums[i - 1] + diff[i]
    
    return nums

# Example usage:
nums = [1, 4, 7, 3, 9]
diff = diff_array(nums)
range_update(diff, 1, 3, 2)  # Update the range [1, 3] by adding 2
range_update(diff, 2, 4, -1)  # Update the range [2, 4] by subtracting 1
updated_nums = compute_updated_array(diff)
print(updated_nums)
```
