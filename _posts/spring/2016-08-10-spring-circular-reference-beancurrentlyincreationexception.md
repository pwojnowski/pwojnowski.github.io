---
title: Spring Circular Reference - BeanCurrentlyInCreationException
date: 2016-08-10 22:01:00 +0200
tags: [java,spring]
---

**BeanCurrentlyInCreationException** is thrown by **Spring Framework** when it meets
**circular reference** while setting up application context. Here we show what is
it and how to fix it!

<!--more-->


# Beans with circular reference

## Bean A with constructor injected Bean B

*Library* is a **Spring Bean** that is using another bean - *BookRepository* - as
a dependency [injected through constructor](https://farenda.com/spring/spring-constructor-injection):
```java
package com.farenda.spring.tutorial.injection.conflict;

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

    public String name() {
        return "Public library #1";
    }
}
```


## Bean B with constructor injected Bean A - circular reference

*BookRepository*, for some reason, want's to have *Library* injected, also
through constructor:
```java
package com.farenda.spring.tutorial.injection.conflict;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import java.util.HashMap;
import java.util.Map;

@Repository
public class BookRepository {

    private Library library;

    @Autowired
    public BookRepository(Library library) {
        this.library = library;
    }

    // It's our local database ;-)
    private Map<Integer, String> books = new HashMap<>();

    {
        books.put(1, "Effective Java, 2nd edition");
        books.put(2, "Java Concurrency in Practice");
        books.put(3, "Spring in Action");
    }

    public String titleById(int id) {
        System.out.println("In a library: " + library.name());
        return books.get(id);
    }
}
```


# The application runner

The simplest Spring Boot app to run the code:

```java
package com.farenda.spring.tutorial.injection.conflict;

import org.springframework.boot.SpringApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class InjectionConflict {

    public static void main(String[] args) {
        ApplicationContext context = SpringApplication
                .run(InjectionConflict.class, args);

        Library library = context.getBean(Library.class);

        System.out.println("Title 2: " + library.findBook(2));
    }
}
```

When we run the above code the application will fail during startup with
**UnsatisfiedDependencyException** exception, which is caused by **BeanCurrentlyInCreationException**:

    2016-08-09 23:29:01.530  INFO 11796 --- [           main] c.f.s.t.i.conflict.InjectionConflict     : Starting InjectionConflict on namek with PID 11796 (/home/przemek/Dropbox/projekty/farenda/java/spring-tutorial/build/classes/main started by przemek in /home/przemek/Dropbox/projekty/farenda/java/spring-tutorial)
    2016-08-09 23:29:01.535  INFO 11796 --- [           main] c.f.s.t.i.conflict.InjectionConflict     : No active profile set, falling back to default profiles: default
    2016-08-09 23:29:01.612  INFO 11796 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@2eda0940: startup date [Tue Aug 09 23:29:01 CEST 2016]; root of context hierarchy
    2016-08-09 23:29:02.147  WARN 11796 --- [           main] s.c.a.AnnotationConfigApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'bookRepository' defined in file [/home/przemek/Dropbox/projekty/farenda/java/spring-tutorial/build/classes/main/com/farenda/spring/tutorial/injection/conflict/BookRepository.class]: Unsatisfied dependency expressed through constructor argument with index 0 of type [com.farenda.spring.tutorial.injection.conflict.Library]: Error creating bean with name 'library' defined in file [/home/przemek/Dropbox/projekty/farenda/java/spring-tutorial/build/classes/main/com/farenda/spring/tutorial/injection/conflict/Library.class]: Unsatisfied dependency expressed through constructor argument with index 0 of type [com.farenda.spring.tutorial.injection.conflict.BookRepository]: Error creating bean with name 'bookRepository': Requested bean is currently in creation: Is there an unresolvable circular reference?; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'bookRepository': Requested bean is currently in creation: Is there an unresolvable circular reference?; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'library' defined in file [/home/przemek/Dropbox/projekty/farenda/java/spring-tutorial/build/classes/main/com/farenda/spring/tutorial/injection/conflict/Library.class]: Unsatisfied dependency expressed through constructor argument with index 0 of type [com.farenda.spring.tutorial.injection.conflict.BookRepository]: Error creating bean with name 'bookRepository': Requested bean is currently in creation: Is there an unresolvable circular reference?; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'bookRepository': Requested bean is currently in creation: Is there an unresolvable circular reference?
    2016-08-09 23:29:02.166 ERROR 11796 --- [           main] o.s.boot.SpringApplication               : Application startup failed
    
    org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'bookRepository' defined in file [/home/przemek/Dropbox/projekty/farenda/java/spring-tutorial/build/classes/main/com/farenda/spring/tutorial/injection/conflict/BookRepository.class]: Unsatisfied dependency expressed through constructor argument with index 0 of type [com.farenda.spring.tutorial.injection.conflict.Library]: Error creating bean with name 'library' defined in file [/home/przemek/Dropbox/projekty/farenda/java/spring-tutorial/build/classes/main/com/farenda/spring/tutorial/injection/conflict/Library.class]: Unsatisfied dependency expressed through constructor argument with index 0 of type [com.farenda.spring.tutorial.injection.conflict.BookRepository]: Error creating bean with name 'bookRepository': Requested bean is currently in creation: Is there an unresolvable circular reference?; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'bookRepository': Requested bean is currently in creation: Is there an unresolvable circular reference?; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'library' defined in file [/home/przemek/Dropbox/projekty/farenda/java/spring-tutorial/build/classes/main/com/farenda/spring/tutorial/injection/conflict/Library.class]: Unsatisfied dependency expressed through constructor argument with index 0 of type [com.farenda.spring.tutorial.injection.conflict.BookRepository]: Error creating bean with name 'bookRepository': Requested bean is currently in creation: Is there an unresolvable circular reference?; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'bookRepository': Requested bean is currently in creation: Is there an unresolvable circular reference?
            at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:749) ~[spring-beans-4.2.6.RELEASE.jar:4.2.6.RELEASE]
            at org.springframework.beans.factory.support.ConstructorResolver.autowireConstructor(ConstructorResolver.java:185) ~[spring-beans-4.2.6.RELEASE.jar:4.2.6.RELEASE]

The reason is that **Spring Framework** first tries to instantiate all the beans
and then inject them. When constructor dependency is used, **Spring Framework**
cannot instantiate mutually dependent beans. To fix that the **beans**
participating in **circular reference** have to use **field/setter injection**.


# Field/setter injection to break constructor circular references

To fix that let's use [field injection](https://farenda.com/spring/spring-field-injection) in the beans:

```java
@Repository
public class BookRepository {

    @Autowired
    private Library library;

//    @Autowired
//    public BookRepository(Library library) {
//        this.library = library;
//    }

//...
}

public class Library {

    @Autowired
    private BookRepository bookRepository;

//    public Library(BookRepository bookRepository) {
//        this.bookRepository = bookRepository;
//    }

//...
}
```

Now **Spring Framewokr** can instantiate *BookRepository*, inject it through
the constructor into the *Library*, and then inject it back to the
*BookRepository*:

    2016-08-09 23:55:54.594  INFO 12472 --- [           main] c.f.s.t.i.conflict.InjectionConflict     : Started InjectionConflict in 1.553 seconds (JVM running for 2.679)
    In a library: Public library #1
    Title 2: Java Concurrency in Practice
    2016-08-09 23:55:54.605  INFO 12472 --- [       Thread-1] s.c.a.AnnotationConfigApplicationContext : Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@2eda0940: startup date [Tue Aug 09 23:55:54 CEST 2016]; root of context hierarchy


# References:

- [Spring Constructor Injection](https://farenda.com/spring/spring-constructor-injection)
- [Spring Setter Injection](https://farenda.com/spring/spring-setter-injection)
- [Spring Field Injection](https://farenda.com/spring/spring-field-injection)

