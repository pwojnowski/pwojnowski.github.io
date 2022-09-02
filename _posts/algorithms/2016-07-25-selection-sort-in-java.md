---
title: Selection Sort in Java
date: 2016-07-25 07:01:00 +0200
tags: [algorithms,java]
---


**Selection Sort** is an intuitive sorting algorithm. It's not the fastest
algorithm, but a good starting point when learning about sorting. Also, due to
its simplicity, it can be used to verify implementations of harder sorting
algorithms. Let's see how it works!

<!--more-->


# How it works

The sorting algorithm consists of 3 parts:

1. Iterate over the input data.
2. Find the index of the smallest number (*selection*), starting from the
   current position up to the end of the given array.
3. Swap the smallest element with the one at current position.

Let's see how that looks in the code!


# Selection Sort implementation

The central part of the algorithm is just the execution of the above 3 steps:
```java
public void sort(int[] values) {
    // 1. Iteration over the input data
    for (int i = 0; i < values.length; i++) {

        // 2. Find the index of the smallest element
        int minPos = indexOfMinimum(values, i);
        
        // 3. Move the smallest element into the current position
        swap(values, i, minPos);
    }
}
```

To find the smallest value (minimum) in the rest of the elements, we just
compare the current minimum `values[minPos]` with the currently seen element
`values[i]`. If the value under the index `i` is smaller, we take it as the
current index of the smallest element `minPos`:

```java
private int indexOfMinimum(int[] values, int i) {
    int minPos = i;
    for (; i < values.length; ++i) {
        if (values[i] < values[minPos]) {
            minPos = i;
        }
    }
    return minPos;
}
```

And the swap of elements under the given indices - `first` and `second`:
```java
private void swap(int[] values, int first, int second) {
    int tmp = values[first];
    values[first] = values[second];
    values[second] = tmp;
}
```

Notice that all parts of the algorithm we've implemented in separate
methods. This make the code more readable and is a good style.

If you are worried about the cost of calling methods in a loop, then no need
to. *JVM Hot-Spot* is taking care of that and will optimize (e.g. inline
methods) code where needed. Better focus on writing readable code!


# Testing the implementation

Let's write a couple of [JUnit](https://farenda.com/junit-tutorial) tests to
make sure that our implementation of Selection Sort behaves correctly:
```java
import org.junit.Test;

import static org.junit.Assert.assertArrayEquals;

public class SelectionSortTest {

    private SelectionSorter sorter = new SelectionSorter();

    @Test
    public void shouldDoNothingWithEmptyArray() {
        int[] values = {};

        sorter.sort(values);
    }

    @Test
    public void shouldDoNothingWithOneElementArray() {
        int[] values = {42};

        sorter.sort(values);

        assertArrayEquals(new int[] {42}, values);
    }

    @Test
    public void shouldSortValues() {
        int[] values = { 9, -3, 5, 0, 1};
        int[] expectedOrder = { -3, 0, 1, 5, 9};

        sorter.sort(values);

        assertArrayEquals(expectedOrder, values);
    }
}
```


# Selection Sort properties

- The running time is **ϴ(n²)** (Big Theta n²), which means that it won't run
  faster nor slower than *n²*,
- Selection (`indexOfMinimum` method) will be executed `n²/2 + n/2` times,
- Memory complexity is **O(1)** (constant), because the algorithm reuses the
  input array.


# References

- [Binary Search](https://farenda.com/algorithms/binary-search-algorithm-in-java) is expecting to receive sorted data

