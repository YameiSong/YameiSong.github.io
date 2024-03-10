---
layout: post
title:  "Binary Search Tree vs. Hash Table"
date:   2024-03-10 16:29 +1100
categories: data structure
---

## Summary

| Comparison Criteria                               | Hash Table                                            | BST                                      |
| ------------------------------------------------- | ----------------------------------------------------- | ---------------------------------------- |
| Search Time Complexity                            | O(1)                                                  | O(log n)                                 |
| Insertion Time Complexity                         | O(1)                                                  | O(log n)                                 |
| Deletion Time Complexity                          | O(1)                                                  | O(log n)                                 |
| Memory Overhead                                   | High                                                  | Low                                      |
| Range Searches                                    | Requires special implementation                       | Efficient                                |
| Rebalancing                                       | Not necessary                                         | Required for self-balancing BSTs         |
| Ordering                                          | Not inherently ordered                                | Inherently ordered with sorted traversal |
| Recursion                                         | Not possible                                          | Possible                                 |
| Handling Collisions                               | Hash function and collision resolution strategies     | Not applicable                           |
| Implementation                                    | Relies on libraries provided by programming languages | Can be easily implemented and customized |
| Multiple Keys with Same Value                     | Not possible                                          | Possible                                 |
| Suitability for Small Data Sets with Few Elements | Less suitable due to memory overhead                  | More suitable                            |
| Mergeability                                      | Not straightforward                                   | Easily merged with B+ trees              |

## Advantages of BST over Hash Table

1. We can get all keys in **sorted order** by just doing Inorder Traversal of BST. This is not a natural operation in Hash Tables and requires extra efforts.
2. With Self-Balancing BSTs, all operations are **guaranteed** to work in O(Logn) time. But with Hashing, O(1) is average time and some particular operations may be costly i.e, O(n), especially when there are many collisions and the hash table needs to be resized.