---
title: Java volatile and atomic operations
date: 2015-11-23 07:01:00 +0200
tags: [java,concurrency]
---

How to use **Java volatile** and **atomic** operations to write lock-free code?
These are two fundamental concepts in **Java concurrency** and understanding
both of them is required to write correct, thread-safe code.

<!--more-->

## Atomic

**Atomic operation** in **Java** is an operation that is guaranteed to **not be
interrupted** in between by the thread scheduler. The following example shows
an assignment that may be **interrupted** by the **Thread Scheduler** to execute
other threads:

```java
public class NotAtomicOperations {

   private long number;

   public void setNumber(long newNumber) {
       // This assignment may be split into two 32-bit operations,
       // between which Thread Scheduler may execute other thread!
       number = newNumber;
   }
}
```

What operations are atomic? Basically only read and write operations of:

-   all **basic types except 64-bit types**, namely **long** and **double**,
-   **references variables** (no matter whether it is 32-bit or 64-bit JVM),
-   **all variables** declared as **volatile** (including long and double).


**Note:** *only read and write* means that, for example, incrementation (**++
operator**) is not atomic and you cannot compare and set an int as one atomic
operation.


## volatile keyword

Even though some operations are **atomic**, it doesn't mean that their results
will be immediately visible by all threads on **multiprocessor** or **multi-core**
machines (which are standard these days). JVM doesn't guarantee that and such
results may be stored in a **processor's cache** to which, threads running on
other processors, don't have access. The purpose of **volatile** keyword is to
make those results **visible in main memory**, hence **for all threads**, like so:

```java
public class VolatileVisibleToAllThreads {

   // volatile makes it visible to all threads!
   private volatile boolean running = true;

   public void stop() {
       // Simple write is atomic:
       running = false;
   }

   public boolean doWork() {
       // Will see the result of stop() called by other thread:
       while (running) {
           // doing some work...
       }
   }
}
```


**Note:** It is important to remember that **volatile** keyword can be used with
only one variable. It doesn't make sense to use it with more that one, because
a thread can do only one atomic operation, without synchronization. So the
following code won't work correctly:

```java
public class DoubleVolatileNotAtomic {

   private volatile int number1;
   private volatile int number2;

   public boolean setNumbers(int a, int b) {
       // Thread Scheduler may interrupt in between:
       number1 = a;
       number2 = b;
   }
}
```


Also adding **volatile** to **long** and **double** makes them atomic too:

```java
public class VolatileWithAtomicLong {

   private volatile long number;

   public void setNumber(long newNumber) {
       // This assignment won't will be atomic thanks to volatile:
       number = newNumber;
   }
}
```


## synchronized

In previous posts (e.g. [synchronized method](https://farenda.com/java/java-synchronized-method) and [synchronized object](https://farenda.com/java/java-synchronized-object)) we've
shown how to use **synchronized** keyword to make the code **thread-safe**. One
nice feature of **synchronized** is that it makes operations on variables
**atomic** and **visible** to other threads at the same time. What's more, it
allows to do **atomic operations** on **more than one variable**.


To summarize: atomic variables allow to write **lock-free** code, but its very
easy to do it incorrectly, therefore **synchronized** is preferred in most cases.
