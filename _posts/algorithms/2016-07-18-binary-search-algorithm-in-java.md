---
title: Binary Search algorithm in Java
date: 2016-07-18 07:01:00 +0200
tags: [algorithms,java]
---


**Binary Search** is the fastest algorithm for finding an element in a *sorted*
list. Also it's easy to implement and very useful. Let's see how it works and
implement!

<!--more-->


# How it works

The procedure is simple:

1.  Take element from the center and compare with expected value.
2.  If found return its index.
3.  If expected value is bigger, repeat from step 1, but only for part of array
    from the second half. Because the array is sorted we know that everything
    from the first half is smaller than the target element, so no need to compare.
4.  If expected value is lower, repeat from step 1, but only for part of array
    from the first half.
5.  Return some value that indicates that the expected value was not found
    (usually -1) or `negative_index - 1` of place where it can be inserted, to
    simplify insertion for users (optional, but easy to do).


# Binary Search properties

- input array/list must be sorted
- computational complexity in the worst case is **O(log n + 1)** (when element
  is not found), best is **O(1)** (hit on the first guess)
- returns *index* of target element when found
- returns *negative index minus one* of the *insertion point* of the target
  element, in case when was not found


# Binary Search implementation

In this implementation we are working with integers, but it can be easily
changes to work with any objects that are *comparable* (or by providing a custom
`Comparator`):
```java
public class BinarySearch {

    public static int find(int[] numbers, int target) {
        int min = 0, max = numbers.length-1;

        while (min <= max) {
            int pos = (min+max) / 2;
            if (numbers[pos] == target) {
                return pos;
            }
            if (numbers[pos] < target) {
                min = pos + 1;
            } else {
                max = pos - 1;
            }
        }

        // +1, because 0 belongs to positive indices
        return -(min+1);
    }
}
```


# Unit testing

Let's write some [unit tests](https://farenda.com/junit-tutorial) to verify our
implementation. We can use [Fibonacci numbers](https://farenda.com/java/java-fibonacci)
as our *sorted array* of numbers: 
```java
import java.util.Arrays;
import org.junit.Test;

import static org.junit.Assert.assertEquals;

public class BinarySearchTest {

    private static final int[] FIBOS
            = { 1, 1, 2, 3, 5, 8, 13, 21, 34, 55 };

    // Tests will go here:
}
```

# Test to check that numbers are found correctly
Here we've added calls to `binarySearch` method from `java.util.Arrays` just to
compare the results produced by both implementations:
```java
@Test
public void shouldFindIndexOfNumber() {
    // Our version:
    assertEquals(3, BinarySearch.find(FIBOS, 3));
    assertEquals(9, BinarySearch.find(FIBOS, 55));

    // JDK version:
    assertEquals(3, Arrays.binarySearch(FIBOS, 3));
    assertEquals(9, Arrays.binarySearch(FIBOS, 55));
}
```


# Test to check insertion point

In the case when expected number is not found it's easy to return a pointer to a
place where it can be inserted. Here we're going to verify it works:
```java
@Test
public void shouldReturnNegativeInsertionPointWhenNotFound() {
    // Our version:
    assertEquals(-1, BinarySearch.find(FIBOS, 0));
    assertEquals(-5, BinarySearch.find(FIBOS, 4));

    // JDK version:
    assertEquals(-1, Arrays.binarySearch(FIBOS, 0));
    assertEquals(-5, Arrays.binarySearch(FIBOS, 4));
}
```


# References

- [JUnit Tutorial](https://farenda.com/junit-tutorial)
- [Java BinarySearch with custom Comparator](https://farenda.com/java/java-binary-search-comparator)
