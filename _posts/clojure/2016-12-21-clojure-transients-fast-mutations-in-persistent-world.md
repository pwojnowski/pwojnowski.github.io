---
title: Clojure transients - fast mutations in persistent world
date: 2016-12-21 07:01:00 +0200
tags: [clojure]
---


Clojure transients is a nice way to optimize performance of sensitive code
without leaving familiar Clojure world. In this post we show how to use them to
boost performance.

<!--more-->


# Basic flow with transients

The code structure is the same as in any Clojure code, except turning mutations
on using transient function, modifications using **bang!** versions of 
functions, and persisting back with `persistent!` function:
```clojure
(defn transient-flow [v]
  (let [tv (transient v)]
    (-> tv
        ;; mutate with bang! functions
        persistent!)))
```

Now you can call it like any other Clojure code:
```clojure
(is (= [1 2 3] (transient-flow [1 2 3])))
```


# Transient mutation functions

Like their immutable versions *transients* support read-only operations with
standard functions (`count`, `get`, etc.), but for mutations they have their own
versions, which we will cover here.


## conj! - add elements to a transient collection
```clojure
(defn conj-inline [coll val]
  (let [tv (transient coll)]
    (persistent! (conj! tv val))))
```

The test:
```clojure
(is (= [1 2 3 42] (conj-inline [1 2 3] 42)))
```


## assoc! - create/replace mapping for a key

Here we just replace each value with its multiplication by *n*:
```clojure
(defn times [v n]
  (loop [tv (transient v) i (dec (count v))]
    (if (<= 0 i)
      (recur (assoc! tv i (* n (get tv i))) (dec i))
      (persistent! tv))))
```

And test:
```clojure
(is (= [2 4 6] (times [1 2 3] 2)))
```


## dissoc! - remove mapping for a key
```clojure
(defn remove-key [m key]
  (-> m transient (dissoc! key) persistent!))
```

Let's run it:
```clojure
(is (= {:points 1234}
       (remove-key {:points 1234 :level 4} :level)))
```


## pop! - remove last from a vector
```clojure
(defn remove-last [v]
  (-> v transient pop! persistent!))
```

Pop from stack seems to work:
```clojure
(is (= [1 2] (remove-last [1 2 3])))
```


## disj! - remove last from a set
```clojure
(defn remove-from-set [s key]
  (-> s transient (disj! key) persistent!))
```

The test:
```clojure
(is (= #{1 3} (remove-from-set #{1 2 3} 2)))
```


# Transient types - vectors, maps, sets

Only *persistent vectors*, *hash maps*, and *hash sets* can be transient. When
you try to turn an unsupported type into a *transient* then `ClassCastException`
will be thrown:
```clojure
(defn transient-list [l]
  (persistent! (transient l)))
```

The test:
```clojure
(deftest should-throw-exception-for-non-transientable-types
  (try
    (transient-list (range 10))
    (throw (IllegalStateException. "Should fail!"))
    (catch ClassCastException e
      (prn "List cannot be transient: " (.getMessage e)))))
```


# Properties of transients

There are a few things that it's good to keep in mind when using *transients*:

- creating and persisting a transient take **O(1)** time each
- they require *thread isolation*, because they share mutable state between operations.


# References:

- [How to use transients to implement Insertion Sort](https://farenda.com/clojure/insertion-sort-in-clojure)
- Check out other [Clojure Articles](https://farenda.com/clojure-tutorial)!
- The code is available at [GitHub](https://github.com/pwojnowski/clojure-playground)

