---
layout: math
title:  "String Match Algorithm"
date:   2024-02-07 10:33 +1100
categories: algorithm
---

## KMP

```python
def computeLPS(pattern):
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
    lps = computeLPS(pattern)
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