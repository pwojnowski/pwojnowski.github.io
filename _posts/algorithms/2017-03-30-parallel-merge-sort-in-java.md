---
title: Parallel Merge Sort in Java
date: 2017-03-30 19:40:00 +0200
tags: [algorithms,java,concurrency]
---

*Divide and Conquer* algorithms are great subject for **parallelism**. Here we
 present **Parallel Merge Sort** implemented using *Java ForkJoin Framework*.

# Parallel sort steps

Parallel Merge Sort consists of the same steps as other tasks executed in
[ForkJoin Pool](https://farenda.com/java/java-fork-join-example), namely:

1.  If the task is small enough - do it directly.
    In case of [Merge Sort](https://farenda.com/algorithms/merge-sort-in-java)
    we can decide that when a subarray to sort is smaller than some value (we
    name it `SORT_THRESHOLD`), then we'll use [Insertion
    Sort](https://farenda.com/algorithms/insertion-sort-in-java), which will
    sort it in-place. 
2.  Else do Merge Sort of both parts as separate tasks, executed in a **ForkJoin
    Pool**. This will allow us to sort parts of an array independently and then
    just merge them.


# MergSort as RecursiveAction

The implementation consists of a few key points we need to do:

- extend `RecursiveAction` to allow execution in ForkJoin Pool
- decide when to sort directly
- execute parallel tasks for subarrays
- merge everything at the end.

Therefore the heart of the Parallel Merge Sort with Fork Join Framework looks
like this:
```java
import java.util.concurrent.RecursiveAction;

public class ParallelMergeSort extends RecursiveAction {

    // Decides when to fork or compute directly:
    private static final int SORT_THRESHOLD = 128;

    // Defines subarray to sort:
    public ParallelMergeSort(int[] values, int from, int to)

    @Override
    protected void compute() {
        if (from < to) {
            int size = to - from;
            if (size < SORT_THRESHOLD) {
                // Sort in-place using Insertion Sort:
                insertionSort();
            } else {
                int mid = from + Math.floorDiv(size, 2);
                // Do MergeSort in parallel on subarrays.
                // This will submit the tasks to the current
                // ForkJoin Pool:
                invokeAll(
                        new ParallelMergeSort(values, from, mid),
                        new ParallelMergeSort(values, mid + 1, to));
                // Merge sorted subarrays:
                merge(mid);
            }
        }
    }
    // ...
}
```

# Complete Parallel Merge Sorter

This is complete implementation of Parallel Merge Sort using **Fork-Join
Framework** to do parallel execution:
```java
import java.util.Arrays;
import java.util.concurrent.RecursiveAction;

public class ParallelMergeSort extends RecursiveAction {

    // Decides when to fork or compute directly:
    private static final int SORT_THRESHOLD = 128;

    private final int[] values;
    private final int from;
    private final int to;

    public ParallelMergeSort(int[] values) {
        this(values, 0, values.length-1);
    }

    public ParallelMergeSort(int[] values, int from, int to) {
        this.values = values;
        this.from = from;
        this.to = to;
    }

    public void sort() {
        compute();
    }

    @Override
    protected void compute() {
        if (from < to) {
            int size = to - from;
            if (size < SORT_THRESHOLD) {
                insertionSort();
            } else {
                int mid = from + Math.floorDiv(size, 2);
                invokeAll(
                        new ParallelMergeSort(values, from, mid),
                        new ParallelMergeSort(values, mid + 1, to));
                merge(mid);
            }
        }
    }

    private void insertionSort() {
        for (int i = from+1; i <= to; ++i) {
            int current = values[i];
            int j = i-1;
            while (from <= j && current < values[j]) {
                values[j+1] = values[j--];
            }
            values[j+1] = current;
        }
    }

    private void merge(int mid) {
        int[] left = Arrays.copyOfRange(values, from, mid+1);
        int[] right = Arrays.copyOfRange(values, mid+1, to+1);
        int f = from;

        int li = 0, ri = 0;
        while (li < left.length && ri < right.length) {
            if (left[li] <= right[ri]) {
                values[f++] = left[li++];
            } else {
                values[f++] = right[ri++];
            }
        }

        while (li < left.length) {
            values[f++] = left[li++];
        }

        while (ri < right.length) {
            values[f++] = right[ri++];
        }
    }
}
```

# Test and performance comparison

As usually we need to check somehow that it works. :wink: But for brevity we'll
show only [JUnit](https://farenda.com/junit-tutorial) test for substantial
amount of data (100 millions) and we'll do that for both [Merge
Sort](https://farenda.com/algorithms/merge-sort-in-java) and Parallel Merge
Sort:
```java
import org.junit.Test;

import java.util.Arrays;
import java.util.Random;

import static org.junit.Assert.assertArrayEquals;

public class ParallelMergeSortTest {

    //... other tests cut for brevity

    @Test
    public void performanceTest() {
        int[] serial = new Random().ints(100_000_000).toArray();
        int[] parallel = Arrays.copyOf(serial, serial.length);

        MergeSort mergeSort = new MergeSort();
        long start = System.currentTimeMillis();
        mergeSort.sort(serial);
        System.out.println("Merge Sort done in: "
                + (System.currentTimeMillis()-start));

        ParallelMergeSort sorter = new ParallelMergeSort(parallel);
        start = System.currentTimeMillis();
        sorter.sort();
        System.out.println("Parallel Merge Sort done in: "
                + (System.currentTimeMillis()-start));

        assertArrayEquals(parallel, serial);
    }
}
```

I run that a few times on my laptop with 2 cores and the results were around this:

    Merge Sort done in: 31077
    Parallel Merge Sort done in: 20020

Clearly Parallel Merge Sort is much faster for arrays of 100M of elements. The
speedup is not 2x, because there are overheads for scheduling tasks in pool, but
still the difference is substantial.

Both sorting algorithms use [Insertion
Sort](https://farenda.com/algorithms/insertion-sort-in-java) when subarray to
sort is below defined threshold. I found value **128** to give pretty good
results in my environment - maybe there is a better value around, but, in my
case, it gave better results than values 64, 256, 512, 768, 1024, etc.


# References

- See details of [Insertion Sort](https://farenda.com/algorithms/insertion-sort-in-java)
- See detailed explanation of [Merge Sort](https://farenda.com/algorithms/merge-sort-in-java)
- [ForkJoin explained](https://farenda.com/java/java-fork-join-example)
- Check out other algorithms in [Algorithms section](https://farenda.com/tags/algorithms)

