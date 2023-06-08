---
title: Quicksort in Java
date: 2016-12-16 12:40:00 +0200
tags: [algorithms,java]
---

Quicksort is one of the fastest sorting algorithms. In this article we implement
Quicksort in Java, describe how it works and its properties.

<!--more-->


# How Quicksort works

Similarly to the [merge sort](https://farenda.com/algorithms/merge-sort-in-java),
quicksort belongs to Divide and Conquer group of algorithms. It consists of the
following steps:

1.  Pick an element that will serve as comparison point - *pivot*.
2.  Partition step - reorders elements in such a way that elements *lower than
    pivot* are on its *left side*, and *greater or equal to pivot* are on its
    right side.
3.  Recursively sort subarrays on both sides of *pivot element*.

Contrary to the merge sort, where real work was done in the merge step, in
quicksort the real work is performed in partitioning step. And it is the
**partition function** that differentiate implementations of quicksort.

The algorithm can be also parallelized, because it is a **divide and conquer**
algorithm. But the *pallalelization* is worse than in case of merge sort,
because the sizes of subarrays are *sensitive* to the *selection of pivot*
element.


## Quicksort Pivot calculation strategies

- pick the first/last element: easy to implement, but will downgrade
  performance, when the data is already sorted (why?)
- pick an element from the middle
- pick random element
- calculate median of three (first, medium, and last): better performance in
  case of sorted data
- calculate median of medians.


# Quicksort properties

Quicksort has the following properties:

- worst case performance is **O(n^2)** (selected pivot doesn't split much)
- average case performance is **O(n log n)**
- worst case space complexity is **O(n)** or **O(log n)** (optimized version)
- has many implementations (stable and unstable, in-place or not)


# Quicksort implementation

In the following quicksort implementation we always use the last element of
subarray as the pivot element:
```java
package com.farenda.tutorials.algorithms.sorting;

public class QuickSort {

    public void sort(int[] values) {
        sort(values, 0, values.length-1);
    }

    private void sort(int[] values, int first, int last) {
        if (first < last) {
            int pivot = partition(values, first, last);
            sort(values, first, pivot-1);
            sort(values, pivot+1, last);
        }
    }

    private int partition(int[] values, int first, int last) {
        int greater = first;
        for (int i = first; i < last; ++i) {
            if (values[i] <= values[last]) {
                swap(values, i, greater++);
            }
        }
        swap(values, greater, last);
        return greater;
    }

    private void swap(int[] values, int i, int j) {
        int tmp = values[i];
        values[i] = values[j];
        values[j] = tmp;
    }
}
```

As an exercise implement pivot selection using the **median-of-three** strategy
(calculate the median of the first, medium, and the last element). This strategy
is more resilient to sorted data.


# References:

- See other sorting [algorithms](https://farenda.com/algorithms-and-data-structures)
