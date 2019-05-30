---
title: "Algorithms and Data Structures"
description: ""
keywords: []
tags: []
categories: []
author: ""
date: 2017-02-14T22:34:46+09:00
lastmod: 2017-02-14T22:34:46+09:00
draft: false
isCJKLanguage: false

# thumbnailImage: image-1.png
# thumbnailImagePosition: bottom
# autoThumbnailImage: yes


comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
showMeta: true
showActions: true
---

<!-- toc -->

# Time Complexity
Usually only the worst-case running time is considered because:

* it gives an upper bound
* For many situations, the worst case occurs fairly often. (Eg. searching for items which may frequently be absent)
* The average case is often as bad as worst case (:TO-CHECK:)

To easily describe the performance of an algorithm, we usually use **order of growth**, e.g., O(n^2).

# Sorting Algorithms

[Comparison of sorting algorithms](https://www.cs.usfca.edu/~galles/visualization/ComparisonSort.html)

## Selection Sort

> All cases: O(n^2) compares and O(1) swaps

Two important properties of selection sort:

* Running time is insensitive to input
* Data movement is minimal

## Insertion Sort

> Worst case: O(n^2) compares, O(n^2) swaps

> Average case: O(n^2) compares, O(n^2) swaps

> Best case: O(n) compares and O(1) swaps

Unlike that of selection sort, the running time of insertion sort depends on the initial order of the items in the input. For example, if the array is large and its entries are already in order (or nearly in order), then insertion sort is much, much faster than if the entries are randomly ordered or in reverse order.

More generally, arrays with any one of the following characteristics can be efficiently sorted using insertion sort:

* An array where each entry is not far from its final position
* A small array appended to a large sorted array
* An array with only a few entries that are not in place

In summary, insertion sort is an excellent method for partially sorted arrays and is also a fine method for tiny arrays. These facts are important not just because such arrays frequently arise in practice, but also because both types of arrays arise in intermediate stages of advanced sorting algorithms

## Shell Sort

> Worst case: lower than O(n^2), like O(n^1.5) or O(n^1.3),

> depending on the increment sequence used.

Shell sort is a simple extension of insertion sort that gains speed by allowing exchanges of array entries that are far apart, to produce partially sorted arrays that can be efficiently sorted, eventually by insertion sort.

Shell sort is much faster than insertion sort and selection sort, and its speed advantage increases with the array size.

Experienced programmers sometimes choose shell sort because

* it has acceptable running time even for moderately large arrays;
*  it requires a small amount of code;
*  and it uses no extra space.


## Merge Sort

One of merge sort’s most attractive properties is that it guarantees to sort any array of N items in time proportional to NlogN. Its prime disadvantage is that it uses extra space proportional to N.

Some modifications could be applied to improve merge sort:

* Use insertion sort for small subarrays
* Test whether the array is already in order by adding a test to skip the call to merge() if a[mid] is less than or equal to a[mid+1]


## QuickSort

> Worst case: O(n^2)

> Average case: O(n log n)
Some algorithmic improvements:

Quick sort requires an extra space of lgN because of the recursive call stack, which cannot be convert to iterative calls. (BTW, merge sort can)

* Cutoff to insertion sort
* Median-of-three partitioning: use the median of a small sample of items(e.g. 3 items) taken from the subarray as the partitioning item.


## Priority Queue

Useful when input data is too large in size to be completely sorted, and we only care about some of the largest or smallest items.

> In an N-key priority queue, the heap algorithms require no more than 1 + lg N compares for insert and no more than 2lg N compares for remove the maximum.


## Heapsort

Heapsort can be implemented by using priority queue. We insert all the items to be sorted into a minimum-oriented priority queue, then repeatedly use remove the minimum to remove them all in order.

The binary heap is a data structure that can efficiently support the basic priority-queue operations.

![Binary Heap](/images/algorithms-and-data-structures/Screen-Shot-2017-02-24-at-12.45.27.png)

The parent of the node in position k is in position ⎣k /2⎦ and, con- versely, the two children of the node in position k are in positions 2k and 2k + 1.

Heapsort is optimal in both time and space, but is rarely used in typical applications on modern systems because it has poor cache performance: array entries are rarely compared with nearby array entries, so the number of cache misses is far higher than for quicksort, mergesort, and even shellsort, where most compares are with nearby entries.

![Summary of sort algorithms](/images/algorithms-and-data-structures/sort_algorithms.png)


# Search Algorithms

## Binary Search

Binary search in an ordered array with N keys uses no more than lg N + 1 compares for a search (successful or unsuccessful).

However, inserting new items into the ordered array is a N^2 operation.


## Binary Search Tree

The running times of algorithms on binary search trees depend on the shapes of the trees, which, in turn, CS depend on the order in which keys are inserted.

But the problem of BST is the worst-case performance is poor: O(N).

## Balanced Binary Search Tree (2-3 Tree, RedBlack Tree)
To always guarantee logarithmic performance no matter what sequence of keys is used to construct the tree, we would like to keep our binary search trees perfectly balanced.

In an N-node tree, we would like the height to be ~lg N so that we can guarantee that all searches can be completed in ~lg N compares.

Unfortunately, maintaining perfect balance for dynamic insertions is too expensive. In this section, we consider a data structure that slightly re- laxes the perfect balance requirement to provide guaranteed logarithmic performance not just for the insert and search operations in our symbol-table API but also for all of the ordered operations (except range search).

**2-3 Tree**

Search and insert operations in a 2-3 tree with N keys are guaranteed to visit at most lg N nodes.

Unlike standard BSTs, which grow down from the top, 2-3 trees grow up from the bottom.

**RedBlack Tree**

Use red links to connect two 2-nodes as a 3-node.

## Hash Table

First, compute hash using hash function. Second, resolve the possible collision.

Two approaches to collision-resolution:

* separate chaining
* linear probing

Hashing is a classic example of a time-space tradeoff. If there were no memory limitation, then we could do any search with only one memory access by simply using the key as an index in a (potentially huge) array.

Popular hashing functions:

* **modular hashing for integer keys**: choose the array size M to be prime and, for any positive integer key k, compute the remainder when dividing k by M. (If M is not prime, it may be the case that not all of the bits of the key play a role)
* **for floating numbers**: use modular hashing on the binary representation of the key

## Pros and Cons of Search Algorithms
![Summary of search algorithms](/images/algorithms-and-data-structures/search_algorithms.png)

## Time Complexity of Search Algorithms
![Complexity of search algorithms](/images/algorithms-and-data-structures/search_algorithms_time.png)


