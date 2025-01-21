# Working With Events

## Introduction

In this lab we extend the basic application to respond to various types of events.


Estimated time: 15 minutes

### Objectives

In this lab, you will:
       
* Add Map Events on the customers cache to show when data has been modified
* Add Entry Events, to modify data as it is being updated


### Prerequisites
     
This lab assumes you have:

* An Oracle Free Tier(Trial), Paid or LiveLabs Cloud Account
* You have completed:
  * Lab: Prepare Setup (Free-tier and Paid Tenants only)
  * Lab: Initialize Environment
  * Lab: Getting Started with Coherence and Spring


## Task 1: Working With Map Events

In this task we will add Coherence event listeners to respond to cache `MapEvents`.  
A `MapEvent` observer method is a method on a Spring bean that is annotated with `@CoherenceEventListener`. 
The annotated method must have a void return type and must take a single method parameter of type `MapEvent`, 
typically this has the generic types of the underlying map/cache key and value.

These observe methods, can respond to insert, update or delete events and contain the new value for insert, the old and new value
for updates and the old value for deletes. You can also filter events based upon a where clause.

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
   
      /**
       * Listener that will be fired upon update of a {@link Customer} where the new balance < 1000.
       * Note: the 1000.0d indicates to Coherence this is a double value.
       * @param event event
       */
      @CoherenceEventListener
      @WhereFilter("balance < 1000.0d")
      public void onCustomerUpdatedLowBalance(@Updated @CacheName("customers") MapEvent<Integer, Customer> event) {
          Logger.info(String.format("Low balance for customer, Updated customer key=%d, value=%s", event.getKey(), event.getNewValue()));
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
    
6. Run the following to change the balance to $500. This will then cause the low balance listener to trigger based upon the where clause:

      ```bash
      curl -X POST -H "Content-Type: application/json" -d '{"id": 1, "name": "Tim", "balance": 500}' http://localhost:8080/api/customers 
      ``` 
   
      You should see output showing two event listeners firing, one for the low balance and one for the general update. 

      ```bash
      Low balance for customer, Updated customer key=1, value=Customer{id=1, name='Tim', balance=500.0}
      Updated customer key=1, old value=Customer{id=1, name='Tim', balance=5000.0}, new value=Customer{id=1, name='Tim', balance=500.0}
      ```   
   
      > Note: When using where clauses, you should consider adding indexes on fields in the where clause for more efficient access. See the [Documentation](https://docs.oracle.com/en/middleware/fusion-middleware/coherence/14.1.2/develop-applications/querying-data-cache.html#GUID-75243AC5-8FF4-4485-8754-25FE8B3A8101) for more information.
    
7. Run the following to delete customer 1:

      ```bash
      curl -X DELETE http://localhost:8080/api/customers/1 
      ``` 
   
      You should see output showing the old value of the deleted customer.

      ```bash
      Deleted customer key=1, old value=Customer{id=1, name='Tim', balance=500.0}
      ```

8. Start a second application server, without the http server, using the following command in a new terminal:

      ```bash
      java -Dserver.port=-1 -Dloader.main=com.tangosol.net.Coherence -Dcoherence.management.http=none -jar target/springboot-1.0-SNAPSHOT.jar
      ```   
 
      Once the second server starts up you should see the following message on the first server console. This indicates that the cluster has partitioned 
      the data between the two members for high availability.

      ```bash
      Partition ownership has stabilized with 2 nodes
      ```

9. Run the following command, (note the changed balance) to update the customer:

      ```bash
      curl -X POST -H "Content-Type: application/json" -d '{"id": 1, "name": "Tim", "balance": 6000}' http://localhost:8080/api/customers
      ```      
   
      You should see output similar to the following showing the new and old values captured **on both servers**.

      ```bash
      ... Updated customer key=1, old value=Customer{id=1, name='Tim', balance=1000.0}, 
      new value=Customer{id=1, name='Tim', balance=5000.0}
      ```        

      Note: The reason for both members receiving the events is that each of the servers has registered for it. This is fine for responding to events,
      but in the next lab we cover how we can write interceptors to work with or modify data before, during or after it has been added to the cluster. 
    
## Task 2: Working With Entry Events
    
An EntryEvent is emitted when data is mutated on a cache. These events are only emitted on the storage enabled member 
that is the primary owner of the entry that is mutated.

For example, the onEvent method below will receive entry events for all caches.

```java
@CoherenceEventListener
public void onEvent(EntryEvent event) {
    // TODO: process the event
}
```

There are a number of different EntryEvent types:

* Inserting - an entry is being inserted into a cache, use the @Inserting annotation
* Inserted - an entry has been inserted into a cache, use the @Inserted annotation
* Updating - an entry is being updated in a cache, use the @Updating annotation
* Updated - an entry has been updated in a cache, use the @Updated annotation
* Removing - an entry is being deleted from a cache, use the @Removing annotation
* Removed - an entry has been deleted from a cache, use the @Removed annotation

To restrict the EntryEvent types received by a method apply one or more of the annotations above to the method parameter. For example, the method below will receive Inserted and Removed events.
  
```java
@CoherenceEventListener
public void onEvent(@Inserted @Removed EntryEvent event) {
    // TODO: process the event
}
```

1. Add an event listener that will ensure that a customers name is always uppercase.

   Open the file `./src/main/java/com/oracle/coherence/demo/frameworks/springboot/controller/DemoController.java` in VisualStudio code and add the following to the end of the file.
    
   
   

       


   
## Learn More
            


## Acknowledgements

* **Author** - Tim Middleton
* **Contributors** - Ankit Pandey, Sid Joshi
* **Last Updated By/Date** - Ankit Pandey, November 2024