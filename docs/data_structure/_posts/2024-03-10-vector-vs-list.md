---
layout: post
title:  "Vecotr vs. List"
date:   2024-03-10 16:40 +1100
categories: data structure
---

## Vector

Vector is a type of dynamic array which has the ability to resize automatically after insertion or deletion of elements. The elements in vector are placed in **contiguous storage** so that they can be accessed and traversed using iterators. Element is inserted at the end of the vector.

## List

List is a double linked sequence that supports both forward and backward traversal. The time taken in the insertion and deletion in the beginning, end and middle is constant. It has the **non-contiguous memory** and there is no pre-allocated memory.


## Summary

| Comparison Criteria       | C++ Vector                                       | C++ List                                    |
| ------------------------- | ------------------------------------------------ | ------------------------------------------- |
| Underlying Data Structure | Dynamic array                                    | Doubly linked list                          |
| Random Access             | O(1)                                             | O(n)                                        |
| Insertion/Deletion at End | O(1) amortized                                   | O(1)                                        |
| Insertion/Deletion Middle | O(n)                                             | O(1) with iterator                          |
| Space Complexity          | Linear in the number of elements stored          | Linear in the number of elements stored     |
| Memory Overhead           | Lower compared to lists due to contiguous memory | Higher due to maintaining links in the list |
| Suitable Use Cases        | Fast random access, dynamic resizing             | Frequent insertion/deletion at any position |
