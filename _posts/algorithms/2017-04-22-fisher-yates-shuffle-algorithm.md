---
title: Fisher-Yates Shuffle Algorithm
date: 2017-04-22 18:16:00 +0200
tags: [algorithms,java]
---


Shuffling of collection of items sounds like a trivial task, but in reality
there are subtle traps. In this post we're going to learn how to shuffle things
using unbiased **Fisher-Yates Shuffle Algorithm** and how to do that in
different languages.

<!--more-->


# Shuffling traps

There are some things to be aware when implementing/choosing a *shuffling algorithm*:

- Use good **Pseudo-Random Number Generator**, one that uniformly distributes
  random numbers;
- Shuffling algorithm should produce **each permutation with equal probability**;
  That's why correct range for random numbers is important. Taking wrong range
  may produce biased results, which is illustrated nicely at [Data Genetics](https://datagenetics.com/blog/november42014/)
  (see the Monte Carlo simulation at the bottom).


# Fisher-Yates Shuffle

The most known and optimal shuffling algorithm is **Fisher-Yates shuffle**. The 
algorithm is very easy to implement and produces **unbiased results**. It has
been modernized by Durstenfeld and popularized by Donald Knuth in *The Art of
Computer Programming* TV series. The Durstenfeld's implementation of the
algorithm is extremely simple and boils down to just a few lines of code. What's
important, it runs in **linear time** O(n) and works **in place**:

```java
// T[] data, Random rand
for (int i = data.length-1; i > 0; --i) {
    swap(data, i, rand.nextInt(i+1));
}
```


## How Durstenfeld implementation works

1.  Go from the end of the list (you can process in the oposite direction
    by just changing directions in all the steps).
2.  Pick random number between 0 and the current position (both inclusive).
3.  Exchange the current item with the random one.


## Complete Java example

```java
import java.util.Arrays;
import java.util.List;
import java.util.Random;

import static java.util.Collections.swap;

public class RandomSort {

    public static void main(String[] args) {
        // Data to sort is a list of 1-char Strings:
        List<String> items = Arrays.asList("HelloShuffleWorld!".split(""));

        System.out.println("Before shuffle: " + items);

        shuffle(items, new Random());

        System.out.println("Shuffled: " + items);
    }

    private static <T> void shuffle(List<T> items, Random rand) {
        // going from the last item to the second
        for (int i = items.size() - 1; i > 0; --i) {
            // swap the current item with random from 0 to the current (inclusive)
            swap(items, i, rand.nextInt(i + 1));
        }
    }
}
```

To swap elements of the array we just reuse `Collections.swap(List, i, j)`. If
you happen to know why there is no `swap()` method in `java.util.Arrays`, please
let me know in the comments! This is just incredible.

The above code produces shuffled output:

    Before shuffle: [H, e, l, l, o, S, h, u, f, f, l, e, W, o, r, l, d, !]
    Shuffled: [r, W, l, l, h, o, f, d, e, l, e, f, H, o, !, u, S, l]

Now, you are ready to implement *card shuffling* in you games. :-)


## Shuffling in JVM world
In Java and other JVM-based languages, the shuffling can be done using two
methods from `java.util.Collections` class:
  - `shuffle(List)`
  - `shuffle(List, Random)`

The `shuffle(List)` just calls the other method with `new Random()` as the
randomness source, therefore both use exactly the same algorithm. This
implementation works in basically the same way, just with optimizations for
swapping elements adjusted for types of the list.

The usage is straightforward:

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.Random;

public class CollectionsShuffle {

    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        System.out.println("Numbers: " + numbers);

        Collections.shuffle(numbers);
        System.out.println("Shuffled: " + numbers);

        Collections.shuffle(numbers);
        System.out.println("Shuffled again: " + numbers);

        Collections.shuffle(numbers, new Random());
        System.out.println("Shuffled with Random: " + numbers);
    }
}
```

To shuffle an array just convert it to list using `Arrays.asList()` (or
`Arrays.stream(arr).boxed().toList()` or whatever method works for you) and then
call `Collections.shuffle(List)` as in the example.

The above code produces the following output:

    Numbers: [1, 2, 3, 4, 5]
    Shuffled: [1, 3, 4, 5, 2]
    Shuffled again: [2, 1, 3, 5, 4]
    Shuffled with Random: [1, 2, 4, 3, 5]


# References

- [Fisher-Yates shuffle in Wikipedia](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle)
- [Biased and unbiased shuffling visualization](https://datagenetics.com/blog/november42014/) at DataGenetics

