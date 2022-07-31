---
title: Spring Boot "Hello, World"
date: 2015-06-09 10:05:00 +0200
categories: [Spring Framework]
tags: [java,spring,spring-boot]
---


# Problem:

How to create "**Hello, World**" app in **Spring Boot**?

<!--more-->


# Solution:

**Gradle** build script will all required dependencies:

```groovy
buildscript {
    repositories {
        jcenter()
        maven { url "http://repo.spring.io/snapshot" }
        maven { url "http://repo.spring.io/milestone" }
    }
    dependencies {
        // Provides tasks to create executable JAR and run project from source.
        // It also adds ResolutionStrategy, which allows to ommit versions in
        // "blessed" dependencies below.
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.2.3.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'spring-boot'

jar {
    baseName = 'spring-boot-hello-world'
    version =  '0.0.1-SNAPSHOT'
}

repositories {
    jcenter()
    maven { url "http://repo.spring.io/snapshot" }
    maven { url "http://repo.spring.io/milestone" }
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

Sample **REST** service to show how it is to do it with **Spring Boot**:

```java
package com.farenda.java.spring;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

// Tells Spring to consider this bean for handling incoming web requests
@Controller
// Tells Spring to guess application type based on its dependencies and
// setup application context accordingly:
@EnableAutoConfiguration
public class JavaSolved {

    // Make application accessible under http://localhost:8080/ path:
    @RequestMapping("/")
    // Tells to convert return value into HTTP response:
    @ResponseBody
    String home() {
        return "Hello, Spring Boot World!";
    }

    public static void main(String[] args) throws Exception {
        // Lets Spring bootstrap the application with
        // preconfigured Tomcat. The class name is used
        // to indicate primary Spring component:
        SpringApplication.run(JavaSolved.class, args);
    }
}
```


# Result:

Execute "**gradle bootRun**" from the console and open <http://localhost:8080> to
access the app:

```
$> gradle bootRun
:compileJava
:processResources UP-TO-DATE
:classes
:findMainClass
:bootRun

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.2.3.RELEASE)

[... a lot of lines ...]
2015-06-06 18:20:19.896  INFO 9273 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2015-06-06 18:20:19.906  INFO 9273 --- [           main] com.farenda.solved.JavaSolved            : Started JavaSolved in 5.255 seconds (JVM running for 6.0)
```

Let's use **curl** to check the result:

```
$> curl localhost:8080
Hello, Spring Boot World!
```

It works as advertised!
