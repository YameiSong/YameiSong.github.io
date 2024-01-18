---
layout: post
title:  "Segment Tree"
date:   2024-01-17 11:37 +1100
categories: algorithm
---

## Problems Suitable for Segment Trees

Segment trees are a versatile data structure that can efficiently solve a variety of problems related to range queries and updates. Here are some common problems for which segment trees are particularly well-suited:

1. **Range Sum Queries:**
   - *Problem:* Given an array, perform queries to find the sum of elements in a given range.
   - *Solution:* Segment trees can be used to store the cumulative sum of elements in various ranges, allowing for efficient range sum queries.

2. **Range Minimum/Maximum Queries:**
   - *Problem:* Find the minimum or maximum element in a given range of an array and update elements in the array.
   - *Solution:* Segment trees can maintain minimum or maximum values in specific ranges and quickly answer queries for range minimum or maximum.

3. **Counting Elements in a Range:**
   - *Problem:* Count the number of elements in a given range that satisfy a certain condition.
   - *Solution:* Segment trees can store the count of elements in various ranges, facilitating quick queries on the count.

4. **Finding the First Occurrence or Last Occurrence:**
   - *Problem:* Find the index of the first or last occurrence of a particular element in a given range of an array.
   - *Solution:* Segment trees can be adapted to efficiently find the first or last occurrence of an element in a specified range.

5. **Interval Overlapping:**
   - *Problem:* Determine if there is any overlap between intervals in a set of intervals.
     - Add 1 on an interval when performing operations on it
     - Query sum over an interval to check overlap (sum > 0)
   - *Solution:* Segment trees can help efficiently check for overlapping intervals and handle interval-related queries.

## Functions

- Point Update
- Range Update
- Point Query: value
- Range Query: sum, max, min

## Range Add Template

```python
class Node:
    def __init__(self):
        self.val = 0
        self.left = None
        self.right = None
        self.lazy = 0
        # use "self.updated" when self.lazy=0 means "update value to 0", otherwise omit it (just use "lazy" is enough)


class SegmentTree:
    # This segement tree maintains the "sum" state, which is also reflected on "pushUp" and "pushDown" (briefly, cur = left + right)
    # if we want to maintain the "max" state, we should change "pushUp" and "pushDown" accordingly (briefly, cur = max(left, right))

    def __init__(self, n):
        self.n = n
        self.root = Node()
            
    def query(self, index):
        return self.innerQuery(index, 0, self.n, self.root)
    
    def addToPoint(self, index, x):
        self.innerAddToPoint(index, x, 0, self.n, self.root)
    
    def updateToPoint(self, index, x):
        self.innerUpdateToPoint(index, x, 0, self.n, self.root)

    def addToRange(self, left, right, x):
        self.innerAddToRange(left, right, x, 0, self.n, self.root)
    
    def sum(self, left, right):
        return self.innerSum(left, right, 0, self.n, self.root)

    def addNode(self, cur):
        if cur.left is None:
            cur.left = Node()
        if cur.right is None:
            cur.right = Node()

    def pushUp(self, cur):
        cur.val = cur.left.val + cur.right.val

    def pushDown(self, cur, s, c, t):
        lchild = cur.left
        rchild = cur.right
        lchild.val += (c - s + 1) * cur.lazy
        lchild.lazy += cur.lazy
        rchild.val += (t - c) * cur.lazy
        rchild.lazy += cur.lazy
        cur.lazy = 0
    
    def innerQuery(self, index, s, t, cur):
        if s == t:
            return cur.val

        self.addNode(cur)
        c = (s + t) // 2
        if cur.lazy:
            self.pushDown(cur, s, c, t)
        if index <= c:
            return self.innerQuery(index, s, c, cur.left)
        else:
            return self.innerQuery(index, c + 1, t, cur.right)
    
    def innerAddToPoint(self, index, x, s, t, cur):
        if s == t:
            cur.val += x
            return

        self.addNode(cur)
        c = (s + t) // 2
        if cur.lazy:
            self.pushDown(cur, s, c, t)
        if index <= c:
            return self.innerAddToPoint(index, x, s, c, cur.left)
        else:
            return self.innerAddToPoint(index, x, c + 1, t, cur.right)
        self.pushUp(cur)
                
    def innerUpdateToPoint(self, index, x, s, t, cur):
        if s == t:
            # This is the only difference with "innerAddToPoint"
            cur.val = x
            return

        self.addNode(cur)
        c = (s + t) // 2
        if cur.lazy:
            self.pushDown(cur, s, c, t)
        if index <= c:
            return self.innerAddToPoint(index, x, s, c, cur.left)
        else:
            return self.innerAddToPoint(index, x, c + 1, t, cur.right)
        self.pushUp(cur)

    def innerAddToRange(self, left, right, x, s, t, cur):
        if left <= s and t <= right:
            cur.val += (t - s + 1) * x
            if s != t:
                cur.lazy += x
            return

        self.addNode(cur)
        c = (s + t) // 2
        if cur.lazy:
            self.pushDown(cur, s, c, t)
        if left <= c:
            self.innerAddToRange(left, right, x, s, c, cur.left)
        if right > c:
            self.innerAddToRange(left, right, x, c + 1, t, cur.right)
        self.pushUp(cur)
    
    def innerSum(self, left, right, s, t, cur):
        if left <= s and t <= right:
            return cur.val

        self.addNode(cur)
        c = (s + t) // 2
        ans = 0
        if cur.lazy:
            self.pushDown(cur, s, c, t)
        if left <= c:
            ans += self.innerSum(left, right, s, c, cur.left)
        if right > c:
            ans += self.innerSum(left, right, c+1, t, cur.right)
        return ans
```

## Range Update Template

```python
class Node:
    def __init__(self):
        self.val = 0
        self.left = None
        self.right = None
        self.lazy = 0
        self.updated = False
        # use "self.updated" when self.lazy=0 means "update value to 0", otherwise omit it (just use "lazy" is enough)


class SegmentTree:
    # This segement tree maintains the "max" state, which is also reflected on "pushUp" and "pushDown" (briefly, cur = max(left, right))
    # if we want to maintain the "sum" state, we should change "pushUp" and "pushDown" accordingly (briefly, cur = left + right)

    def __init__(self, n):
        self.n = n
        self.root = Node()
            
    def query(self, index):
        return self.innerQuery(index, 0, self.n, self.root)
    
    def addToPoint(self, index, x):
        self.innerAddToPoint(index, x, 0, self.n, self.root)
    
    def updateToPoint(self, index, x):
        self.innerUpdateToPoint(index, x, 0, self.n, self.root)

    def updateToRange(self, left, right, x):
        self.innerUpdateToRange(left, right, x, 0, self.n, self.root)
    
    def max(self, left, right):
        return self.innerMax(left, right, 0, self.n, self.root)

    def addNode(self, cur):
        if cur.left is None:
            cur.left = Node()
        if cur.right is None:
            cur.right = Node()

    def pushUp(self, cur):
        cur.val = max(cur.left.val, cur.right.val)

    def pushDown(self, cur):
        lchild = cur.left
        rchild = cur.right
        lchild.val = cur.lazy
        lchild.lazy = cur.lazy
        lchild.updated = True
        rchild.val = cur.lazy
        rchild.lazy = cur.lazy
        rchild.updated = True
        cur.lazy = 0
        cur.updated = False
    
    def innerQuery(self, index, s, t, cur):
        if s == t:
            return cur.val

        self.addNode(cur)
        c = (s + t) // 2
        if cur.updated:
            self.pushDown(cur)
        if index <= c:
            return self.innerQuery(index, s, c, cur.left)
        else:
            return self.innerQuery(index, c + 1, t, cur.right)
    
    def innerAddToPoint(self, index, x, s, t, cur):
        if s == t:
            cur.val += x
            return

        self.addNode(cur)
        c = (s + t) // 2
        if cur.updated:
            self.pushDown(cur)
        if index <= c:
            return self.innerAddToPoint(index, x, s, c, cur.left)
        else:
            return self.innerAddToPoint(index, x, c + 1, t, cur.right)
        self.pushUp(cur)
                
    def innerUpdateToPoint(self, index, x, s, t, cur):
        if s == t:
            # This is the only difference with "innerAddToPoint"
            cur.val = x
            return

        self.addNode(cur)
        c = (s + t) // 2
        if cur.updated:
            self.pushDown(cur)
        if index <= c:
            return self.innerAddToPoint(index, x, s, c, cur.left)
        else:
            return self.innerAddToPoint(index, x, c + 1, t, cur.right)
        self.pushUp(cur)

    def innerUpdateToRange(self, left, right, x, s, t, cur):
        if left <= s and t <= right:
            cur.val = x
            if s != t:
                cur.lazy = x
                cur.updated = True
            return

        self.addNode(cur)
        c = (s + t) // 2
        if cur.updated:
            self.pushDown(cur)
        if left <= c:
            self.innerUpdateToRange(left, right, x, s, c, cur.left)
        if right > c:
            self.innerUpdateToRange(left, right, x, c + 1, t, cur.right)
        self.pushUp(cur)
    
    def innerMax(self, left, right, s, t, cur):
        if left <= s and t <= right:
            return cur.val

        self.addNode(cur)
        c = (s + t) // 2
        ans = 0 # set a valid minimum value
        if cur.updated:
            self.pushDown(cur)
        if left <= c:
            ans = max(ans, self.innerMax(left, right, s, c, cur.left))
        if right > c:
            ans = max(ans, self.innerMax(left, right, c+1, t, cur.right))
        return ans
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