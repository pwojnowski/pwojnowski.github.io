---
title: Merge Sort in Java
date: 2016-11-11 14:10:00 +0200
tags: [algorithms,sorting,java]
---

In this post we'll implement Merge Sort in Java. It's fast, divide and conquer,
sorting algorithm that can be also parallelized.

<!--more-->


# How Merge Sort works

The algorithm belongs to the **Divide and Conquer** family. This, basically,
means that it divides the sorting problem into smaller parts to solve. The core
of the merge sort goes as follows:

1.  When the array consists of only one element do nothing, because it is already
    sorted.
2.  *Find the middle point* of array (or mid points when you want to split into
    more parts).
3.  Recursively merge sort part of array from the start to the middle point.
4.  Recursively merge sort part of array above the middle point.
5.  *Merge* (join) both sorted parts.

The above algorithm can be simply expressed in Java code:
```java
void mergeSort(int[] numbers, int from, int to) {
    if (from < to) {
        int mid = from + Math.floorDiv(to-from, 2);
        mergeSort(numbers, from, mid);
        mergeSort(numbers, mid+1, to);
        merge(numbers, from, mid, to);
    }
}
```

Notice that we calculate the `mid` point using subtraction instead of addition
`(from + to) / 2`, because if both values are big enough they could overflow,
which is a common error.


# Merge Sort properties

The notable properties of the *merge sort* are:

- it is a stable sort (preserves order of already sorted elements)
- best/average/worst case performance is **O(n log n)**
- memory complexity is **O(n)** (requires additional memory for n elements)
- can be parallelized, because parts of array are sorted independently


# Merge Sort implementation

Keep in mind that there are a few different ways to implement the algorithm with
various levels of complexity. The one we present here is more complex than [Selection
Sort](https://farenda.com/algorithms/selection-sort-in-java) and [Insertion
Sort](https://farenda.com/algorithms/insertion-sort-in-java), but it's readable
and very instructive to learn:
```java
package com.farenda.tutorials.algorithms.sorting;

import java.util.Arrays;

public class MergeSort {

    public void sort(int[] numbers) {
        mergeSort(numbers, 0, numbers.length-1);
    }

    void mergeSort(int[] numbers, int from, int to) {
        if (from < to) {
            int mid = from + Math.floorDiv(to-from, 2);
            mergeSort(numbers, from, mid);
            mergeSort(numbers, mid+1, to);
            merge(numbers, from, mid, to);
        }
    }

    void merge(int[] numbers, int from, int mid, int to) {
        int[] left = Arrays.copyOfRange(numbers, from, mid+1);
        int[] right = Arrays.copyOfRange(numbers, mid+1, to+1);

        int li = 0, ri = 0;
        while (li <= left.length && ri < right.length) {
            if (left[li] < right[ri]) {
                numbers[from++] = left[li++];
            } else {
                numbers[from++] = right[ri++];
            }
        }

        while (li < left.length) {
            numbers[from++] = left[li++];
        }

        while (ri < right.length) {
            numbers[from++] = right[ri++];
        }
    }
}
```


# Unit tests

Here are some [JUnit](https://farenda.com/junit-tutorial) tests to verify the implementation:
```java
package com.farenda.tutorials.algorithms.sorting;

import org.junit.Test;

import static org.junit.Assert.assertArrayEquals;
import static org.junit.Assert.assertEquals;

public class MergeSortTest {

    private MergeSort sorter = new MergeSort();

    @Test
    public void shouldDoNothingWithEmptyArray() {
        int[] values = {};

        sorter.sort(values);

        assertEquals(values.length, 0);
    }

    @Test
    public void shouldDoNothingWithOneElementArray() {
        int[] values = {42};

        sorter.sort(values);

        assertArrayEquals(new int[] {42}, values);
    }

    @Test
    public void shouldSortValues() {
        int[] values = new int[] { 9, -3, 5, 0, 1};
        int[] expectedOrder = new int[] { -3, 0, 1, 5, 9};

        sorter.sort(values);
        assertArrayEquals(expectedOrder, values);
    }
}
```


# References:

- Check out other sorting algorithms in [Algorithms articles](https://farenda.com/algorithms-and-data-structures)
