---
title: Prime numbers - Sieve of Eratosthenes in Java
date: 2016-09-02 13:40:00 +0200
tags: [algorithms,java]
---


A prime number is a natural number with only two divisors: 1 and itself. In this
post we're going to show how to find prime numbers using **Sieve of
Eratosthenes** and explain how it works.

<!--more-->


# How it works

1.  Create a boolean array for the numbers you want to verify.
    We create an array with size greater by one to comfortably use index as
    underlying number.
2.  Initially mark all numbers in the array as primes.
3.  Start sieving from the `smallest prime number - 2`.
4.  Mark as non primes all multiplies of the current prime.
    Notice that we start the sieve from `i * i`. This is because all numbers
    below have been already sieved. For example, when removing multiplies of 5
    we can start from 25, because all lower multiplies of 5 have been removed
    when 2 and 3 were processed - `5*2 = 2*5` and `5*3 = 3*5`.
5.  Repeat with the next prime.
    We don't have to look for primes above `maxPrime`, because of reasoning in
    point 4.

```java
void markComplexNumbers(boolean[] primes) {
    primes[0] = primes[1] = false;
    int maxPrime = (int) Math.sqrt(primes.length);
    for (int i = 2; i <= maxPrime; ++i) {
        if (primes[i]) {
            // remove all multiplies of this prime:
            for (int j = i*i; j < primes.length; j += i) {
                primes[j] = false;
            }
        }
    }
}
```


# Properties of the Sieve of Eratosthenes

- time complexity is **O(n log log n)**
- memory complexity is **O(n+1)**
- works great for small (in mathematical sense ;-) ) prime numbers


# Complete example
```java
import java.util.Arrays;

public class Sieve {

    public static void main(String[] args) {
        boolean[] primes = findPrimes(101);
        printPrimes(primes);
    }

    public static boolean[] findPrimes(int to) {
        // +1 to have index and number with the same value:
        boolean[] primes = new boolean[to+1];
        Arrays.fill(primes, true);
        markComplexNumbers(primes);
        return primes;
    }

    private static void markComplexNumbers(boolean[] primes) {
        primes[0] = primes[1] = false;
        int maxPrime = (int) Math.sqrt(primes.length);
        for (int i = 2; i <= maxPrime; ++i) {
            if (primes[i]) {
                // remove all multiplies of this prime:
                for (int j = i*i; j < primes.length; j += i) {
                    primes[j] = false;
                }
            }
        }
    }

    private static void printPrimes(boolean[] primes) {
        for (int i = 2; i < primes.length; ++i) {
            if (primes[i]) {
                System.out.print(i + ", ");
            }
        }
    }
}
```

The above code produces the following output:

    2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47,
    53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101,
