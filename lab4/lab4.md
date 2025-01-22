# Using a Custom Cache Configuration File

## Introduction
                     
Coherence Spring uses the Coherence bootstrap API introduced in Coherence CE 20.12 to configure and create Coherence instances. 
This means that Coherence resources in a Spring application are typically part of a Coherence Session.

By default, Coherence will start a single Session configured to use the default Coherence configuration file. 
This behavior can easily be configured using traditional Coherence using system properties or using dedicated configuration.
The default cache configuration file is `coherence-cache-config.xml` which is located in `coherence.jar`. 
This can be sufficient for simple applications, but usually you require a custom cache configuration file.

In this lab we will define and use a custom Coherence cache configuration file and configure Coherence to use it.

Estimated time: 10 minutes

### Objectives

In this lab, you will:

* Create a new cache configuration file
* Crate a `SessionConfigurationBean` to use the new cache configuration 
* Ensure that Coherence loads this on startup

### Prerequisites
     
This lab assumes you have:

* An Oracle Free Tier(Trial), Paid or LiveLabs Cloud Account
* You have completed:
  * Lab: Prepare Setup (Free-tier and Paid Tenants only)
  * Lab: Initialize Environment
  * Lab: Getting Started with Coherence and Spring
  * Lab: Working With Events

## Task 1: Create a cache configuration file

1. In Visual Studio code, create a new file in the directory `src/main/resources/` called `example-cache-config.xml`

2. Add the following contents to the new file:
   
      ```xml
      <?xml version="1.0"?>
      <cache-config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                    xmlns="http://xmlns.oracle.com/coherence/coherence-cache-config"
                    xsi:schemaLocation="http://xmlns.oracle.com/coherence/coherence-cache-config coherence-cache-config.xsd">

        <caching-scheme-mapping>
          <cache-mapping>
            <cache-name>*</cache-name>
            <scheme-name>server</scheme-name>
          </cache-mapping>
        </caching-scheme-mapping>

        <caching-schemes>
          <distributed-scheme>
            <scheme-name>server</scheme-name>
            <backing-map-scheme>
              <local-scheme>
                <unit-calculator>BINARY</unit-calculator>
              </local-scheme>
            </backing-map-scheme>
          </distributed-scheme>
        </caching-schemes>
      </cache-config>
      ```

   > Note: This is just a basic cache configuration for the moment. We will update this in future labs.

## Task 2: Use the new cache configuration

1. Create a new file called `CoherenceConfig.java` in the directory `./src/main/java/com/oracle/coherence/demo/frameworks/springboot/` with the following contents:

      ```java
      package com.oracle.coherence.demo.frameworks.springboot;

      import com.oracle.coherence.spring.configuration.session.SessionConfigurationBean;
      import org.springframework.context.annotation.Bean;
      import org.springframework.context.annotation.Configuration;

      @Configuration
      public class CoherenceConfig {
          @Bean
          SessionConfigurationBean sessionConfigurationBeanDefault() {
              final SessionConfigurationBean sessionConfigurationBean = new SessionConfigurationBean();
              sessionConfigurationBean.setConfig("example-cache-config.xml");
              return sessionConfigurationBean;
          }
      }
      ``` 
   
      In the above example, we are only setting the cache configuration file via the `setConfig()` method, but there are 
      a number of other options, shown below, that are available to set. See the [JavaDoc](https://spring.coherence.community/4.3.0/refdocs/api/com/oracle/coherence/spring/configuration/session/SessionConfigurationBean.html).

      * `setType()` - specify the type of connection, either `SessionType.SERVER` - Join as Coherence cluster member (the default), or `SessionType.CLIENT` - connect to Coherence as a Coherence*Extend or gRPC client
      * `setScope()` - allows setting of the scope for service to separate caches and services based upon this scope

2. In a terminal, issue the following command to re-build the application:

      ```bash
      mvn clean install -DskipTests
      ```

3. Then run the following command to start the application:

      ```bash
      java -jar target/springboot-1.0-SNAPSHOT.jar
      ```   
   
      You should see output similar to the following indicating our new cache config has been loaded:

      ```bash
      Loaded cache configuration from "jar:nested:/Users/timmiddleton/Documents/CoherenceEngineering/github/spring-workshop/target/springboot-1.0-SNAPSHOT.jar/!BOOT-INF/classes/!/example-cache-config.xml"
      ```

## Learn More
            


## Acknowledgements

* **Author** - Tim Middleton
* **Contributors** - Ankit Pandey, Sid Joshi
* **Last Updated By/Date** - Ankit Pandey, November 2024