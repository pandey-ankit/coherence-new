# Getting Started with Coherence and Spring

## Introduction

This section starts with a simple application configured with Coherence using [Spring Framework](https://spring.io/projects/spring-framework).
We will extend this application with various Coherence features and explain how they are used.


Estimated time: 30 minutes

### Objectives

In this lab, you will:

* Run the simple REST application
* Add Coherence event listeners
* Work with Interceptors 

### Prerequisites

This lab assumes you have:

* An Oracle Free Tier(Trial), Paid or LiveLabs Cloud Account
* You have completed:
  * Lab: Prepare Setup (Free-tier and Paid Tenants only)
  * Lab: Initialize Environment
  * Lab: Run the Spring Boot Quick Start Demo

The following has already been set up in this VM:

1. The spring-workshop repository has been cloned from .... into `~/spring-workshop`. We will build upon this workshop in this lab.
    
> Note: Make sure you have stopped all Java processes from the previous lab.

## Task 1: Build and run the simple example

1. Open a new terminal and change to the `spring-workshop` directory and verify the environment.

      ```bash
      cd ~/spring-workshop
      mvn -v
      ```   
   
   You will have output similar to the following:

      ```bash
      Apache Maven 3.8.8 (4c87b05d9aedce574290d1acc98575ed5eb6cd39)
      Maven home: /home/opc/Downloads/apache-maven-3.8.8
      Java version: 21.0.5, vendor: Oracle Corporation, runtime: /usr/lib/jvm/jdk-21.0.5-oracle-x64
      Default locale: en_US, platform encoding: UTF-8
      OS name: "linux", version: "5.15.0-104.119.4.2.el8uek.x86_64", arch: "amd64", family: "unix"
      ```   

2. In the same terminal, issue the following command to build the application:

      ```bash
      mvn clean install -DskipTests
      ```
   
   You should see output similar to the following indicating that the sample has been built:

      ```bash
      [INFO] ------------------------------------------------------------------------
      [INFO] BUILD SUCCESS
      [INFO] ------------------------------------------------------------------------
      ``` 

3. In the same terminal, run the following command to start the application:

      ```bash
      java -jar target/springboot-1.0-SNAPSHOT.jar
      ```

   You should see output similar to the following indicating that the sample is running:

      ```bash
      2025-01-21T10:12:51.315+08:00  INFO 12338 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 1 endpoint beneath base path '/actuator'
      2025-01-21T10:12:51.353+08:00  INFO 12338 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
      2025-01-21T10:12:51.366+08:00  INFO 12338 --- [           main] c.o.c.d.f.springboot.DemoApplication     : Started DemoApplication in 8.858 seconds (process running for 9.422)
      ``` 

4. In a new terminal window, run the following command to insert a customer.

      ```bash
      curl -X POST -H "Content-Type: application/json" -d '{"id": 1, "name": "Tim", "balance": 1000}' http://localhost:8080/api/customers
      ```

5. Run the following command to retrieve a customer.

      ```bash
      curl http://localhost:8080/api/customers/1
      ```   
   
   You should see output similar to the following indicating that the customer has been retrieved:

      ```json 
      {"id":1,"name":"Tim","balance":1234.0}
      ```  
     
6. Run the following command to delete a customer.

      ```bash
      curl -X DELETE http://localhost:8080/api/customers/1
      ```   
   
   You should see output similar to the following showing the deleted customer.

      ```json 
      {"id":1,"name":"Tim","balance":1234.0}
      ```  
   
   > Note: You can verify the customers cache  by using VisualVM as we did in the previous lab. Ensure that you close the tab you opened with the previous process and double-click on the new (`springboot-1.0-SNAPSHOT.jar`) process.
   
7Use `CTRL-C` to quit the Spring Boot application before you move to the next task.
    
> Note: In this simple example we are running the application as a single storage-enabled member meaning that 
> the application is serving JAX-RS endpoints as well as storing data. This is fine for a demo application, but 
> typical applications usually have a separate tier of storage-enabled clients, and the application is storage-disabled
> and allows for scaling of both the client and coherence cluster tiers.

## Task 2: Understand the simple application dependencies

   In this task, we will cover the dependencies required for the simple application. You can refer to the file `~/spring-workshop/pom.xml` for the full contents.

1. In the `pom.xml` we define the parent pom to be `spring-boot-starter-parent`, as this will bring in the required spring boot dependencies.

      ```xml
      <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.1</version>
       <relativePath/>
      </parent>
      ```
    
2. We then define the name of this project and the `properties` for other dependencies:

      ```xml
      <groupId>com.oracle.coherence.demo.workshops</groupId>
      <artifactId>springboot</artifactId>
      <version>1.0-SNAPSHOT</version>
      <name>demo</name>
      <description>Demo project for Spring Boot and Coherence</description>

      <properties>
        <java.version>21</java.version>
        <coherence.version>24.09</coherence.version>
        <coherence.group.id>com.oracle.coherence.ce</coherence.group.id>
        <coherence-spring.version>4.3.0</coherence-spring.version>
      </properties>
      ```
 
3. The `spring-boot-starter-web` artifact is included to provide the basic REST and Tomcat app server.

      ```xml
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
      ```          
      
4. Include the Coherence dependencies:

      ```xml
      <dependency>
        <groupId>com.oracle.coherence.spring</groupId>
        <artifactId>coherence-spring-boot-starter</artifactId>
        <version>${coherence-spring.version}</version>
      </dependency>
      <dependency>
        <groupId>${coherence.group.id}</groupId>
        <version>${coherence.version}</version>
        <artifactId>coherence</artifactId>
      </dependency>
      <dependency>
        <groupId>${coherence.group.id}</groupId>
        <version>${coherence.version}</version>
        <artifactId>coherence-json</artifactId>
      </dependency>
      ``` 
      
      > Note: `coherence-spring-boot-starter` is required for autoconfiguration and Coherence spring boot support, `coherence` are the core components and `coherence-json` is not technically required, but is needed to enable management API over REST.   

5. Finally, we include the plugin dependencies for `spring-boot-maven-plugin` and the `pof-maven-plugin` that automatically instruments the POJO's with Coherence POF serialization. 

      ```xml
      <build>
        <plugins>
          <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>

          <plugin>
            <groupId>${coherence.group.id}</groupId>
            <version>${coherence.version}</version>
            <artifactId>pof-maven-plugin</artifactId>
            <executions>
              <execution>
                <id>instrument</id>
                <goals>
                  <goal>instrument</goal>
                </goals>
              </execution>
              <execution>
                <id>instrument-tests</id>
                <goals>
                  <goal>instrument-tests</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
        </plugins>
      </build>
      ```
      
## Task 3: Understand the simple application config and code

In this task, we will cover the application configuration and code.

1. The file `src/main/resources/application.properties` contains any system properties we want the application to use:

      ```properties 
      # Common Coherence system properties
      # Enable POF serialization
      coherence.pof.enabled=true
      # Confine Coherence to the current server
      coherence.wka=127.0.0.1
      # Enable management
      coherence.management.http=all
      ```
  
      The three properties are explained below: 

      1. `coherence.pof.enabled=true` enable serialization using Coherence's Portable Object Format (POF). POF is a language agnostic binary format. POF is efficient in both space and time and is a cornerstone technology in Coherence. See [the documentation](https://docs.oracle.com/en/middleware/fusion-middleware/coherence/14.1.2/develop-applications/using-portable-object-format.html) for more information.
      2. `coherence.wka=127.0.0.1` restricts the Coherence cluster traffic to the localhost only. The loopback address is just used for demonstration purposes and would not be used in production.
      3. `coherence.management.http=all` this is not mandatory, but if set to `all`, enables Coherence management over REST api

2. The Customer class (`./src/main/java/com/oracle/coherence/demo/frameworks/springboot/Customer.java`) is used to store customer information in the cache. The getter/setter and object methods have been left out for brevity.

      ```java 
      @PortableType(id = 1000)
      @JsonIgnoreProperties(value = { "evolvable", "evolvableHolder", "empty" })
      public class Customer {
          @JsonProperty("id")
          private int id;

          @JsonProperty("name")
          private String name;

          @JsonProperty("balance")
          private double balance;

         public Customer() {}
      ```
      
      > Note: The `@PortableType` annotation indicates that this class should be instrumented by the `pof-maven-plugin` during build to use POF serialization.

3. Demonstration application and Controller

      The `./src/main/java/com/oracle/coherence/demo/frameworks/springboot/DemoApplication.java` is a standard Spring Boot application entry point.

      ```java
      @SpringBootApplication
      public class DemoApplication {
	      public static void main(String[] args) {
		      SpringApplication.run(DemoApplication.class, args);
	      }
      } 
      ```  
      
      The `./src/main/java/com/oracle/coherence/demo/frameworks/springboot/controller/DemoController.java` class is a JAX-RS application using Coherence.
     
      ```java
      @RestController
      @RequestMapping(path = "/api/customers")
      public class DemoController {
    
          @CoherenceCache
          private NamedCache<Integer, Customer> customers;

          @GetMapping
          public Collection<Customer> getCustomers() {
              return customers.values();
          }

          @PostMapping
          public ResponseEntity<Void> createCustomer(@RequestBody Customer customer) {
              customers.put(customer.getId(), customer);
              return ResponseEntity.accepted().build();
          }

          @GetMapping("/{id}")
          public ResponseEntity<Customer> getCustomer(@PathVariable int id) {
              Customer customer = customers.get(id);
              return customer == null ? ResponseEntity.notFound().build() : ResponseEntity.ok(customer);
          }

          @DeleteMapping("/{id}")
          public void removeCustomer(@PathVariable int id) {
              customers.remove(id);
          }
      }
      ```
      
   Note: The `@CoherenceCache` annotation injects and `NamedCache` with key of `Integer` and value of `Customer. When using basic cache operations they are the same as `Map` operations. For example `values()`, `put()` and `remove()`.

## Task 4: Working With Events

In this task we will add Coherence event listeners to respond to cache events.

1. Open the file `./src/main/java/com/oracle/coherence/demo/frameworks/springboot/controller/DemoController.java` in VisualStudio code and add the following to the end of the file.
   
      ```java
      /**
       * Listener that will be fired upon insertion of a new {@link Customer}.
       * @param event event
       */
      @CoherenceEventListener
      public void onCustomerInserted(@Inserted @CacheName("customers") MapEvent<Integer, Customer> event) {
          Logger.info(String.format("Inserted customer key=%d, value=%s", event.getKey(), event.getNewValue()));
      }

      /**
       * Listener that will be fired upon deletion of a {@link Customer}.
       * @param event event
       */
      @CoherenceEventListener
      public void onCustomerDeleted(@Deleted @CacheName("customers") MapEvent<Integer, Customer> event) {
          Logger.info(String.format("Deleted customer key=%d, old value=%s", event.getOldEntry().getKey(), event.getOldValue()));
      }

      /**
       * Listener that will be fired upon update of a {@link Customer}.
       * @param event event
       */
      @CoherenceEventListener
      public void onCustomerUpdated(@Updated @CacheName("customers") MapEvent<Integer, Customer> event) {
          Logger.info(String.format("Updated customer key=%d, old value=%s, new value=%s", event.getKey(), event.getOldValue(), event.getNewValue()));
      }
      ``` 

      Looking a one of the methods `onCustomerInserted`, this is annotated as `@CoherenceEventListener`, which indicates to 
      Coherence that this is an event listener. The `@Inserted` annotation indicates this should be run only on insert events and the `@CacheName` specifies the cache that this listener applies to.

      > Note: You will also have to add the following imports:
      > 1. `import com.oracle.coherence.common.base.Logger;`
      > 2. `import com.oracle.coherence.spring.annotation.event.CacheName;` 
      > 3. `import com.oracle.coherence.spring.annotation.event.Deleted;` 
      > 4. `import com.oracle.coherence.spring.annotation.event.Inserted;` 
      > 5. `import com.oracle.coherence.spring.annotation.event.Updated;`
      > 6. `import com.oracle.coherence.spring.event.CoherenceEventListener;`
      > 7. `import com.tangosol.util.MapEvent;`

2. In a terminal, issue the following command to build the application:

      ```bash
      mvn clean install -DskipTests
      ```

3. Then run the following command to start the application:

      ```bash
      java -jar target/springboot-1.0-SNAPSHOT.jar
      ```
             
4. In a new terminal window, run the following command to insert a customer:

      ```bash
      curl -X POST -H "Content-Type: application/json" -d '{"id": 1, "name": "Tim", "balance": 1000}' http://localhost:8080/api/customers
      ```      
   
      You should see output similar to the following showing the event listener firing:

      ```bash
      ... Inserted customer key=1, value=Customer{id=1, name='Tim', balance=1000.0}   
      ```

5. Run the following command, (note the changed balance) to update the customer:

      ```bash
      curl -X POST -H "Content-Type: application/json" -d '{"id": 1, "name": "Tim", "balance": 5000}' http://localhost:8080/api/customers
      ```      
   
      You should see output similar to the following showing the new and old values captured.

      ```bash
      ... Updated customer key=1, old value=Customer{id=1, name='Tim', balance=1000.0}, 
      new value=Customer{id=1, name='Tim', balance=5000.0}
      ```
    
6. Run the following to delete the customer:

      ```bash
      curl -X DELETE http://localhost:8080/api/customers/1
      ``` 
   
      You should see output showing the old value of the deleted customer.

      ```bash
      ... Deleted customer key=1, old value=Customer{id=1, name='Tim', balance=5000.0}
      ```

7. Start a second application server, without the http server, using the following command in a new terminal:
   
      ```bash
      java -Dserver.port=-1 -Dloader.main=com.tangosol.net.Coherence -Dcoherence.management.http=none -jar target/springboot-1.0-SNAPSHOT.jar
      ```   
   
      Once the second server starts up you should see the following message on the first server console. This indicates that the cluster has partitioned 
      the data between the two members for high availability.

      ```bash
      Partition ownership has stabilized with 2 nodes
      ```

8. Run the following command, (note the changed balance) to update the customer:

      ```bash
      curl -X POST -H "Content-Type: application/json" -d '{"id": 1, "name": "Tim", "balance": 6000}' http://localhost:8080/api/customers
      ```      
   
      You should see output similar to the following showing the new and old values captured **on both servers**.

      ```bash
      ... Updated customer key=1, old value=Customer{id=1, name='Tim', balance=1000.0}, 
      new value=Customer{id=1, name='Tim', balance=5000.0}
      ```        
   
      > Note: The reason for both members receiving the events is that each of the servers has registered for it. This is fine for responding to events,
      > but in the next lab we cover how we can write interceptors to work with or modify data before, during or after it has been added to the cluster. 

## Learn More
            
* [Using Portable Object Format](https://docs.oracle.com/en/middleware/fusion-middleware/coherence/14.1.2/develop-applications/using-portable-object-format.html)

## Acknowledgements

* **Author** - Tim Middleton
* **Contributors** - Ankit Pandey, Sid Joshi
* **Last Updated By/Date** - Ankit Pandey, November 2024