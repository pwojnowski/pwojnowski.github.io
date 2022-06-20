---
title: Caesar cipher in Java
date: 2017-07-21 22:00:00 +0200
categories: [Algorithms]
tags: [java,cryptography]
---


One of the simplest cryptographic algorithms is **Caesar cipher**. It's not really
secure, but sometimes you may encounter it in some exercises or crackmes.

<!--more-->


# Intro

The cipher method is named after Julius Caesar, who presumably have used it to
encrypt his military messages (see the history section at [Wikipedia](https://en.wikipedia.org/wiki/Caesar_cipher)). Maybe at
that time it was the state of the art. Anyway, the algorithm is good to know
in case you happen to find yourself in the middle of war against the Roman
Empire and will need to decrypt their messages to win the war! :wink:


# The algorithm

The steps of the the **Caesar cipher** algorithm are as follows:

1.  Take each character of given message.
2.  Replace the char with the one on position distant by given number of
    chars. If the position is outside the alphabet, we are looking for the
    position from the beginning of the alphabet, thus creating the loop: a, b,
    &#x2026;, y, z, a, b, &#x2026;, y, z, &#x2026;
3.  If we want to decipher the message we just need to rotate the encrypted
    message by the same number of chars, but into the opposite direction.

Just look at the unit tests and it will be clear how it works.

So, the cipher takes as an input two parameters:

1.  The message to encrypt.
2.  The size of the rotation.

```java
public String cipher(String message, int rotateBy) {
    // rotate by only the size of the alphabet:
    rotateBy %= ALPHABET_SIZE;
    char[] chars = message.toCharArray();
    rotate(chars, rotateBy);
    return new String(chars);
}
```

## Message traversal

The first step is to go through all chars in the message and substitute it
with its rotated counterpart. In our case we want to rotate only the alphabet
characters. The thing here is *ASCII* codes for lower case chars and upper case
chars are in different numeric ranges. Therefore we have to check the limits
of the two alphabets - lower and upper case.

```java
private void rotate(char[] chars, int rotateBy) {
    for (int i = 0; i < chars.length; ++i) {
        if (isLowerCase(chars[i])) {
            chars[i] = rotateChar(chars[i], rotateBy, 'a', 'z');
        } else if (isUpperCase(chars[i])) {
            chars[i] = rotateChar(chars[i], rotateBy, 'A', 'Z');
        }
    }
}
```


## The rotate char function

The key function in the **Caesar's cipher** is the rotation function, which will
rotate a single character by given number of chars. Keep in mind that the
number may be positive or negative, so we can go off the alphabet in both
directions. When we go off the alphabet, we just create a loop and find the
correct char from the opposite side by moving by the alphabet size (*do you
know why it works?*):

```java
private char rotateChar(char c, int rotateBy,
                        char firstChar, char lastChar) {
    c += rotateBy;
    // check lower bounds for left rotation:
    if (c < firstChar) {
        return (char) (c + ALPHABET_SIZE);
    }
    // check upper bounds for right rotation:
    if (c > lastChar) {
        return (char) (c - ALPHABET_SIZE);
    }
    // this one is within alphabet bounds:
    return c;
}
```

# Complete Caesar cipher in Java

Here's the complete **Caesar cipher in Java**, ready to encrypt your emails or
Julius Caesar's military messages! ;-)

```java
package com.farenda.tutorials.algorithms.strings;

import java.util.Scanner;

import static java.lang.Character.isLowerCase;
import static java.lang.Character.isUpperCase;

public class CaesarCipher {

    private static final int ALPHABET_SIZE = 26;

    public String cipher(String message, int rotateBy) {
        // rotate by only the size of the alphabet:
        rotateBy %= ALPHABET_SIZE;
        char[] chars = message.toCharArray();
        rotate(chars, rotateBy);
        return new String(chars);
    }

    private void rotate(char[] chars, int rotateBy) {
        for (int i = 0; i < chars.length; ++i) {
            if (isLowerCase(chars[i])) {
                chars[i] = rotateChar(chars[i], rotateBy, 'a', 'z');
            } else if (isUpperCase(chars[i])) {
                chars[i] = rotateChar(chars[i], rotateBy, 'A', 'Z');
            }
        }
    }

    private char rotateChar(char c, int rotateBy, char firstChar, char lastChar) {
        c += rotateBy;
        if (c < firstChar) {
            return (char) (c + ALPHABET_SIZE);
        }
        if (c > lastChar) {
            return (char) (c - ALPHABET_SIZE);
        }
        return c;
    }
}
```


# Unit tests

As always, the best way to verify is to write some [tests](http://farenda.com/junit-tutorial):

```java
package com.farenda.tutorials.algorithms.strings;

import org.junit.Test;

import static org.junit.Assert.*;

public class CaesarCipherTest {

    private CaesarCipher caesar = new CaesarCipher();

    @Test
    public void shouldDoNothingWithEmptyString() {
        assertEquals("", caesar.cipher("", 3));
    }

    @Test
    public void shouldNotCipherSymbols() {
        assertEquals("-", caesar.cipher("-", 3));

        String symbols = "1!@#$%^&*(){}/";
        assertEquals(symbols, caesar.cipher(symbols, 3));
    }

    @Test
    public void shouldCipherLowerCaseLetter() {
        assertEquals("a", caesar.cipher("a", 0));

        assertEquals("b", caesar.cipher("a", 1));
        assertEquals("d", caesar.cipher("a", 3));
        assertEquals("j", caesar.cipher("e", 5));

        assertEquals("a", caesar.cipher("z", 1));
        assertEquals("c", caesar.cipher("x", 5));
    }

    @Test
    public void shouldCipherUpperCaseLetter() {
        assertEquals("A", caesar.cipher("A", 0));

        assertEquals("B", caesar.cipher("A", 1));
        assertEquals("D", caesar.cipher("A", 3));
        assertEquals("J", caesar.cipher("E", 5));

        assertEquals("A", caesar.cipher("Z", 1));
        assertEquals("C", caesar.cipher("X", 5));
        assertEquals("Z", caesar.cipher("C", -3));
    }

    @Test
    public void shouldCipherWholeAlphabet() {
        String allChars = "THE QUICK BROWN FOX JUMPS OVER THE LAZY DOG";
        assertEquals("QEB NRFZH YOLTK CLU GRJMP LSBO QEB IXWV ALD",
                     caesar.cipher(allChars, -3));
    }

    @Test
    public void shouldCycle() {
        assertEquals("aoeu-snth", caesar.cipher("aoeu-snth", 52));
        assertEquals("cqgw-upvj", caesar.cipher("aoeu-snth", 54));
    }

    @Test
    public void shouldDecipher() {
        String s = "aoeu";

        String encrypted = caesar.cipher(s, 2);

        assertEquals(s, caesar.cipher(encrypted, -2));
    }
}
```


# References

-   [Algorithms and Data Structures Tutorial](http://farenda.com/algorithms-and-data-structures)

