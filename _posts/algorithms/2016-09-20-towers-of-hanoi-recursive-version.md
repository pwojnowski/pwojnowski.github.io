---
title: Towers of Hanoi recursive version
date: 2016-09-20 15:40:00 +0200
categories: [Algorithms]
tags: [java]
---


**Towers of Hanoi** is a well known puzzle, very often used to teach recursion. In
this post we show how to solve the puzzle recursively.

<!--more-->


# The rules

There are three towers/pegs/rods with a number of disks on one of them. The
disks are of different sizes, stacked one on another, and the sorted from
smallest to the biggest (on the bottom). The objective is to move entire stack
to another peg, but:

-   only one disk can be moved at a time,
-   only top disks can be moved,
-   disks cannot be put on top of smaller disk.

The complexity of the solution is **2^n - 1**.


# Towers as stacks

In this implementation we keep pegs/towers as stacks to simplify handling of
disks. You can use arrays here, but then you would have to *manually* track where
you are on each peg. We use [Collections.asLifoQueue()](https://farenda.com/java/java-deque-stack-queue) to turn **LinkedList** into a
**Stack**.
<br />
Also to easily identify each peg we've created a helpful *enum*.

```java
enum Peg {A, B, C}

private final EnumMap<Peg, Queue<Integer>> pegs
        = new EnumMap<>(Peg.class);

{ // Use lists as Stacks:
    pegs.put(A, asLifoQueue(new LinkedList<>()));
    pegs.put(B, asLifoQueue(new LinkedList<>()));
    pegs.put(C, asLifoQueue(new LinkedList<>()));
}
```

# Towers initialization

The constructor takes only one parameter - a number of disks. In makes sense
only for non-negative numbers, so we check that and then initialize the first
peg with disks - smallest on top:

```java
public HanoiTowers(int n) {
    if (n < 0) {
        throw new IllegalArgumentException(
                "Pegs must not be negative. Was: " + n);
    }
    while (n > 0) {
        // Peg A always starts with all the disks:
        this.pegs.get(A).add(n--);
    }
}
```

# The recursive algorithm

The algorithm is simple and consists of three steps:

1.  Move **all but one** disks form peg **A to B**.
2.  Move **the last disk** from peg **A to C**.
3.  Move **all disks** from **B to C**.


# Complete example


## Recursive implementation

```java
package com.farenda.tutorials.algorithms;

import java.util.EnumMap;
import java.util.LinkedList;
import java.util.Queue;

import static com.farenda.tutorials.algorithms.HanoiTowers.Peg.*;
import static java.util.Collections.asLifoQueue;

public class HanoiTowers {

    enum Peg {A, B, C}

    private final EnumMap<Peg, Queue<Integer>> pegs
            = new EnumMap<>(Peg.class);

    { // Use lists as Stacks:
        pegs.put(A, asLifoQueue(new LinkedList<>()));
        pegs.put(B, asLifoQueue(new LinkedList<>()));
        pegs.put(C, asLifoQueue(new LinkedList<>()));
    }

    public HanoiTowers(int n) {
        if (n < 0) {
            throw new IllegalArgumentException(
                    "Pegs must not be negative. Was: " + n);
        }
        while (n > 0) {
            // Peg A always starts with all the disks:
            this.pegs.get(A).add(n--);
        }
    }

    // return content of selected peg
    public Integer[] peg(Peg peg) {
        Queue<Integer> p = pegs.get(peg);
        return p.toArray(new Integer[p.size()]);
    }

    public void solve(Peg from, Peg to) {
        solve(pegs.get(from).size(), from, to);
    }

    private void solve(int n, Peg from, Peg to) {
        if (n >= 1) {
            Peg spare = sparePeg(from, to);
            solve(n-1, from, spare);
            move(from, to);
            solve(n-1, spare, to);
        }
    }

    private Peg sparePeg(Peg from, Peg to) {
        Peg spare = null;
        switch (from) {
            case A: spare = notSelected(to, B, C); break;
            case B: spare = notSelected(to, A, C); break;
            case C: spare = notSelected(to, A, B); break;
        }
        return spare;
    }

    private Peg notSelected(Peg given, Peg a, Peg b) {
        return (given == a) ? b : a;
    }

    private void move(Peg from, Peg to) {
        pegs.get(to).add(pegs.get(from).remove());
    }

    @Override
    public String toString() {
        // helpful for debugging:
        return pegs.toString();
    }
}
```


## JUnit tests

```java
package com.farenda.tutorials.algorithms;

import com.farenda.tutorials.algorithms.HanoiTowers.Peg;
import org.junit.Test;

import static com.farenda.tutorials.algorithms.HanoiTowers.Peg.*;
import static org.junit.Assert.assertArrayEquals;

public class HanoiTowersTest {

    private static final Integer[] EMPTY = {};
    private static final Integer[] ONE_DISK = {1};
    private static final Integer[] THREE_DISKS = {1, 2, 3};
    private static final Integer[] FIVE_DISKS = {1, 2, 3, 4, 5};
    private static final Integer[] TWENTY_DISKS
            = {1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20};

    @Test(expected = IllegalArgumentException.class)
    public void shouldThrowExceptionForNegativePegs() {
        new HanoiTowers(-1);
    }

    @Test
    public void shouldInitializePegsWithGivenNumberOfDisks() {
        HanoiTowers hanoi = new HanoiTowers(0);
        assertPegs(hanoi, EMPTY, EMPTY, EMPTY);

        hanoi = new HanoiTowers(1);
        assertPegs(hanoi, ONE_DISK, EMPTY, EMPTY);

        hanoi = new HanoiTowers(5);
        assertPegs(hanoi, FIVE_DISKS, EMPTY, EMPTY);
    }

    @Test
    public void shouldDoNothingWithEmptyPegs() {
        HanoiTowers hanoi = new HanoiTowers(0);
        hanoi.solve(A, B);
        assertPegs(hanoi, EMPTY, EMPTY, EMPTY);
    }

    @Test
    public void shouldMoveTheOnlyDiskToSelectedPeg() {
        HanoiTowers hanoi = new HanoiTowers(1);
        hanoi.solve(A, B);
        assertPegs(hanoi, EMPTY, ONE_DISK, EMPTY);

        hanoi.solve(B, C);
        assertPegs(hanoi, EMPTY, EMPTY, ONE_DISK);

        hanoi.solve(C, A);
        assertPegs(hanoi, ONE_DISK, EMPTY, EMPTY);
    }

    @Test
    public void shouldMoveAllDisksToSelectedPeg() {
        HanoiTowers hanoi = new HanoiTowers(3);
        hanoi.solve(A, C);
        assertPegs(hanoi, EMPTY, EMPTY, THREE_DISKS);

        hanoi = new HanoiTowers(5);
        hanoi.solve(A, C);
        assertPegs(hanoi, EMPTY, EMPTY, FIVE_DISKS);

        hanoi = new HanoiTowers(20);
        hanoi.solve(A, C);
        assertPegs(hanoi, EMPTY, EMPTY, TWENTY_DISKS);
    }

    private void assertPegs(HanoiTowers hanoi, Integer[] pegA,
                            Integer[] pegB, Integer[] pegC) {
        assertPeg(hanoi, A, pegA);
        assertPeg(hanoi, B, pegB);
        assertPeg(hanoi, C, pegC);
    }

    private void assertPeg(HanoiTowers hanoi, Peg peg,
                           Integer[] actual) {
        assertArrayEquals(hanoi.toString(),
                hanoi.peg(peg), actual);
    }
}
```


# References:

-   How to use [List as Stack](https://farenda.com/java/java-deque-stack-queue)
-   [JUnit Tutorial](https://farenda.com/junit-tutorial) with examples

