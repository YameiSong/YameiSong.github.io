---
layout: math
title:  "String Match Algorithm"
date:   2024-02-07 10:33 +1100
categories: algorithm
---

## KMP

### Z algorithm

In string matching, the Z algorithm is used to efficiently find all occurrences of a pattern string within a given text string. It preprocesses the pattern string to produce an array that indicates the length of the longest substring at each position that matches the prefix of the pattern string.

```python
def z_algorithm(s):
    n = len(s)
    z = [0] * n
    l, r = 0, 0  # left and right boundaries of the Z-box
    for i in range(1, n):
        if i <= r:
            # uses previously computed z[i - l] (relative to the left boundary), and ensures it does not exceed the right boundary
            z[i] = min(r - i + 1, z[i - l]) 
        while i + z[i] < n and s[z[i]] == s[i + z[i]]:
            l, r = i, i + z[i] # extend the boundary
            z[i] += 1
    return z

# Example usage:
text = "aabcaabxaaaz"
z_values = z_algorithm(text)
print("Z values:", z_values)
```

### Failure Function

The failure function is an array that associates each position in the pattern string with the length of the longest proper prefix (which is also a suffix) that matches a substring ending at that position.

```python
def failureFunction(pattern):
    """
    Computes the Longest Prefix Suffix (LPS) array for the given pattern.
    """
    m = len(pattern)
    lps = [0] * m
    length = 0  # Length of the previous longest prefix suffix

    for i in range(1, m):
        while length > 0 and pattern[i] != pattern[length]:
            length = lps[length - 1]

        if pattern[i] == pattern[length]:
            length += 1

        lps[i] = length
    return lps

def KMP(text, pattern):
    """
    Implementation of the Knuth-Morris-Pratt (KMP) algorithm.
    """
    n = len(text)
    m = len(pattern)
    lps = failureFunction(pattern)
    matches = []

    i, j = 0, 0
    while i < n:
        if pattern[j] == text[i]:
            i += 1
            j += 1

            if j == m:
                matches.append(i - j)
                j = lps[j - 1]
        else:
            if j != 0:
                j = lps[j - 1]
            else:
                i += 1

    return matches

# Example usage:
text = "ABABDABACDABABCABAB"
pattern = "ABABCABAB"
matches = KMP(text, pattern)
print("Pattern found at indices:", matches)
```
