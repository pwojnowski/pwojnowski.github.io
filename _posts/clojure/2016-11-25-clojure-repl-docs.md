---
title: Clojure REPL docs
date: 2016-11-25 17:05:00 +0200
categories: [Clojure]
tags: [clojure]
---

In this post we show how to read docs in **Clojure REPL**. You'll learn useful
commands to find and display Clojure and Java docs, how to display Clojure
source code.

<!--more-->


# Print documentation

To print documentation of a **function**, **var**, **namespace**, **macro**,
etc. just use the **doc** macro - `(doc var\_or\_special\_form)`:

    user> (doc doc)
    -------------------------
    clojure.repl/doc
    ([name])
    Macro
      Prints documentation for a var or special form given its name
    
    user> (doc clojure.repl)
    -------------------------
    clojure.repl
      Utilities meant to be used interactively at the REPL


# Find docs matching patterns

To find **any documentation by regexp** use `(find-doc "RegExp")`:

    user> (find-doc "red.cer")
    -------------------------
    clojure.core.reducers/->Cat
    ([cnt left right])
      Positional factory function for class clojure.core.reducers.Cat.
    -------------------------
    clojure.core.reducers/reducer
    ([coll xf])
      Given a reducible collection, and a transformation function xf,
      returns a reducible collection, where any supplied reducing
      fn will be transformed by xf. xf is a function of reducing fn to
      reducing fn.
    -------------------------
    clojure.core.reducers
      A library for reduction and parallel folding. Alpha and subject
          to change.  Note that fold and its derivatives require Java 7+ or
          Java 6 + jsr166y.jar for fork/join support. See Clojure's pom.xml for the
          dependency info.


# Display source

One of the most useful things - `(source symbol)`:

    user> (source identity)
    (defn identity
      "Returns its argument."
      {:added "1.0"
       :static true}
      [x] x)


# Display JavaDoc

To display **API docs of Java** classes in default browser - `(javadoc object-or-class)`:

    ;; opens JavaDoc for String class in default browser
    user> (javadoc String)
    
    ;; The same as above, because `javadoc` will take class
    ;; of the given object and then proceed in the same way:
    user> (javadoc "aoeu")

By default it opens remote **Oracle's JDK** page for given class/object.

However, if you would like to display Javadoc locally you will need to add its
location:

    ;; Add local doc path if you have:
    user> (clojure.java.javadoc/add-local-javadoc "/home/przemek/doc/java/jdk-8-api/api/")

Now `(javadoc Thread)` will display local page for given class or will fallback
to remote site if it won't find it locally (usually because a wrong path was
defined).


# Clojure REPL helpers in other namespaces

When you start **Clojure REPL** and begin within **user namespace** you will
have the following functions at your disposal: **doc**, **find-doc**,
**source**, and **javadoc**. But when you switch to other namespace and try to
use them you will get an exception:

    myns> (doc clojure.repl/apropos)
    CompilerException java.lang.RuntimeException: Unable to resolve symbol: doc in this context, compiling:(*cider-repl localhost*:134:16)

To fix that you have to **import the functions** by yourself:

    myns> (use '[clojure.repl :only (dir doc find-doc source)])
    myns> (doc clojure.repl/apropos)
    -------------------------
    clojure.repl/apropos
    ([str-or-pattern])
      Given a regular expression or stringable thing, return a seq of all
    public definitions in all currently-loaded namespaces that match the
    str-or-pattern.


# Inject repl tools automatically

If you are using **Leiningen** then you can inject helper functions automatically
from your `~/.lein/profiles.clj` file:

```clojure
{:repl {:injections [(use '[clojure.repl :only [doc find-doc source]])]}}
```


# References:

- Check out other parts of [Clojure Tutorial](https://farenda.com/clojure-tutorial)!
- [Clojure reference](https://clojure.org/reference/repl_and_main)
