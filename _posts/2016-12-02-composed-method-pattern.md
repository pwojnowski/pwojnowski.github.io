---
title: Composed Method Pattern explained
date: 2016-12-02 23:10:00 +0200
tags: [software-design,java,clojure]
---


**Composed Method Pattern** is the most useful and practical pattern I use and
at the same time it's not known to many developers. It's surprising, because the
pattern is the foundation of maintainable code.

<!--more-->

Almost every code review I do contains the sentence "Refactor this to Composed
Method". The *Composed Method Pattern* is easy and fast to implement, with
immediate benefits.


# Purpose of Composed Method

    You cannot immediately understand method's logic.

In corporate word it's a standard. There is plenty of low quality code that
have to be maintained, extended, etc. But how to change anything, when the
business logic is a total mess? Sounds familiar? Here enters the *Composed
Method* and the pattern can be summarized in one sentence:

    Composed Method literally turns shit into honey.


# How to introduce

Apply *Extract Method* refactoring on *logically coherent* blocks of code until
all are in separate methods, where names of the methods tell what they do (not
*how*).


## Single Level Of Abstraction (SLA Principle)

All steps of processing in a method should be at the same level of
abstraction. In other words, they should not expose their details, but move
the details to separate methods that give them *intent-revealing names*. The
parts of a method should tell what is being done, not how.


# Composed Method in Java

In the below code example there's a method that calculates something. It takes
parameters: a scores as a String, numbers of scores in the String, and how many
of them should be taken into account.


## Bofore Composed Method
```java
private int sumBest(String data, int size, int howMany) {
    Scanner scanner = new Scanner(data);
    int[] scores = new int[size];
    for (int i = 0; i < size; ++i) {
        scores[i] = scanner.nextInt();
    }
    Arrays.sort(scores);
    int sum = 0;
    for (int i = scores.length-howMany; i < scores.length; ++i) {
        sum += scores[i];
    }
    return sum;
}
```

Even though the code is simple, it's not immediately obvious how it works and
what it does. You have to dig into details to pick out *steps of processing* it
does and remember them for further code analysis - just recall a real business
code with much longer and more complicated methods.

The above method can be called like this:
```java
int sum = sumBestBefore("1 2 3 4 5 6 7 8 9 10", 10, 3);
System.out.println("Sum of best 3: " + sum);
// prints: Sum of best 3: 27
```


## Refactored to Composed Method

Here we've got the same code, but refactored to *Composed Method* making **each
step explicit**, thus the whole code is much more readable:
```java
private int sumBest(String data, int size, int howMany) {
    int[] scores = loadScores(data, size);
    sort(scores);
    return sumLast(scores, howMany);
}

private int[] loadScores(String data, int size) {
    Scanner scanner = new Scanner(data);
    int[] numbers = new int[size];
    for (int i = 0; i < size; ++i) {
        numbers[i] = scanner.nextInt();
    }
    return numbers;
}

private int sumLast(int[] scores, int howMany) {
    int sum = 0;
    for (int i = scores.length-howMany; i < scores.length; ++i) {
        sum += scores[i];
    }
    return sum;
}
```

Now it's clearly visible that whole processing consists of three steps:

1.  Loading scores from given data string.
2.  Sorting the scores.
3.  Summing the last (biggest) scores.

The `sumBest()` method does whole processing at the single level of abstraction -
loads scores, sorts them, and calculates sums. It operates on scores. How it
loads them? Who cares!? At this level it's not important. If someone want's to
know, they can always go to the `loadScores()` method.

The same with other steps - is it important, for understanding `sumBest` what
kind of sorting is used? Of course not! It's just a detail here, so we hid it.

Also, note how small and easy to understand the methods are. The details of each
step can be understand in no time!


# Composed Method in functional languages

Although the *Composed Method Pattern* is known from Object-Oriented
Programming, it is equally well applicable in *Functional Programming*. Here we
can name it **Composed Function**. :-)


## Before Composed Function

The following Clojure version does the same thing as the Java code above. Again,
to understand what it does you have to dig into unimportant details:
```clojure
(defn sum-best [data size how-many]
  (let [scanner (Scanner. data)]
    (->> (for [i (range 0 size)] (.nextInt scanner))
         (sort (java.util.Collections/reverseOrder))
         (take how-many)
         (apply +))))
```


## Refactored to Composed Function

Here is the same code logic, but with readable, explicit processing steps -
*Composed Function*:
```clojure
(defn load-scores [data size]
  (let [scanner (Scanner. data)]
    (for [i (range 0 size)] (.nextInt scanner))))

(defn sort-desc [scores]
  (sort (java.util.Collections/reverseOrder) scores))

(defn sum-first [how-many scores]
  (apply + (take how-many scores)))

(defn sum-best
  [data size how-many]
  (->> (load-scores data size)
       sort-desc
       (sum-first how-many)))
```


# Properties of code refactored to Composed Method


## Benefits

1.  Makes the code immediately understandable
    Extracted methods have names that immediately tell what they do (**intent
    revealing names**), without having to analyze ton of code.
2.  Helps to find duplicated code.
3.  Helps to identify responsibilities.
4.  Makes unit testing easy.


## Drawbacks

1.  Creates more small methods.
    From my experience it is not a problem, because duplicated code is unified
    and methods with other responsibilities are moved to their own or correct
    classes.
2.  May make the debugging more difficult, because the logic is spread in
    different methods.
    There is more methods and classes, however they are smaller, easier to
    understand, and reason about.


# References

-   "*Refactoring to Design Patterns*" by *Joshua Kerievsky*
