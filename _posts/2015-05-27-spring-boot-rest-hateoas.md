---
title: Spring Boot REST HATEOAS
date: 2015-05-27 18:24:00 +0200
categories: [Spring Framework]
tags: [java,spring,spring-boot,rest]
---


# Problem:

How to create simple **Spring Boot REST** service with **HATEOAS** support?

It's very simple with **spring-boot-hateoas** project.

<!--more-->


# Solution:

Project configuration for Gradle (*build.gradle*):

```groovy
apply plugin: 'java'
apply plugin: 'groovy'
apply plugin: 'idea'
apply plugin: 'spring-boot'

sourceCompatibility = 1.8
targetCompatibility = 1.8

buildscript {
   repositories {
      mavenCentral()
   }
   dependencies {
      classpath("org.springframework.boot:spring-boot-gradle-plugin:1.2.3.RELEASE")
   }
}

jar {
   baseName = 'spring-boot-rest-hateoas-sample'
   version = '0.1.0'
}

repositories {
   mavenCentral()
}

dependencies {
   // Dependencies for Spring Boot with REST HATEOAS:
   compile "com.fasterxml.jackson.core:jackson-databind"
   compile "com.jayway.jsonpath:json-path:0.9.1"
   compile "org.springframework.boot:spring-boot-starter-hateoas"

   // Dependencies for Spock tests:
   compile "org.codehaus.groovy:groovy-all:2.4.1"
   testCompile "org.spockframework:spock-core:1.0-groovy-2.4"
   testCompile "org.spockframework:spock-spring:1.0-groovy-2.4"

   testCompile "org.springframework.boot:spring-boot-starter-test"
}
```

In **TDD** we start with a test, so create
src/test/groovy/com/farenda/java/spring/EchoServiceTest.groovy with the following
content:

```java
package com.farenda.java.spring

import com.fasterxml.jackson.databind.ObjectMapper
import org.springframework.beans.factory.annotation.Value
import org.springframework.boot.test.SpringApplicationContextLoader
import org.springframework.boot.test.TestRestTemplate
import org.springframework.boot.test.WebIntegrationTest
import org.springframework.hateoas.MediaTypes
import org.springframework.hateoas.ResourceSupport
import org.springframework.hateoas.hal.Jackson2HalModule
import org.springframework.hateoas.mvc.TypeConstrainedMappingJackson2HttpMessageConverter
import org.springframework.http.HttpStatus
import org.springframework.http.converter.HttpMessageConverter
import org.springframework.test.context.ContextConfiguration
import spock.lang.Shared
import spock.lang.Specification

// Need to specify "loader" here to workaround Spock issue
// with meta annotations:
// https://code.google.com/p/spock/issues/detail?id=349
@ContextConfiguration(classes = [HateoasExample],
                      loader = SpringApplicationContextLoader)
// Tell Spring to start Web Server on random port, because
// usual 8080 may be busy:
@WebIntegrationTest(randomPort = true)
class EchoServiceTest extends Specification {

   // Inject random port of test Web Server:
   @Value('${local.server.port}')
   def int serverPort

   @Shared
   def rest = getRestTemplateWithHalMessageConverter()

   // Registered converters are processed in order, so it's
   // not enough to add the new one - we need to add it at
   // the beginning, so it can process before generic JSON
   // converter, else HAL links won't be interpreted!
   def TestRestTemplate getRestTemplateWithHalMessageConverter() {
      def restTemplate = new TestRestTemplate()
      restTemplate.getMessageConverters().add(0, halMessageConverter())
      return restTemplate
   }

   // Needed to interpret HAL "_links". Without this
   // response will be with empty links!
   private static HttpMessageConverter halMessageConverter() {
      def objectMapper = new ObjectMapper().registerModule(new Jackson2HalModule())
      def halConverter = new TypeConstrainedMappingJackson2HttpMessageConverter(ResourceSupport)
      halConverter.setSupportedMediaTypes([MediaTypes.HAL_JSON])
      halConverter.setObjectMapper(objectMapper)
      return halConverter
   }

   def 'should receive sent message with self link'() {
      given:
      // toString() is only to convert from GString to a String:
      def url = "http://localhost:${serverPort}/echo?message=".toString()
      def query = url + '{message}'
      def myMessage = 'HelloWorld!'

      when:
      // Post GET to the service and receive response as ResponseEntity.
      // We want ResponseEntity to verify HttpStatus code and links.
      // The last parameter can also be a map: [message:myMessage]
      def response = rest.getForEntity(query, EchoResource, myMessage)

      then:
      response.statusCode == HttpStatus.OK
      response.body.message == myMessage
      response.body.getLinks().size() == 1
      response.body.getLink('self').href == url+myMessage
   }
}
```

Now, we've got a failing test. So here's the service:

**Spring Boot** config for the app:

```java
package com.farenda.java.spring;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HateoasExample {
    public static void main(String[] args) throws Exception {
        SpringApplication.run(HateoasExample.class, args);
    }
}
```

@SpringBootApplication gives *@Component, @Configuration, and
@EnableAutoConfiguration* for free.

**REST HATEOAS** resource:

```java
package com.farenda.java.spring;

import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import org.springframework.hateoas.ResourceSupport;

// Created separate class for "only" message to have
// also HATEOAS "links" from ResourceSupport - see
// EchoService for usage.
public class EchoResource extends ResourceSupport {

   private final String message;

   @JsonCreator
   public EchoResource(@JsonProperty("message") String message) {
      this.message = message;
   }

   public String getMessage() {
      return message;
   }
}
```

And, finally, **Spring REST HATEOAS** service:

```java
package com.farenda.java.spring;

import org.springframework.http.HttpEntity;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import static org.springframework.hateoas.mvc.ControllerLinkBuilder.linkTo;
import static org.springframework.hateoas.mvc.ControllerLinkBuilder.methodOn;

// It's like @Controller, but automatically assumes @ResponseBody.
@RestController
public class EchoService {

   @RequestMapping("/echo")
   public HttpEntity<EchoResource> echo(
          @RequestParam("message") String message) {
      EchoResource resource = new EchoResource(message);
      // Nice DSL for creating links. Here we don't have
      // other actions, so the only link is "self":
      resource.add(linkTo(methodOn(EchoService.class).echo(message))
                         .withSelfRel());
      return new ResponseEntity<>(resource, HttpStatus.OK);
   }
}
```


# Result:

You can run Spock tests as usual:

    $> gradle test

Also you can start the app and verify it:

    $> gradle bootRun
    $> curl http://localhost:8080/echo?message=Drakaris!
    {
        "message": "Drakaris!",
        "_links": {
            "self": {
                "href": "http://localhost:8080/echo?message=Drakaris!"
            }
        }
    }

