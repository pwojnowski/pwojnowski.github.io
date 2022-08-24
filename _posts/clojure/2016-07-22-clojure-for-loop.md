---
title: Clojure for loop
date: 2016-07-22 19:55:00 +0200
categories: [Clojure]
tags: [clojure]
---

In Clojure the *for loop* looks quite differently than in most common
languages. Here we're going to show how to do some interesting things with it
and how it relates to Java versions.

<!--more-->

Clojure *for loop*, actually **list comprehension** (whatever that means ;-) ,
is a **macro** with the following signature:
```clojure
(for [seq-exprs body-expr])
```

It just means that **for** consists of two parts:

1.  **seq-exps**, which is just a bunch of declarations.
2.  **body-expr**, what should be run inside the loop.

The following examples should make it clear. :-)


# Iteration over values
```clojure
(for [i (range 1 6)] i)
;; gives: '(1 2 3 4 5)
```

# Iteration over more values
```clojure
(for [x (range 1 3)
      y (range 6 8)]
  [x y])
;; gives: '([1 6] [1 7] [2 6] [2 7])
```

Think about the order of iteration as nested for loops in Java:
```java
for (int x = 1; x < 3; ++x) {
  for (int y = 6; y < 8; ++y) {
     new int[] {x, y};
  }
}
```

# For loop in Clojure allows to filter values

Unlike in Java, but Similarly to Python's *for-loop*, in Clojure we can filter
the values. The **:when** clause serves exactly that purpose:
```clojure
(for [i (range 1 11) :when (even? i)]
  i)
;; gives: '(2 4 6 8 10)
```

It corresponds to the following code in Java:
```java
for (int i = 1; i < 11; ++i) {
  if (isEven(i)) {
     numbers.add(i);
  }
}
```


# Iterate as long as invariant holds
```clojure
(for [i (range 1 11) :while (<= i 5)] i)
;; gives: '(1 2 3 4 5)
```

# Create new vars in scope

You can use **:let clause** that behaves like **let form** and allows to create
local variables:
```clojure
(for [i (range 1 6) :let [squared (* i i)]]
  [i squared])
;; gives: '([1 1] [2 4] [3 9] [4 16] [5 25])
```

Java counterpart:
```java
for (int i = 1; i < 6; ++i) {
   int squared = i * i;
   // How to easily return pair in Java? :-)
}
```

# Combination of filtering and conditional processing

Clauses inside **for-loop** can be used together. Here's a sample filtering with
checking of invariant:
```clojure
(for [i (range 1 100)
      :while (< i 10)
      :when (odd? i)]
  i)
;; gives: '(1 3 5 7 9)
```

# Combination of when, while, and let
```clojure
(for [i (range 1 100) ;; lazy sequence of 100 numbers
      :while (< i 10) ;; limit above to only first 10
      :when (even? i) ;; use only even numbers
      :let [tripple (* i i i)]] ;; var for readability
  tripple)
;; gives: '(8 64 216 512)
```


# The complete test that you can run:
```clojure
(deftest test-for-examples
  (testing "Simple iteration over values"
    (is (= (for [i (range 1 6)] i)
           '(1 2 3 4 5))))

  (testing "Iteration over two vars"
    (is (= (for [x (range 1 3) y (range 6 8)] [x y])
           '([1 6] [1 7] [2 6] [2 7]))))

  (testing "With filtering"
    (is (= (for [i (range 1 11) :when (even? i)] i)
           '(2 4 6 8 10))))

  (testing "As long as condition is met"
    (is (= (for [i (range 1 11) :while (<= i 5)] i)
           '(1 2 3 4 5))))

  (testing "With new vars based on iterated value"
    (is (= (for [i (range 1 6) :let [squared (* i i)]]
             [i squared])
           '([1 1] [2 4] [3 9] [4 16] [5 25]))))

  (testing "when then while"
    (is (= (for [i (range 1 100)
                 :when (odd? i) :while (< i 10)]
             i)
           '(1 3 5 7 9))))

  (testing "Combo"
    (is (= (for [i (range 1 100)
                 :while (< i 10)
                 :when (zero? (rem i 2))
                 :let [tripple (* i i i)]]
             tripple)
           '(8 64 216 512)))))
```

# References:
- "Programming Clojure" by Stuart Halloway and Aaron Bedra
