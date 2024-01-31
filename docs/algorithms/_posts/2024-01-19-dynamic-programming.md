---
layout: post
title:  "Dynamic Programming"
date:   2024-01-19 11:15 +1100
categories: algorithm
---

## Determine State

### Max/Min

- leetcode 1289. Minimum Falling Path Sum II: State is the smallest and second smallest cumulative sum.

```python
def find_smallest_and_second_smallest(nums):

    smallest = float('inf')
    second_smallest = float('inf')

    for i, num in enumerate(nums):
        if num < smallest:
            second_smallest = smallest
            smallest = num
        elif num < second_smallest:
            second_smallest = num

    return smallest, second_smallest

# Example usage:
numbers = [3, 8, 1, 12, 5, 7]
smallest, smallest_index, second_smallest, second_smallest_index = find_smallest_and_second_smallest(numbers)

print("Smallest:", smallest, "at index", smallest_index)
print("Second Smallest:", second_smallest, "at index", second_smallest_index)

```

## Rolling Array Optimization

**Space complexity: O(n)**

In dynamic programming, the "rolling array optimization" is a technique used to optimize the space complexity of a solution by reusing a fixed-size array or a portion of it to store only the necessary intermediate results. This is particularly useful when solving problems with a large state space, where each entry in the dynamic programming table depends on a fixed number of previous entries.

### How It Works

1. **Original Dynamic Programming Approach:**
   - In a typical dynamic programming solution, a 2D array (or more dimensions) is used to store results of subproblems.
   - The dimensions of the array are determined by the problem requirements and the recurrence relation.

2. **Rolling Array Optimization:**
   - Identify the fact that each entry in the dynamic programming table only depends on a fixed number of previous entries.
   - Allocate a smaller array (the rolling array) that is sufficient to store the necessary information for the current calculation.
   - Reuse this smaller array for subsequent calculations by updating its contents iteratively.

### Technique

Just change one of the dimensions directly to 2. For the row-based dimensional array f[i], we can change it to f[i&1] or f[i%2] (the former is recommended; on machines with different architectures, the computational efficiency is more stable).

In this case, we are only dealing with 2 arrays f[0] and f[1]. One stores the previous DP state, and the other stores the current DP state.

### Example Usage

Here's a simple example using the Fibonacci sequence in Python:

```python
def fibonacci(n):
    if n <= 1:
        return n
    
    # Using a rolling array with two elements to store previous results
    dp = [0, 1]

    for i in range(2, n + 1):
        # Update the rolling array with the new result
        dp[i & 1] = dp[(i - 1) & 1] + dp[(i - 2) & 1]

    return dp[n & 1]

# Example usage:
result = fibonacci(5)
print(result)  # Output: 5
```

## Tree DP & Linked Forward Star

Using Linked Forward Star, we can easily iterate through children of a given tree node.

```python
class Edge:
    def __init__(self):
        self.to = 0
        self.next = 0

class LinkedForwardStar:
    def __init__(self, N):
        self.edges = []
        self.head = [-1] * N # head[i]: index of "the last edge that starts from node (i)" in the edges array
        self.cnt = 0
    
    def add_edge(self, u, v):
        e = Edge()
        e.to = v
        e.next = self.head[u]
        self.edges.append(e)
        self.head[u] = self.cnt
        self.cnt += 1
    
    def print_neighbors(self, u): # a demo of how to visit neighbors
        i = head[u] # start point
        while i != -1: # end point
            print(f'u: {u}, v: {self.edges[i].to}')
            i = self.edges[i].next # next
```

Combine Tree DP with Linked Forward Star:

```python
def maxValue(N, C, parents, values, weights):
    graph = LinkedForwardStar(N)
    root = -1
    for i in range(n):
        if p[i] == -1:
            root = i
        else:
            graph.add_edge(p[i], i)
    
    f = [[0] * (C+1) for _ in range(N)]

    def dfs(u):
        # Node u's value and volume
        cw = weights[u]
        cv = values[u]
        
        # To select any node, its parent u must be selected first, which also limits the minimum volume to cv
        for i in range(cv, C + 1):
            f[u][i] += cw

        # Traverse all child nodes x of node u
        i = graph.head[u]
        while i != -1:
            e = graph.edges[i]
            x = e.to
            # Recursive processing of node x
            dfs(x)
            # Traverse the knapsack capacity from large to small
            for j in range(C, -1, -1):
                # Traverse how much knapsack capacity to allocate to node x
                for k in range(j - cv, -1, -1):
                    f[u][j] = max(f[u][j], f[u][j - k] + f[x][k])
            i = e.next

    dfs(root)
    return f[root][C]
```