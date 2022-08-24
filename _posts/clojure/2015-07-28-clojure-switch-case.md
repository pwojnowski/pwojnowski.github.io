---
title: Clojure switch-case
date: 2015-07-28 07:01:00 +0200
categories: [Clojure]
tags: [clojure]
---


One of the first control flow statements in many programming languages is
**switch-case**. In Clojure switch-case can be achieved in different ways, what
we will show in this article.

<!--more-->


# The case macro

There are a couple of functions/macros in **Clojure** that serve as
**switch-case** control flow statements. The first one is `case` macro:
```clojure
(defn case-example [word]
  (case word
    "saluton" "Esperanto!"
    "hi" "English"
    "Unknown"))
```

It takes an expression (here `word` evaluates to its value), tries to match
its result with constant in the first column and returns value from the right
column when it matches.

A **default value** can be provided as a single value, without test
expression. If default value is not provided and there is no match then
`IllegalArgumentException` is thrown.

Let's test that:
```clojure
(deftest test-case-examples
  (testing "case example"
    (is (= "Unknown" (case-example "aoeu")))
    (is (= "English" (case-example "hi")))
    (is (= "Esperanto!" (case-example "saluton")))))
```


# The cond macro

The next approach to switch-case in Clojure is `cond` macro:
```clojure
(defn cond-example [p]
  (cond
    (< (count p) 8) "too short"
    (< 32 (count p)) "too long"
    :else "ok"))
```

It takes a number of pairs **test expression** and a **value**. When test
expression matches, then the value is returned. The last expression is `:else`,
which, as other keywords, evaluates to itself. It could be anything which
evaluates itself to boolean true, however, `:else` is commonly used for that
purpose.

When there is no default, the `case` returns `nil`.

Running the code gives:
```clojure
(deftest test-cond-examples
  (testing "cond example"
    (is (= (cond-example "abc") "too short"))
    (is (= (cond-example "very long string") "too long"))
    (is (nil? (cond-example "ok one")))))
```


# The condp macro

The last one is `condp`:
```clojure
(defn condp-example [op a b]
  (condp = op
    + (+ a b)
    - (- a b)
    "Unknown operator"))
```

Clojure `condp` macro takes a binary predicate (here `=`) and an expression
(here `op` from the parameters) and a number of pairs `test-expression
result-expression`. For each clause the following expression is evaluated:
`(pred test-expr expr)`. So, for arguments `'+ 1 2` it would be `(= '+
'+)`. When the `test expression` returns true then the `result expression` is
evaluated and returned.

`Default` value is indicated by the line *without separate test expression*.

When no default value is provided, `condp` will throw `IllegalArgumentException`
if nothing matches.

Running the example gives:
```clojure
(deftest test-condp-examples
  (testing "condp example"
    (is (= (condp-example + 1 2) 3))
    (is (= (condp-example - 1 2) -1))
    (is (= (condp-example * 1 2) "Unknown operator"))))
```


# What next?

Now switching from Java switch-case to Clojure switch-case should be a breeze. ;-)

Check out other [Clojure articles](https://farenda.com/clojure-tutorial)!

