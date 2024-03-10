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

## Hash Collision

A hash collision occurs when two distinct keys produce the same hash value when hashed by a hash function. In other words, multiple keys map to the same index or bucket in a hash table.

Hash collisions can impact the performance and efficiency of a hash table. If collisions are frequent, it can degrade the time complexity of operations like insertion, deletion, and lookup, as the hash table needs to handle more elements at the same index.

When a collision occurs, the hash table must resolve it by handling multiple keys that hash to the same index. There are several common strategies for collision resolution:

1. Chaining: In chaining, each bucket in the hash table contains a linked list (or another data structure) of all the elements that hash to that particular index. When a collision occurs, the new element is simply appended to the end of the linked list. Chaining allows for an unlimited number of elements to hash to the same index.

2. Open Addressing: In open addressing, when a collision occurs, the hash table looks for another, possibly vacant, slot (usually determined by a probe sequence) to place the colliding element. This method typically involves finding the next available slot by using a collision resolution technique like linear probing, quadratic probing, or double hashing.
