---
title: Spring Field Injection
date: 2016-07-12 07:01:00 +0200
tags: [java,spring]
---


This is the form of **Dependency Injection** that requires almost no boilerplate
code, contrary to setter or constructor injections. Let's see how it works!

<!--more-->

## Spring bean to inject as dependency

To start with, we'll specify an interface for the dependency:
```java
package com.farenda.spring.tutorial.injection.field;

public interface BookRepository {
    String titleById(int id);
}
```

And the actual Spring Bean implementation that will be injected:
```java
package com.farenda.spring.tutorial.injection.field;

import org.springframework.stereotype.Repository;

import java.util.HashMap;
import java.util.Map;

@Repository
public class InMemoryBookRepository implements BookRepository {

    // It's our local database ;-)
    private Map<Integer, String> books = new HashMap<>();

    {
        books.put(1, "Effective Java, 3rd edition");
        books.put(2, "Java Concurrency in Practice");
        books.put(3, "Spring in Action");
    }

    @Override
    public String titleById(int id) {
        return books.get(id);
    }
}
```


## Spring component with dependency injected using field injection

Here we define another Spring Bean, but we are going to inject /BookRepository/
using field injection. To do that we have to annotate appropriate fields using
**@Autowired** (or @Inject or @Resource) annotation:
```java
package com.farenda.spring.tutorial.injection.field;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Library {

    @Autowired
    private BookRepository bookRepository;

    public String findBook(int id) {
        return bookRepository.titleById(id);
    }
}
```

It works without any setter, because underneath Spring is using reflection to
inject dependencies.


## Constructor or field injection

Although [constructor injection](https://farenda.com/spring/spring-constructor-injection) allows to create beans as **immutable objects**
(fields can be marked using **final** modifier) and makes dependencies explicit
it requires to write some boilerplate code. Because of that, in practice, field
injection is used.


## Example App using Spring Boot

Here's the simplest Spring Boot application to run the above code:
```java
package com.farenda.spring.tutorial.injection.field;

import org.springframework.boot.SpringApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class FieldInjection {

    public static void main(String[] args) {
        ApplicationContext context = SpringApplication
                .run(FieldInjection.class, args);

        Library library = context.getBean(Library.class);

        System.out.println("Title 1: " + library.findBook(1));
        System.out.println("Title 2: " + library.findBook(2));
    }
}
```

The above code produces the following output:

    Title 1: Effective Java, 3rd edition
    Title 2: Java Concurrency in Practice


# References:

- [Spring Reference Documentation](https://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle)
- [Spring Constructor Injection](https://farenda.com/spring/spring-constructor-injection)
- [Spring Setter Injection](https://farenda.com/spring/spring-setter-injection)

