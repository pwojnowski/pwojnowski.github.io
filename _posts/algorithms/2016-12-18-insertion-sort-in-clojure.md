---
title: Insertion Sort in Clojure
date: 2016-12-18 17:12:00 +0200
tags: [clojure,algorithms]
---

Insertion sort in Clojure can be implemented in different ways. In this post we
compare implementations with and without Clojure *transients*.

<!--more-->


# Unit Test

Sometimes, after writing a test, it turns out that everything works and there's
nothing to implement. So, let's start with a test and see if that's the case
here:
```clojure
(ns poligon.algorithms.sorting-test
  (:require [poligon.algorithms.sorting :refer :all]
            [clojure.test :refer :all]))

;; random 10.000 numbers:
(def unsorted-data
  (vec (doall (repeatedly 10000 #(rand-int 10000)))))

;; expected result:
(def sorted-data (sort unsorted-data))

(deftest insertion-sort-test
  (is (= [] (insertion-sort [])))
  (is (= [1 2 3] (insertion-sort [1 2 3])))
  (is (= [1 2 3 4 5] (insertion-sort [5 2 3 4 1])))
  ;; transients version:
  (is (= sorted-data (time (insertion-sort unsorted-data))))
  ;; persistent vector version:
  (is (= sorted-data (time (insertion-sort-simple unsorted-data)))))
```


# Insertion sort

First let's implement the algorithm. The flow is simple: take next element and
insert into correct position in already sorted vector. In this version we just
reorder elements of *persistent vector* as we usually do in Clojure:
```clojure
(defn- insert-simple
  [tv idx]
  (let [current-value (get tv idx)]
    (loop [i idx v tv]
      (let [left-value (get v (dec i))]
        (if (and (pos? i)
                 (> left-value current-value))
          (recur (dec i) (assoc v i left-value))
          (assoc v i current-value))))))

(defn insertion-sort-simple
  [v]
  (let [size (-> v count dec)]
    (loop [i 1 tv v]
      (if (<= i size)
        (recur (inc i) (insert-simple tv i))
        tv))))
```

After running the tests we can see that this version works. But here we have
overhead of handling of persistent vector. If we would like to change only a few
values it wouldn't matter much, but we cannot assume that. Insertion sort, as
many other algorithms, may reorder many elements. In such cases we can resort to
*transients* and *mutate local data structure*.


# Transients for performance

A nice thing about *transients* is that the code structure is almost the same as
we normally write in Clojure. So we can develop an algorithm and then introduce
*transients*. The only differences are:

1. Conversion of persistent data structure to transient using `transient` function.
2. Mutation using *bang* versions of the functions (`assoc!`, `dissoc!`, etc.)
3. Turning transient back into persistent structure using `persistent!`.

In case of *insertion sort* we only use `transient`, `assoc!`, and
`persistent!`. The rest of the code stays the same:

```clojure
(ns poligon.algorithms.sorting)

(defn- insert
  "Insert element from `idx` into correct position in transient vector `tv`."
  [tv idx]
  (let [current-value (get tv idx)]
    (loop [i idx v tv]
      (let [left-value (get v (dec i))]
        (if (and (pos? i)
                 (> left-value current-value))
          (recur (dec i) (assoc! v i left-value))
          (assoc! v i current-value))))))

(defn insertion-sort
  "Insertion sort using transients."
  [v]
  (let [size (-> v count dec)]
    (loop [i 1 tv (transient v)]
      (if (<= i size)
        (recur (inc i) (insert tv i))
        (persistent! tv)))))
```


# Difference in performance

After running the tests a couple of times, the run-times were pretty much the
same as here. The *transients version* is about 3 times faster:

    lein test :only poligon.algorithms.sorting-test
    
    lein test poligon.algorithms.sorting-test
    "Elapsed time: 8855.431509 msecs"
    "Elapsed time: 24010.397826 msecs"
    
    Ran 2 tests containing 6 assertions.
    0 failures, 0 errors.

Of course the numbers way be different for different data. But it's pretty
clear that *transients* increase performance without complicating code much.


# References:

- [Insertion Sort explained](https://farenda.com/algorithms/insertion-sort-in-java)
- Check out other cool [Clojure articles](https://farenda.com/clojure-tutorial)!
- The code is available at [GitHub](https://github.com/pwojnowski/clojure-playground)
