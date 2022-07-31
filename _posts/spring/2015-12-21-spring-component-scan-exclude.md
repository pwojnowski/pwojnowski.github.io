---
title: Spring component scan exclude
date: 2015-12-21 07:01:00 +0200
categories: [Spring Framework]
tags: [java,spring]
---


How to **exclude classes/packages** from **Component Scan** in **Spring Framework**? The
task is a bit tricky, especially when **regexp** is involved. See how it works!

<!--more-->


# Example Spring beans

Let's say that we've got the following two classes in
**com.farenda.java.spring.excludescan** package:

```java
package com.farenda.java.spring.excludescan;

import org.springframework.stereotype.Service;

@Service
public class ExcludedService {

    public ExcludedService() {
        System.out.println("Instantiating " + getClass().getSimpleName());
    }
}

package com.farenda.java.spring.excludescan;

import org.springframework.stereotype.Service;

@Service
public class IncludedService {

    public IncludedService() {
        System.out.println("Instantiating " + getClass().getSimpleName());
    }
}
```

We want to scan the whole package and **include every component** except
**ExcludedService**. In our case included beans should contain only
**IncludedService**.


# ComponentScan exclude - using annotations

To exclude some beans we can use **excludeFilters** on the **@ComponentScan**
annotation like this:

```java
package com.farenda.java.spring;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.*;
import org.springframework.context.annotation.ComponentScan.Filter;

import java.util.Arrays;

@ComponentScan(basePackages = "com.farenda.java.spring.excludescan",
        excludeFilters =@Filter(
                type = FilterType.REGEX,
                pattern = "com\\.farenda\\.java\\.spring\\.excludescan\\.Excluded.*"))
@EnableAutoConfiguration
public class ExcludingFromComponentScanExample {

    public static void main(String[] args) throws Exception {
        ApplicationContext context =
                SpringApplication.run(ExcludingFromComponentScanExample.class, args);

        String[] beans = context.getBeanDefinitionNames();

        // sort names only to make output more readable:
        Arrays.sort(beans);

        System.out.println("Defined beans: ");
        for (String bean : beans) {
            System.out.println(bean);
        }
    }
}
```

We apply **REGEX FilterType** that allows us nicely exclude single classes and
whole packages. When we run the application you can see that only
**includedService** is among beans in **ApplicationContext** built from **ComponentScan**:

    ...
     :: Spring Boot ::        (v1.2.3.RELEASE)
    
    2015-12-20 13:08:37.601  INFO 4077 --- [           main] .f.j.s.ExcludingFromComponentScanExample
    2015-12-20 13:08:37.660  INFO 4077 --- [           main] s.c.a.AnnotationConfigApplicationContext
    Instantiating IncludedService
    ...
    2015-12-20 13:08:39.292  INFO 4077 --- [           main] .f.j.s.ExcludingFromComponentScanExample : Started ExcludingFromComponentScanExample in 2.204 seconds (JVM running for 2.841)
    Defined beans:
    excludingFromComponentScanExample
    includedService
    mbeanExporter
    mbeanServer
    objectNamingStrategy
    org.springframework.boot.autoconfigure.AutoConfigurationPackages
    org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration
    org.springframework.boot.autoconfigure.condition.BeanTypeRegistry
    org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration
    org.springframework.context.annotation.ConfigurationClassPostProcessor.enhancedConfigurationProcessor
    org.springframework.context.annotation.ConfigurationClassPostProcessor.importAwareProcessor
    org.springframework.context.annotation.internalAutowiredAnnotationProcessor
    org.springframework.context.annotation.internalCommonAnnotationProcessor
    org.springframework.context.annotation.internalConfigurationAnnotationProcessor
    org.springframework.context.annotation.internalRequiredAnnotationProcessor
    propertySourcesPlaceholderConfigurer
    2015-12-20 13:08:39.295  INFO 4077 --- [       Thread-1] s.c.a.AnnotationConfigApplicationContext : Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@7a187f14: startup date [Sun Dec 20 13:08:37 CET 2015]; root of context hierarchy
    2015-12-20 13:08:39.298  INFO 4077 --- [       Thread-1] o.s.j.e.a.AnnotationMBeanExporter        : Unregistering JMX-exposed beans on shutdown
    
    Process finished with exit code 0

The same filtering mechanism can be used with **includeFilters** attribute, to
**select classes/packages** that should be included in **Spring ApplicationContext**.


# ComponentScan exclude - XML version

If you happen to still work with **XML Spring Configuration**, then the same thing
can be done in the following way:

```xml
<context:component-scan
    base-package="com.farenda.java.spring.excludescan">

  <context:exclude-filter type="regex"
      expression="com\.farenda\.java\.spring\.excludescan\.Excluded.*"/>

</context:component-scan>
```


# Resources

-   Check out [Spring Framework Tutorial](https://farenda.com/spring-framework)!

