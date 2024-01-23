---
layout: math
title:  "Knapsack Problem"
date:   2024-01-23 11:28 +1100
categories: algorithm
---

## 0-1 Knapsack Problem

### Problem Statement

Given a set of items, each with a weight $ w_i $ and a value $ v_i $, and a knapsack with a maximum weight capacity $ W $, determine the combination of items to include in the knapsack to maximize the total value, without exceeding the weight capacity. The decision for each item is binary: either include it (1) or exclude it (0).

### Example

Consider the following set of items:

| Item | Weight (w_i) | Value (v_i) |
| ---- | ------------ | ----------- |
| A    | 2            | 3           |
| B    | 3            | 4           |
| C    | 4            | 5           |
| D    | 5            | 8           |

Maximum weight capacity $ W $: 5

The goal is to find the optimal combination of items to maximize the total value within the weight limit.

### Solution Approach

The problem can be solved using dynamic programming. Construct a table where each entry $ dp[i][j] $ represents the maximum value that can be obtained with the first $ i $ items and a knapsack capacity of $ j $. The state transition equation is:

$$
dp[i][j] = \max(dp[i-1][j], dp[i-1][j - w_i] + v_i)
$$

The final result is in $ dp[4][5] $, representing the maximum value achievable with the given items and knapsack capacity.

```python
def knapsack_01(values, weights, capacity):
    n = len(values)
    dp = [[0 for _ in range(capacity + 1)] for _ in range(n + 1)]

    for i in range(1, n + 1):
        for j in range(1, capacity + 1):
            # If the current item can be included
            if weights[i - 1] <= j:
                dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weights[i - 1]] + values[i - 1])
            else:
                # If the current item cannot be included due to weight constraint
                dp[i][j] = dp[i - 1][j]

    return dp[n][capacity]

# Example
values = [3, 4, 5, 8]
weights = [2, 3, 4, 5]
capacity = 5

max_value = knapsack_01(values, weights, capacity)
print(f"Maximum value in the knapsack: {max_value}")
```

### 1D Optimization

To remove `i` from `dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weights[i - 1]] + values[i - 1])`, we need to loop through j from big to small because `dp[i][j]` depends on the value from the previous row `dp[i-i][...]`.

## Complete Knapsack Problem

The complete knapsack problem, also known as the unbounded knapsack problem, is a variation of the classic knapsack problem. In this variant, you can take any number of instances of each item, rather than being limited to either taking an item or leaving it behind (as in the 0-1 knapsack problem).

### Problem Definition

Given a set of items, each with a weight $ w_i $, a value $ v_i $, and a knapsack with a maximum weight capacity $ W $, the goal is to determine the maximum value that can be obtained by selecting any number of instances of each item while not exceeding the weight capacity of the knapsack.

### Algorithmic Approach

Dynamic programming can also be used to solve the complete knapsack problem. The approach involves building a table where each entry (i, j) represents the maximum value that can be obtained with the first $ i $ items and a knapsack capacity of $ j $.

The recurrence relation for the complete knapsack problem is:

$$
dp[i][j] = \max(dp[i-1][j], dp[i][j-w_i] + v_i)
$$

Here, $ dp[i-1][j] $ represents the maximum value obtained by excluding the i-th item, and $ dp[i][j-w_i] + v_i $ represents the value obtained by including the i-th item.

### Example

Consider the following example:

```plaintext
Items:
Item 1: Weight = 2, Value = 10
Item 2: Weight = 3, Value = 15
Item 3: Weight = 4, Value = 20

Knapsack Capacity (W): 7
```

```python
def complete_knapsack(weights, values, W):
    n = len(weights)
    dp = [[0] * (W + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for j in range(1, W + 1):
            # If the current item can be included
            if weights[i - 1] <= j:
                dp[i][j] = max(dp[i-1][j], dp[i][j - weights[i-1]] + values[i-1])
            else:
                # If the current item cannot be included due to weight constraint
                dp[i][j] = dp[i - 1][j]

    return dp[n][W]

# Example usage:
weights = [2, 3, 4]
values = [10, 15, 20]
W = 7
result = complete_knapsack(weights, values, W)
print("Maximum value:", result)
```

### 1D Optimization

To remove `i` from `dp[i][j] = max(dp[i-1][j], dp[i][j - weights[i-1]] + values[i-1])`, we need to loop through j from small to big (opposite to 0-1 knapsack problem) because `dp[i][j]` depends on the value from the current row `dp[i][j - weights[i-1]]`.