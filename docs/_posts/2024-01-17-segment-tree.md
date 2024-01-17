---
layout: post
title:  "Segment Tree"
date:   2024-01-17 11:37 +1100
categories: database
---

## Solve Problems

- Point Update, Range Query
- Range Update, Point Query
- Range Update, Range Query
- Query: sum, max, min

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