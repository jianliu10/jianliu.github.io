---
layout: post
title:  "Java-Spring Technical notes - 2019"
date:   2019-10-01 00:00:00 -0500
categories: tech-java-spring
---

# 2019 technical notes #

## 2019 IT job market trend

Enterprise Java applications:
- Kafka 
- Spring Core, Spring Boot, Spring WebMVC, Spring Security, Apache Camel
- Database development and tools with SQL, ORM, Flyway, OLTP Database Tuning, etc.
- UNIX (Linux) environment and scripting (bash, shell and python)
- implementing integration solutions with RESTful Web Services, swagger 2, JSON schema
- AWS development using EC2, EB, IAM, S3, SDK, CLI, Code Deploy, Code Commit, Lambda/Step Functions, Cloudfront, Cloudwatch, etc.
- testing automation using Selenium, Cucumber, and Junit is preferred
- NoSQL Database experience
- Spring Batch experience

big data applications:
- Spark


## the art of scalability

- service layer: functional decomposition into micro-services, 
- computing layer: controller-worker partitioned parallel computing.
- storage layer: partitions, sharding
- replica: storage partition replication (leader, follower), micro-service replication (load balancing)

## four types of distributed system architecture

- modern three tiered architecture (SPA, api-gateway, micro-services)
- sharding (Storage - data storage physical nodes, partitions, replica, key range/list based partitioning, columnar partitioning)
- lambda serverless architecture (Computation - small annonymous function that is short-lived run. )
- reactive programming  (event messaging, data streaming)

## design patterm - event sourcing table

To bridge transactional table data change to a messaging table

for example, we add a record in a business table, then publish a event msg for the record to a topic. The db table write and topic msg publish need to be in a single distrubuted transaction. But we cannot use 2PC in modern distributed system. 2PC is not reliable.

Instead, we still need to use local transaction. so the solution is event sourcing.

**event sourcing solution:**    
use db Event Table as **message queue**, this make a local transaction across multiple tables (business table and Event table).  use db triggers to append events to db Event table.  
another process read events from the event table and publish to Topics.  
Here the Event table is the source of events.

**Use case:**  
Order lifecycle: Order created, Order cancelled, Order shipped, Order received.  
there is one Order transactional table, a order record can be inserted, updated, deleted.
There is another order event table, every order event is appended to event table. This event table is used as message QUEUE - append only, no update, no delete. 

## design patterm - CQRS - Command Query Responsibility Segregation  

Command - upsert, delete operations;  
Query - read operations

use cases:  
MongoDB cluster contains a write node and a read node. upsert and delete requests are sent to the write node. the data change is replicated to the read node. Query requests are sent to the read node.


## mode based hashing vs consistent hashing  

https://www.toptal.com/big-data/consistent-hashing

### mode based hashing  

It is used in single local process.  
hash table entry -> bucket, where a bucket is a linked list of (key, value) pairs.  
hash table entry = HashFunc(key) mod N. where N is the hash table size

Problem of mode based hashing: rehashing every keys when the hash table resized. 
solution: consistent hashing. 

### consistent hashing algorithm  

Hash ring.  
In general, only k/N keys need to be remapped when k is the number of keys and N is the number of servers

It is used in distributed system, including sharding, load balancing, distributed caches.  
consistent hash ring is used in Cassandra, Memcached and Redis.


