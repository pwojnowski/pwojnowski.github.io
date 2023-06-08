---
title: Spring Constructor Injection
date: 2016-06-13 07:01:00 +0200
tags: [java,spring]
---


This is one of the simplest and cleanest types of Dependency Injection in
the Spring Framework. In this post we show how to use it with help of
Spring annotations.

<!--more-->


## Spring bean to inject as dependency

Let's specify an interface for dependency:
```java
package com.farenda.spring.tutorial.injection.constructor;

public interface BookRepository {
    String titleById(int id);
}
```

And the actual Spring Bean implementation that will be injected:
```java
package com.farenda.spring.tutorial.injection.constructor;

import org.springframework.stereotype.Repository;

import java.util.HashMap;
import java.util.Map;

@Repository
public class InMemoryBookRepository implements BookRepository {

    // It's our local database ;-)
    private Map<Integer, String> books = new HashMap<>();

    {
        books.put(1, "Effective Java, 2nd edition");
        books.put(2, "Java Concurrency in Practice");
        books.put(3, "Spring in Action");
    }

    @Override
    public String titleById(int id) {
        return books.get(id);
    }
}
```
## Spring component with dependency injected by constructor

Here we define another Spring Bean, but this time we are going to inject
_BookRepository_ through constructor, using *@Autowired* annotation:
```java
package com.farenda.spring.tutorial.injection.constructor;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Library {

    private BookRepository bookRepository;

    @Autowired
    public Library(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    public String findBook(int id) {
        return bookRepository.titleById(id);
    }
}
```

Spring resolves dependencies using argument's type. If it finds a bean
matching the type, then the bean is instantiated and injected. Else exception is
thrown.


## Example App using Spring Boot

Here's the simplest Spring Boot application to run the above code:
```java
package com.farenda.spring.tutorial.injection.constructor;

import org.springframework.boot.SpringApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan // scan this package for Spring beans
public class ConstructorInjection {

    public static void main(String[] args) {
        ApplicationContext context = SpringApplication
                .run(ConstructorInjection.class, args);

        // Get bean from the Spring Application Context:
        Library library = context.getBean(Library.class);

        System.out.println("Title 1: " + library.findBook(1));
        System.out.println("Title 2: " + library.findBook(2));
    }
}
```

The above code produces the following output:

    Title 1: Effective Java, 2nd edition
    Title 2: Java Concurrency in Practice


# References:

- Check out [Spring Boot "Hello, world"](https://farenda.com/spring/spring-boot-hello-world) if you are new to **Spring Boot**

