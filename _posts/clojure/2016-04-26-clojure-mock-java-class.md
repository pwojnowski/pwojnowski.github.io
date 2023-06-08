---
title: How to mock Java classes in Clojure
date: 2016-04-26 07:01:00 +0200
tags: [clojure]
---

In this post we're going to show how to use **Clojure proxy** to mock **Java**
classes and provide own functionality for Java methods.

<!--more-->

# Proxy class with no-arg constructor

Calls superclass default constructor:
```clojure
(def my-list
  (proxy [ArrayList] []
    (get [i] 42)))

;; Let's use it:
;; user> (.get my-list 10)
;; 42
```

# No matching ctor found for class

Superclass has no no-arg constructor:
```clojure
(proxy [FileInputStream] []
        (read []
          42))
```

Produces an exception, because there's no no-arg constructor in
**FileInputStream** class:

    CompilerException java.lang.IllegalArgumentException: No matching ctor found for class user.proxy$java.io.FileInputStream$ff19274a


# Proxy java class with constructor

We have to provide obligatory constructor parameter (here "/proc/version"), that
will be passed to superclass:
```clojure
;; To count calls and return predictive values:
(def counter (atom 0))

(def my-mock
  (proxy [FileInputStream] ["/proc/version"]
    (read []
      (swap! counter inc))))
```

Now our version of **read** method may be called as expected:

    user> (.read my-mock)
    1
    user> (.read my-mock)
    2
    user> (.read my-mock)
    3


# Overload mocked method
```clojure
(def counter (atom 0))

(def my-mock
  (proxy [FileInputStream] ["/proc/version"]
    (read
      ([] (swap! counter inc)) ; read()
      ([^bytes arr]            ; read(byte[])
       (aset-byte arr 0 42)
       (swap! counter inc)))))
```

**read(byte[])** can be called like this:

    user> (.read my-mock (byte-array 3))
    1
    user> (.read my-mock (byte-array 3))
    2


# References:

- [proxy macro](https://clojure.github.io/clojure/clojure.core-api.html#clojure.core/proxy)
- [ClojureDocs - proxy](https://clojuredocs.org/clojure.core/proxy)
