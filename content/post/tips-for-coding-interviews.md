---
title: "Tips for Coding Interviews"
date: 2017-01-14T22:35:47+09:00
lastmod: 2017-01-14T22:35:47+09:00
draft: false
keywords: []
description: ""
tags: ['algorithms', 'data structures']
categories: ['Learning']

isCJKLanguage: false

comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
showMeta: true
showActions: true

---

<!-- toc -->

# Primer

* When target is string, confirm whether it's ASCII, which at most has 256 possible characters. So use space to boost time performance.

* whenever only flags or booleans in an array, use bit vector instead for better space complexity.
* For linked list problems, usually a pair of slow pointer and fast pointer would be a nice try.gb

# Tree

## Depth/Height

* To get the height (from leaf) of each node in a binary tree,  use following recursive code:
```
function getHeight(TreeNode curr) {
    if (curr == null) return 0;
    return Math.max(getHeight(curr.left), 
                    getHeight(curr.right)) + 1;
}
```
So the max depth of a tree is ```getHeight(root)```, and to get min depth, just recursively call ```Math.min(minDepth(curr.left), minDepth(curr.right)) + 1```;

## Lowest Common Ancestor
 If it is a BST, make use of its characteristics:
```
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if(root.val > p.val && root.val > q.val){
        return lowestCommonAncestor(root.left, p, q);
    }else if(root.val < p.val && root.val < q.val){
        return lowestCommonAncestor(root.right, p, q);
    }else{
        return root;
    }
}
```
Otherwise, for normal binary trees, 
```
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    if (left != null && right != null) return root;
    return left != null ? left : right;
    }
```

## Pre, In, and Post Order

* pre-order: Use a queue. Print current node and then add curr.left and curr.right to queue.
* in-order: Use a stack. Push all left nodes, and whenever pops a node, push all left nodes again in its subtree.
* post-order:  A recursive approach is easy.

# Linked List
* Use two pointers

# String
* Before starting, ask the interviewer to clarify some points:
    * ASCII? Unicode?
    * case sensitive or not?
    * is spaces significant?
* ASCII characters can fit into a char[256] array. That will be a great hash table.
* When modifying a string in-place, to avoid overwriting some characters, start from the end.
* To convert a char to a digit, use ```theChar - '0'```
* 


# Good Example Questions

1. Write a program to determine whether a binary tree is a binary search tree.
2. Implement an algorithm to remove all numbers appearing more than once in an unsorted array.
3. Write a program to convert a sorted array into a balanced binary search tree.
4. Implement an algorithm to replace all spaces in a string with "%20". And what about vice versa?
5. Print a binary tree level by level.
6. Find different numbers from two sorted arrays.
7. Determine if two strings are [anagrams](https://en.wikipedia.org/wiki/Anagram).
8. Merge two sorted arrays. What about two sorted linked lists?
9. Search for pairs of numbers in an array so that the two numbers's sum are a predefined constant C.
10. Find the minimum element in a rotated version of sorted array.


# Interview Tips

* Clarify the problem by asking interviewer questions
    * space requirements
    * function signature (input and output)
* Think aloud
* Done is better than perfect

