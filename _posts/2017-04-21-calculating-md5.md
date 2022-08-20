---
title: Calculating MD5 hash
date: 2017-04-21 12:30:00 +0200
tags: [security,java,md5,cryptography]
---


**MD5** is a popular cryptographic algorithm used for sign messages, as
checksum, and various other cases. In this post we're going to show a few
approaches to calculate it, especially in **JVM** land.

<!--more-->

MD5 (Message Digest 5) algorithm should not be used for serious cryptography,
because it's insecure and can be broken very fast using even commodity
hardware. Nevertheless, you may encounter it in various systems used for
different purposes, therefore it's good to know how to work with it. So, let's
start!


# MD5 Hash in Java

*MD5* hash sum is usually represented as a sequence of *32 hex digits*. To
calculate MD5 in JVM languages we can use `java.security.MessageDigest` class
that provides different *Message Digest* algorithms. One of them is
**MD5**. Let's calculate the hash of `Hello World` string:
```java
package com.farenda.java.security;

import java.math.BigInteger;
import java.nio.charset.Charset;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

import static java.nio.charset.StandardCharsets.UTF_8;

public class SecurityPlayground {

    public static void main(String[] args) throws NoSuchAlgorithmException {
        String data = "Hello World";
        System.out.printf("MD5 of '%s': %s%n", data, md5(data));
    }

    private static String md5(String data) throws NoSuchAlgorithmException {
        // Get the algorithm:
        MessageDigest md5 = MessageDigest.getInstance("MD5");
        // Calculate Message Digest as bytes:
        byte[] digest = md5.digest(data.getBytes(UTF_8));
        // Convert to 32-char long String:
        return String.format("%032x%n", new BigInteger(1, digest));
    }
}
```

The most tricky part is the conversion from the *digest bytes* into *32 chars
long* string. We just create very big number from the bytes and then format it into
32 chars long String, with zero padding in case of smaller result (to always
have 32 chars.)

The code prints the following MD5 hash:

    MD5 of 'Hello World': b10a8db164e0754105b7a99be72e3fe5


# MD5 hash using Guava

Another, even simpler, approach to calculating MD5 hash is to use `Hashing`
class from **Google Guava** library. The library is often used in Java projects,
so most probably it's also available in yours.
```java
import com.google.common.hash.Hashing;

//... rest of the code is like the above

private static String guavaMd5(String data) {
    return Hashing.md5().hashString(data, UTF_8).toString();
}

public static void main(String[] args) throws NoSuchAlgorithmException {
    String data = "Hello World";
    System.out.printf("MD5 in Guava of '%s': %s%n",
        data, guavaMd5(data));
}
```

When we execute this code it will print the following text:

    MD5 in Guava of 'Hello World': b10a8db164e0754105b7a99be72e3fe5

So the MD5 hash is the same as in pure Java version.


# Calculate using md5sum command

Sometimes we just need to verify the MD5 hash. This can be easily done using
`md5sum` command available in many systems, especially in Linux.

To compare the hashes calculated above, we can just call the command like this:
```bash
echo -n "Hello World" | md5sum
```

We've used `-n` option to not include the new line character, automatically added
by `echo`. This is very important as the extra new line would change the
calculation. And here's the result:

    b10a8db164e0754105b7a99be72e3fe5  -

This confirms that our programs calculated MD5 as expected.


# References

- [MD5 at Wikipedia](https://en.wikipedia.org/wiki/MD5)
