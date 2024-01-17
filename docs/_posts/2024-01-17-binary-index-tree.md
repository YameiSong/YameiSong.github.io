---
layout: post
title:  "Binary Indexed Tree (Fenwick Tree)"
date:   2024-01-17 15:59 +1100
categories: database
---

## Problems Suitable for Binary Indexed Trees

Binary Indexed Trees (BIT), also known as Fenwick Trees, are efficient data structures for handling cumulative frequency or prefix sum queries. Here are some common problems for which Binary Indexed Trees are particularly well-suited:

1. **Prefix Sum Queries:**
   - *Problem:* Find the sum of elements in a prefix of an array and update elements efficiently.
   - *Solution:* Binary Indexed Trees can maintain prefix sums, allowing for quick queries and updates.

2. **Inverse Cumulative Frequency:**
   - *Problem:* Given an array representing cumulative frequencies, find the index corresponding to a particular cumulative frequency.
   - *Solution:* BITs can be used to efficiently find the position of a specific cumulative frequency.
     - Use binary search on preSum(k)

3. **Counting Inversions:**
   - *Problem:* Count the number of inversions in an array.
       - Count inversed pairs (LCR 170)
       - Count smaller number after self (315)
   - *Solution:* Binary Indexed Trees can be employed to count inversions during the process of building prefix sums.

4. **Frequency Count in a Range:**
   - *Problem:* Count the number of occurrences of elements in a given range.
   - *Solution:* BITs can store the frequency of each element and support efficient range queries.

5. **2D Range Queries:**
   - *Problem:* Perform efficient queries on a 2D matrix, such as finding the sum in a rectangular region.
   - *Solution:* BITs can be extended to handle 2D queries by using a 2D array of BITs.

## Binary Indexed Tree Template

```python
class BIT:
    def __init__(self, n):
        self.n = n
        self.tree = [0] * n
    
    def lowbit(self, i):
        return i & -i 
    
    # nums[k] += x
    def add(self, k, x):
        # i must start from k+1. if k = 0, 0+lowbit(0)=0, will leads to infinite loop
        i = k + 1
        while i < self.n:
            self.tree[i-1] += x
            i += self.lowbit(i)
    
    # return sum(nums[0], ..., nums[k])
    def preSum(self, k):
        ans = 0
        i = k + 1
        while i > 0:
            ans += self.tree[i-1]
            i -= self.lowbit(i)
        return ans

class Solution: # leetcode 315: Count of Smaller Numbers After Self
    def discrete(self, arr):
        # 1. use set to remove duplicates
        # 2. use dict to map sorted value to index (i.e. "discretize")
        mp = {val: idx for idx, val in enumerate(sorted(set(arr)))}
        print(f'discretize {arr} to {mp}')
        return mp

    def countSmaller(self, nums: List[int]) -> List[int]:
        mp = self.discrete(nums)
        nums = [mp[v] for v in nums]
        n = len(nums)
        bit = BIT(n)

        ans = []
        # always start from edge case
        # e.g., the rightmost number has no neighbors on its right
        for x in nums[::-1]:
            ans.append(bit.preSum(x-1))
            bit.add(x, 1)
        return ans[::-1]

def stringToIntegerList(input):
    return json.loads(input)

if __name__ == '__main__':
    nums = stringToIntegerList('[5,2,6,1]')
    print(Solution().countSmaller(nums))
```

## Discretize data

When data range is extremely big, we can discretize data to a smaller range while keeping their relative order.
For example, transform `[-1000, 0, 2000]` to `[0, 1, 2]`

```python
def discrete(arr):
    # 1. use set to remove duplicates
    # 2. use dict to map sorted value to index (i.e. "discretize")
    mp = {val: idx for idx, val in enumerate(sorted(set(arr)))}
    print(f'discretize {arr} to {mp}')
    return mp
```