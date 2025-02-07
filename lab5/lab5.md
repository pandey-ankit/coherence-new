# Working with Cache Stores

## Introduction

Coherence Spring provides dedicated support for database-backed caches using JPA. Spring Data’s [JPA Repositories](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.repositories) 
make basic CRUD database access very simple. An application developer can just provide an interface that extends 
`JpaRepository` with the required generic parameters and Spring will do the rest.

Coherence caches that are backed by a database have two options for how the database integration is provided:

* [CacheLoader]((https://coherence.community/24.09/api/java/com/tangosol/net/cache/CacheLoader.html)) - an application developer writes an implementation of a CacheLoader to read data from a database for a given key (or keys), convert it to entities that are then loaded into a cache for the given keys.
* [CacheStore](https://coherence.community/24.09/api/java/com/tangosol/net/cache/CacheStore.html) - whilst a CacheLoader only loads from a database into a cache, a CacheStore (which extends CacheLoader) also stores cached entities back to the database, or for entries deleted from the cache, erases the corresponding values from the database. The parallels between a CacheLoader or CacheStore and a JpaRepository should be pretty obvious.

The Coherence Spring core module provides two interfaces:

* [JpaRepositoryCacheLoader](https://spring.coherence.community/4.1.3/refdocs/api/com/oracle/coherence/spring/cachestore/JpaRepositoryCacheLoader.html), which extends both JpaRepository and CacheLoader
* [JpaRepositoryCacheStore](https://spring.coherence.community/4.1.3/refdocs/api/com/oracle/coherence/spring/cachestore/JpaRepositoryCacheStore.html), which extends both JpaRepository and CacheStore.

To create a JPA repository cache loader or cache store, all a developer needs to do is extend the relevant interface JpaRepositoryCacheLoader or JpaRepositoryCacheStore with the correct generic parameters. 


Estimated time: 20 minutes

### Objectives

In this lab, you will:
     
* Extend the existing Customer POJO
* Write a JPA Repository CacheStore


### Prerequisites
     
You should have completed the previous labs.

## Task 1: Write the JPA Repository CacheStore

1. In the base `pom.xml`, add the following dependency so we can use the `JpaRepository` interface:

      ```xml
      <dependency>
         <groupId>org.springframework.data</groupId>
         <artifactId>spring-data-jpa</artifactId>
         <scope>provided</scope>
      </dependency>
      ```

2. Create a new file called `CustomerRepository.java` in the directory `./src/main/java/com/oracle/coherence/demo/frameworks/springboot/` with the following contents:

      ```java
      package com.oracle.coherence.demo.frameworks.springboot;

      import org.springframework.data.jpa.repository.JpaRepository;
      import org.springframework.stereotype.Repository;

      @Repository
      public interface CustomerRepository extends JpaRepository<Customer, Integer> {
      }
      ```

## Task 2: Add the `CustomerRepository` to the cache configuration

For this example, we use a co-located Coherence instance that will start as part of the application itself. Normally you would have a 
separate storage-disabled client and storage-enabled tier to allow you to scale each tier separately.

To use a CacheStore in Coherence, it needs to be configured in the Coherence cache configuration file, which in the 
embedded use-case is coherence-cache-config.xml. In order to use the repository bean as a CacheStore, we will make use 
of the Coherence Spring feature that allows injection of Spring beans into the cache configuration file.

To use Spring bean injection in the configuration file we need to declare a custom namespace in the root XML 
element that references the Coherence Spring NamespaceHandler.
    
1. Add the spring Namespace handler by changing the `cache-config` element in the file `example-cache-config.xml` we created in `src/main/resources` in the last lab.

      ```xml
      <cache-config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xmlns="http://xmlns.oracle.com/coherence/coherence-cache-config"
              xmlns:spring="class://com.oracle.coherence.spring.namespace.NamespaceHandler"
              xsi:schemaLocation="http://xmlns.oracle.com/coherence/coherence-cache-config coherence-cache-config.xsd">
      ```
         
   The `xmlns:spring="class://com.oracle.coherence.spring.namespace.NamespaceHandler"` line declares the custom namespace, 
   so elements with a prefix spring will be handled by the com.oracle.coherence.spring.namespace.NamespaceHandler class. The custom namespace handler allows us to use elements of the form 
   `<spring:bean>bean-name</spring:bean>` anywhere in the configuration that Coherence normally 
   allows an <instance> element or a <class-scheme> element.

2. Add the cache store to the cache configuration by updating the `distributed-scheme` to be:

      ```xml
      <caching-schemes>
        <distributed-scheme>
          <scheme-name>server</scheme-name>
          <backing-map-scheme>
            <read-write-backing-map-scheme>
            <internal-cache-scheme>
              <local-scheme>
                <unit-calculator>BINARY</unit-calculator>
              </local-scheme>
            </internal-cache-scheme>
            <cachestore-scheme>
              <spring:bean>{repository-bean}</spring:bean>
            </cachestore-scheme>
          </read-write-backing-map-scheme>
        </backing-map-scheme>
        <autostart>true</autostart>
      </distributed-scheme>
      ```    
   
   > Note: In the snippet above you can see the `<spring:bean>{repository-bean}</spring:bean>` element used as the cache store. 
   > In this case we have not used the name of the repository bean directly, we have used a parameter named repository-bean
   > (XML values in curly-brackets in the `<spring:bean>` element are treated as parameter macros). This allows us to map
   > multiple caches to the same scheme each with a different cache store - this is quite a common 
   > approach in Coherence for a number of elements that may be configured in a scheme per-cache. 
   > We can now also add the cache mapping for our people cache that will use the scheme above.

3. Update the `cache-mapping` to include the `repository-bean` by add the following xml:

      ```xml
      <cache-mapping>
        <cache-name>*</cache-name>
        <scheme-name>server</scheme-name>
        <init-params>
          <init-param>
            <param-name>repository-bean</param-name>
            <param-value>customerRepository</param-value>
          </init-param>
        </init-params>
      </cache-mapping>
      ```   
   
   > Note: The bean name used here is `customerRepository`. This is the default name generated by Spring for the `CustomerRepository`
   > class, which is the simple class name with the first letter lowercase. If we did not want to rely on Spring generating a bean name 
   > we could specify a name in the `@Repository` annotation on the `CustomerRepository` class.

4. Inspect the complete updated cache configuration file:

      ```xml
      
      ```

## Task 3: Update the `DemoController` to add the cache store.

1. Add the `CustomerRepository` to `DemoController.java` by adding the following:

      ```java
      @Autowired
      private CustomerRepository customerRepository;
      ```



## Task 4: Title for task 3
       


   
## Learn More
            


## Acknowledgements

* **Author** - Tim Middleton
* **Contributors** - Ankit Pandey, Sid Joshi
* **Last Updated By/Date** - Ankit Pandey, November 2024