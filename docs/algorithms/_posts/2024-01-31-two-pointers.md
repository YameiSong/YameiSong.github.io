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
        int mirror = 2 * center - i; // center - (i - centers)

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

## Floyd's Cycle Detection Algorithm

### How it Works:

1. **The Setup:**
   - You have two pointers: one slow pointer (the "tortoise") and one fast pointer (the "hare").
   - The slow pointer moves one step at a time, while the fast pointer moves two steps at a time.

2. **Advancing the Pointers:**
   - At each step, move the slow pointer one step forward and the fast pointer two steps forward.
   - If there's no cycle in the list, the fast pointer will eventually reach the end of the list (NULL), and we can stop.
   - However, if there is a cycle, the fast pointer will eventually catch up to the slow pointer, and they will meet at some node within the cycle.

3. **Detecting the Cycle:**
   - If the fast pointer ever equals the slow pointer (i.e., they meet), then there's definitely a cycle in the linked list.
   - This is because the fast pointer moves twice as fast as the slow pointer, so it will "lap" the slow pointer eventually if there's a cycle.

4. **Finding the Start of the Cycle (Floyd's Algorithm for Cycle Detection):**
   - Once the tortoise and hare meet, move one of the pointers (let's say the slow pointer) back to the beginning of the list.
   - Now, move both pointers one step at a time.
   - The point at which they meet again will be the start of the cycle.

   1. **Lengths and Positions:**
      - Let's denote the length of the non-cyclic part of the linked list from the head to the start of the cycle as $ L $.
      - Let's denote the length of the cycle itself as $ C $.
      - Let's denote the position of the meeting point of the tortoise and hare within the cycle as $ P $.
      - Let's denote the position of the cycle's starting point from the head as $ S $.

   2. **Advancement of Pointers:**
      - The tortoise moves $ L + P $ steps.
      - The hare moves $ 2 \times (L + P) $ steps.

   3. **Relation between Pointer Movements:**
      - As the hare moves twice as fast as the tortoise, the hare's position is twice the tortoise's position within the cycle when they meet:
        $ 2 \times (L + P) = L + P + n \times C $
        where $ n $ is the number of times the hare has gone around the cycle when they meet.

   4. **Solving for L and P:**
      - After simplifying the equation, we find that $ L = (n - 1) \times C + (C - P) $, where $ n $ is an integer.
      - We reset one of the pointers (let's say the tortoise) to the head of the list and keep the hare at the meeting point.
      - Both pointers move one step at a time until they meet again.
      - The meeting point will be the start of the cycle $ S $.

   5. **Algorithm Complexity:**
      - The complexity of this algorithm is $ O(L + C) $, where $ L $ is the length of the non-cyclic part and $ C $ is the length of the cycle.
      - This algorithm requires only two pointers and no additional data structures, making it memory-efficient.

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def detectCycle(head):
    # Initialize slow and fast pointers
    slow = head
    fast = head

    # Check for cycle
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        # If slow and fast meet, there's a cycle
        if slow == fast:
            break
    
    # No cycle detected
    if not fast or not fast.next:
        return None

    # Move one pointer back to the head
    slow = head

    # Find the start of the cycle
    while slow != fast:
        slow = slow.next
        fast = fast.next

    # Return the start of the cycle
    return slow

# Test the algorithm
if __name__ == "__main__":
    # Create a linked list with a cycle
    head = ListNode(3)
    head.next = ListNode(2)
    head.next.next = ListNode(0)
    head.next.next.next = ListNode(-4)
    head.next.next.next.next = head.next  # Create cycle

    # Detect the cycle
    cycle_start = detectCycle(head)
    if cycle_start:
        print("Cycle starts at node with value:", cycle_start.val)
    else:
        print("No cycle detected.")

```

## Binary Search

```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1

    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1

# Test the binary search algorithm
arr = [1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
target = 13
print(binary_search(arr, target))  # Output: 6 (index of target value 13 in arr)
```

### Lower Bound

```python
def lower_bound(arr, target):
    left, right = 0, len(arr)

    while left < right:
        mid = left + (right - left) // 2
        if arr[mid] < target:
            left = mid + 1
        else:
            right = mid

    return left

# Test the lower_bound function
arr = [1, 2, 3, 3, 3, 4, 5]
target = 3
print(lower_bound(arr, target))  # Output: 2 (index of the lower bound)
```

#### Choice of left and right boundary

1. **Updating `right = mid`:**
   - If the value at the midpoint is greater than or equal to the target, it's still possible that the current midpoint is the desired lower bound.
   - Therefore, we update `right = mid` to include the current midpoint in the search space for the next iteration.
   - This ensures that if the current midpoint is the lower bound, we still include it in the search space for further validation.

2. **Updating `left = mid + 1`:**
   - If the value at the midpoint is less than the target, we know that the lower bound cannot be at the current midpoint or to the left of it.
   - Therefore, we update `left = mid + 1` to exclude the current midpoint from the search space for the next iteration.
   - This ensures that we narrow down the search space and prevent unnecessary comparisons with the current midpoint.

In both cases, the goal is to reduce the search space efficiently and ensure that the algorithm converges to the correct lower bound.

It's important to consider the specific requirements of the problem and choose the appropriate update strategy for `left` and `right` pointers to achieve the desired behavior and efficiency in binary search.
