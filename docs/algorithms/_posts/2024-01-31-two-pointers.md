---
layout: math
title:  "Two Pointers"
date:   2024-01-24 11:14 +1100
categories: algorithm
---

## Partition

```python
def two_way_partition(arr, low, high):
    pivot = arr[low]  # Choose the pivot as the first element
    i = low + 1  # Index of the smaller element

    for j in range(low + 1, high + 1):
        if arr[j] < pivot:
            # If the current element is smaller than the pivot
            # Swap arr[i] and arr[j]
            arr[i], arr[j] = arr[j], arr[i]
            i += 1

    # Swap arr[low] and arr[i - 1] (pivot)
    arr[low], arr[i - 1] = arr[i - 1], arr[low]
    return i - 1

# Example usage
arr = [4, 2, 1, 5, 3, 2, 6, 1]
pivot_index = two_way_partition(arr, 0, len(arr) - 1)
print("Partitioned array:", arr)
print("Index of the pivot:", pivot_index)
```

```python
def three_way_partition(arr, pivot):
    left, mid, right = 0, 0, len(arr) - 1

    while mid <= right:
        if arr[mid] < pivot:
            arr[left], arr[mid] = arr[mid], arr[left]
            left += 1
            mid += 1
        elif arr[mid] == pivot:
            mid += 1
        else:
            arr[mid], arr[right] = arr[right], arr[mid]
            right -= 1

    return arr

# Example usage
arr = [4, 2, 1, 5, 3, 2, 6, 1]
pivot = 3
print("Original array:", arr)
arr = three_way_partition(arr, pivot)
print("Partitioned array:", arr)
```

## Palindrome

Count all palindromes, or find longest palindrome.

### Dynamic Programming

State transition equation: 

`dp[left][right] = (s[right] == s[left]) && dp[left + 1][right - 1]`

Time complexity: $O(n^2)$

```cpp
int countSubstrings(string s) {
    int ans = 0;
    int n = s.length();
    vector<vector<bool>> dp(n, vector<bool>(n, false));
    for (int right = 0; right < n; right++) {
        for (int left = right; left >= 0; left--) {
            if ((s[right] == s[left]) &&
                (right - left < 2 || dp[left + 1][right - 1])) {
                dp[left][right] = true;
                ans++;
            }
        }
    }
    return ans;
}
```

### Manacher's Algorithm

**Preprocessing:**

Manacher's Algorithm works by transforming the input string into another string, which makes finding palindromes easier. This transformation involves inserting special characters between every character in the string and at the beginning and end. This is to handle both even and odd-length palindromes.

**Palindrome Array (P):**

Manacher's Algorithm uses a palindrome array (often called P), where P[i] represents the radius of the longest palindrome centered at index i. This means if P[i] = k, then the palindrome centered at index i extends k characters to the left and k characters to the right (excluding the center).

**Calculation of Palindrome Array (P):**

Manacher's Algorithm iterates through the transformed string and calculates the palindrome array P. It uses the properties of already computed palindromes to optimize the process.

**Finding the Longest Palindromic Substring:**

Once the palindrome array P is computed, the algorithm finds the maximum value in the array. The index of this maximum value corresponds to the center of the longest palindromic substring, and its value indicates the radius of that palindrome. With this information, you can easily find the longest palindromic substring.

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

string longestPalindrome(string s) {
    // Inserting special characters between characters and at the beginning and end
    string new_s = "$#";
    for (char c : s) {
        new_s += c;
        new_s += '#';
    }
    new_s += '@'; // Adding sentinel character

    int n = new_s.size();
    vector<int> p(n, 0); // p[i] stores the length of the palindrome centered at i (exclude s[i])
    int center = 0, right = 0; // Center and right boundary of the palindrome

    for (int i = 1; i < n - 1; ++i) {
        int mirror = 2 * center - i;

        // Check if i is within the right boundary
        if (i < right)
            p[i] = min(right - i, p[mirror]);

        // Attempt to expand palindrome centered at i
        while (new_s[i + (1 + p[i])] == new_s[i - (1 + p[i])])
            p[i]++;

        // If palindrome centered at i expands past right boundary,
        // adjust center and right boundary accordingly
        if (i + p[i] > right) {
            center = i;
            right = i + p[i];
        }
    }

    // Find the longest palindrome and its center
    int max_len = 0, center_index = 0;
    for (int i = 1; i < n - 1; ++i) {
        if (p[i] > max_len) {
            max_len = p[i];
            center_index = i;
        }
    }

    // Extract and return the longest palindrome
    return s.substr((center_index - max_len) / 2, max_len);
}

int main() {
    string s = "babad";
    cout << "Longest palindromic substring: " << longestPalindrome(s) << endl;
    return 0;
}
```

If P[i] = k, it means that at index i, there is a palindrome of length 2*k + 1 centered at i.