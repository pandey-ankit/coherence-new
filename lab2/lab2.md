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

2. In the same terminal, issue the following command to start the application:

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
   
   > Note: You can verify the customers cache  by using VisualVM as we did in the previous lab. Ensure that you close the tab you opened with the previous process and double-click on the new (`springboot-1.0-SNAPSHOT.jar`) process.
   
   use `CTRL-C` to quit the Spring Boot application before you move to the next task.

## Task 2: Title for task 2



## Task 3: Title for task 3



## Task 4: Title for task 3
       


   
## Learn More
            


## Acknowledgements

* **Author** - Tim Middleton
* **Contributors** - Ankit Pandey, Sid Joshi
* **Last Updated By/Date** - Ankit Pandey, November 2024