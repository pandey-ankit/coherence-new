# Coherence Spring Session

## Introduction

In this chapter you will learn how to configure Coherence as an HTTP session store using Spring Session.

We will modify the previous REST end-points by adding a simple 'hello' service that will allow us to set and change session attributes.

Estimated time: 15 minutes

### Objectives

In this lab, you will:

* Configure Spring Session
* Modify the application to add a 'hello' service
* View the session information in VisualVM

### Prerequisites

You should have completed the previous labs.

## Task 1: Configure Spring Session

1. In the base `pom.xml`, add the following dependency so you can use Coherence Spring Session:

      ```xml
      <dependency>
        <groupId>com.oracle.coherence.spring</groupId>
        <artifactId>coherence-spring-session</artifactId>
        <version>${coherence-spring.version}</version>
      </dependency>
      ```

2.  Add the `@EnableCoherenceHttpSession` annotation to the existing `CoherenceConig.java` to enable the Coherence Spring Session.

      ```java
      @EnableCoherenceHttpSession(
          cache = "spring:session:sessions",
          flushMode = FlushMode.ON_SAVE,
          sessionTimeoutInSeconds = 1800,
          useEntryProcessor = true
      )
      public class CoherenceConfig {
      ...      
      }
      ```
    
      Notes regarding the above:      
    
      * `@EnableCoherenceHttpSession` - Enables Spring Session support for Coherence
      * `session` - Specifies the name of the Coherence Session. Optional, Defaults to Coherence' default session.
      * `cache` - The name of the cache to use. Optional. Defaults to spring:session:sessions.
      * `flushMode` - The FlushMode to use. Optional. Defaults to FlushMode.ON_SAVE.
      * `sessionTimeoutInSeconds` The session timeout. Optional. Defaults to 1800 seconds (30 minutes)
      * `useEntryProcessor` - () When doing HTTP session updates, shall we use a Coherence entry processor? The default is {@code true}.

3.  Enable POF serializer by adding the file `example-pof-config.xml` in `src/main/resources`:

      ```xml
      <?xml version="1.0"?>
      <pof-config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://xmlns.oracle.com/coherence/coherence-pof-config"
         xsi:schemaLocation="http://xmlns.oracle.com/coherence/coherence-pof-config coherence-pof-config.xsd">
    
        <user-type-list>
          <include>coherence-pof-config.xml</include>
          <user-type>
            <type-id>2001</type-id>
            <class-name>org.springframework.session.MapSession</class-name>
            <serializer>
              <class-name>com.oracle.coherence.spring.session.serialization.pof.MapSessionPofSerializer</class-name>
            </serializer>
          </user-type>
          <user-type>
            <type-id>4000</type-id>
            <class-name>com.oracle.coherence.spring.session.SessionUpdateEntryProcessor</class-name>
          </user-type>
        </user-type-list>
      </pof-config>
      ```

      > Note: The above enables specific POF serialization required by Coherence Spring Session when using POF. Also as we are using entry processors, we must also specify the `SessionUpdateEntryProcessor` class. 

4.  Specify to use the above POF config file by adding the following to `src/main/resources/application.properites` 

      ```properties
      coherence.pof.config=example-pof-config.xml
      ```
 
5.  Add a new class called `HelloController` in the same package as `DemoController.java` with the following contents:

      ```java
      package com.oracle.coherence.demo.frameworks.springboot.controller;

      import jakarta.servlet.http.HttpSession;

      import org.apache.logging.log4j.LogManager;
      import org.apache.logging.log4j.Logger;

      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.http.ResponseEntity;
      import org.springframework.stereotype.Controller;
      import org.springframework.web.bind.annotation.DeleteMapping;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.PathVariable;
      import org.springframework.web.bind.annotation.RequestMapping;

      @Controller
      @RequestMapping(path = "/api/hello")
      public class HelloController {

          private static final Logger logger = LogManager.getLogger(HelloController.class);

          @Autowired
          private HttpSession session;

          /**
           * Increments the specified counter variable on each request.
           * @param counter the counter name to increment
           * @return a simple String containing the counter and the session id.
           */
          @GetMapping("/{counter}")
          public ResponseEntity<String> updateCounter(@PathVariable String counter) {
              Integer counterValue = (Integer) this.session.getAttribute(counter);
              if (counterValue == null) {
                  counterValue = 1;
                  this.session.setAttribute(counter, counterValue);
              }
              else {
                  counterValue++;
                  this.session.setAttribute(counter, counterValue);
              }

              logger.info("Session ID: {}; counterValue = {}", this.session.getId(), counterValue);
              return ResponseEntity.ok("hello " + counter + "=" + counterValue);
          }

          /**
           * Deletes the specified counter variable on each request.
           * @param counter the counter name to delete
           */
          @DeleteMapping("/{counter}")
          public ResponseEntity<String> deleteCounter(@PathVariable String counter) {
              this.session.removeAttribute(counter);

              logger.info("Session ID: {}; deleted counter = {}", this.session.getId(), counter);
              return ResponseEntity.ok("deleted counter " + counter);
          }
      }
      ```


## Task 2: Build and run the 



## Task 3: Title for task 3



## Task 4: Title for task 3
       


   
## Learn More
            
* [Coherence Spring Session](https://docs.coherence.community/coherence-spring/docs/latest/refdocs/reference/htmlsingle/index.html#spring-session)

## Acknowledgements

* **Author** - Tim Middleton
* **Contributors** - Ankit Pandey, Sid Joshi
* **Last Updated By/Date** - Ankit Pandey, November 2024