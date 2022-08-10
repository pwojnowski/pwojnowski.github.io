---
title: Spring find annotated classes
date: 2015-08-24 07:01:00 +0200
categories: [Spring Framework]
tags: [java,spring]
---


How to find annotated classes using **Spring Framework** and read metadata from
them? Sometimes you may want to attach metadata to your classes using custom
annotations. Here's an example how you can leverage **Spring**'s classpath
scanning mechanism to do that.

<!--more-->


## The Spring Bean problem

If you use Spring annotations like **@Component**, **@Repository** or
**@Service**, then Spring will find such classes, but will make them Spring
beans.


## Classpath Scanner customization

Good news is that Spring classpath scanning mechanism is configurable and
available in any Spring application. To use custom annotations we have to
create an instance of ClassPathScanningCandidateComponentProvider and set
appropriate filter - here it is **AnnotationTypeFilter**. It returns
**BeanDefinitions** that contains names of found class from which we can get
detailed information. The following example will clarify that.

## Own annotations
We create our custom annotation that allows us to attach some metatdata:
```java
package com.farenda.java.lang;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

// Make the annotation available at runtime:
@Retention(RetentionPolicy.RUNTIME)
// Allow to use only on types:
@Target(ElementType.TYPE)
public @interface Findable {

    /**
     * User friendly name of annotated class.
     */
    String name();
}
```

## Adding metadata to own classes
Sample classes annotated with the custom annotation:
```java
package com.farenda.java.lang;

@Findable(name = "Find me")
public class FirstAnnotatedClass {
}

package com.farenda.java.lang;

@Findable(name = "Find me too")
public class SecondAnnotatedClass {
}
```

## Spring dependency
The classpath scanner is provided by **spring-context** project. Here's the
relevant **Maven dependency**:
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.2.0.RELEASE</version>
</dependency>
```

## The Scanner
The Java code below is using **ClassPathScanningCandidateComponentProvider**
to scan classes in **com.farenda.java.lang** package.
```java
package com.farenda.java.lang;

import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider;
import org.springframework.core.type.filter.AnnotationTypeFilter;

public class SpringClassScanner {

    public static void main(String[] args) throws Exception {
        System.out.println("Finding annotated classes using Spring:");
        new SpringClassScanner().findAnnotatedClasses("com.farenda.java.lang");
    }

    public void findAnnotatedClasses(String scanPackage) {
        ClassPathScanningCandidateComponentProvider provider = createComponentScanner();
        for (BeanDefinition beanDef : provider.findCandidateComponents(scanPackage)) {
            printMetadata(beanDef);
        }
    }

    private ClassPathScanningCandidateComponentProvider createComponentScanner() {
        // Don't pull default filters (@Component, etc.):
        ClassPathScanningCandidateComponentProvider provider
                = new ClassPathScanningCandidateComponentProvider(false);
        provider.addIncludeFilter(new AnnotationTypeFilter(Findable.class));
        return provider;
    }

    private void printMetadata(BeanDefinition beanDef) {
        try {
            Class<?> cl = Class.forName(beanDef.getBeanClassName());
            Findable findable = cl.getAnnotation(Findable.class);
            System.out.printf("Found class: %s, with meta name: %s%n",
                    cl.getSimpleName(), findable.name());
        } catch (Exception e) {
            System.err.println("Got exception: " + e.getMessage());
        }
    }
}
```

The code is straightforward. The hardest thing is to type and read very long
names of Spring classes. ;-)

And here's the output of running the code:

    Finding annotated classes using Spring:
    Found class: SecondAnnotatedClass, with meta name: Find me too
    Found class: FirstAnnotatedClass, with meta name: Find me

## Summary
This sort of scanning is very good fit for applications that already use
Spring Framework. However, in the future, we'll see how to solve the same
problem, but without Spring.
