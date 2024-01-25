---
layout: post
title:  "Bit Manipulation"
date:   2024-01-22 16:36 +1100
categories: algorithm
---

## Bit Manipulation

```python
k = 3
# Checking if a Bit is Set
is_set = (a & (1 << k)) != 0
print(f"Bit k is set: {is_set}")

# Setting a Bit
set_bit = a | (1 << k)
print("Setting Bit k:", set_bit)

# Clearing a Bit
clear_bit = a & ~(1 << k)
print("Clearing Bit k:", clear_bit)

# Toggling a Bit
toggle_bit = a ^ (1 << k)
print("Toggling Bit k:", toggle_bit)

# Counting Set Bits (Population Count)
count_set_bits_a = bin(a).count('1') # or a.bit_count()

# Brian Kernighan's Algorithm (Counting Set Bits)
bits = 0
while x > 0:
    x &= x - 1 # clears the rightmost set bit (1) in x
    bits += 1
# The loop continues until all set bits in the binary representation of x have been cleared.
```