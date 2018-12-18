---
layout: post
title:  "Case Study - FX Market Data Aggregator in a FX trading system"
date:   2016-10-01 00:00:00 -0500
categories: tech trading-system 
---

## Functional and non-functional requirements
 
I was tasked to architect and design a FX market data aggregator for a FX trading system. The system is required to have the following capabilities:
‎
1. Ability to get market data quotes concurrently from 10 ECNs.  
2.‎ Persist and aggregate the quotes
3. Disseminate the aggregate quotes to 5 clients on a real time basis
4. Ability for clients to request persisted quotes‎ directly received from ECNs
5. Ability to handle 10k messages per second.
 

This document explains MDQ (Market Data Quotes) system architecture and design, including the following:
 
1. The various components you will build.
2. The integration of various components.
3. How the system can scale to handle more venues and throughput.

## Key concepts

clients - traders, algorithm trading, or internal pricing engines  
live vs. real time  
live: ECN -> client  
real time: ECN -> MDQ, Bloomberg or Reuters -> client  


## MDQ system design - partitioned by FX instruments # 


### Components Design

**common component** - common classes and functionalities that will be used by other components in the application system.
  
 - drilldown-to-raw-quotes request class  
 - application exception classes  
 - utility classes  

**business rule component** - business rules configuration and functions  

 - aggregation business rules. aggregated quotes = BBO (Best Bid and Offer), highest bid quote and its lot size, lowest offer quote and its lot size  
 - a configuration table to map instrument keys from external (ESN, ticker) to internal unique instrument key  
 
**adapter component** - interface with external ESN, each adapter subscribe to one or more ESN. The application can start n adapter servers, each adapter server subscribe to m=(10/n) ESNs. adaper depends on common component and business rule component.

adapter functionalities:

 - server start up initializatino
	- register itself to controller, including the adapter id and its responsible ESN ids.
	- configuration. when adapter server start up, first read the config table from database, and cache the mappings in memory

 - server shutdown process
	- de-register itself from controller

 - persistence 
	- read/write raw quotes to files. one file per ESN, max file size 2GB. rolling over to next data file when a data file size reached 2GB. .1, .2, .3 etc. 

 - ESN instrument subscription function
	- create a long lasting connection to each ESN
	- subscribe to ESN for list of instruments. receive market data quotes from the m ESNs at the real time, 
	- cache the most recent market quotes in memory. cache name "RawMarketQuoteCache" 
	- asynchronously persist the market data quotes to local file system

 - pre-aggregation function
	- intervally pre-aggregate the quotes cached in memory. interval can be set every 0.1 second (configurable interval in millisecond)
		- set a thread pool type to schedulerThreadPool, which executes tasks with 0.1 second interval. set the thread pool size to 50*(number of computer CPUs)
		- create a pre-aggregation task for every instrument, submit the tasks into a thread pool. 
		- a pre-aggregation task will execute:
			- pre-aggredate using the most recent instrument market qutoes from m ESNs stored in "RawMarketQuoteCache" cache.
			- send pre-aggregated quotes to aggregator server
 
- process client drilldown-to-raw-quote request
	- receive from aggregator the drilldown request contains information such as instrument internal key, client connection information (host and listening port), total number of ESNs.
	- read the raw quotes from in-memory cache
	- create a connection to client (host and listening port). send raw quotes to client, including information {total number of ESNs,  list of {ESN id, list of quotes}}
 
 
**controller component** - interface with external clients. manage the internal adapters. controller component depends on common component and business rule component.

controller functionalities:

 - manage adapters, maintain a active list of adapters and their responsible list fo ESN ids.
 - client delivery
	- process client subscription requests. create a long lasting connection to each client.
	- create a thread pool with size 5. each client has one delivery thread.
	- each thread is responsible for delivering aggregated market qutoes to one client. each thread internally holds a queue
	- each thread FIFO queued aggregated market quotes, deliver to end client.

 - receive pre-aggregated quotes from adapters. cache the most recent pre-aggr market quotes in memory. cache name "PreAggrMarketQuoteCache"
 - aggregation process
	- intervally aggregate the quotes cached in memory. interval can be set every 0.1 second (configurable interval in millisecond)
		- set a thread pool type to schedulerThreadPool, which executes tasks with 0.1 second interval. set the thread pool size to 50*(number of computer CPUs)
		- create a aggregation task for every instrument, submit the tasks into a thread pool. 
		- a aggregation task will execute:
			- aggredate using the most recent instrument pre-aggr market qutoes from m ESNs stored in "PreAggrMarketQuoteCache" cache.
			- put aggregated market quotes into each clientDeliveryThread internal queue.

 - pre-process client other types of requests, such as
	- receive clients request to drill down the aggregated quotes to raw quotes. drilldown request contains information such as instrument internal key, client connection information (host and listening port)
	- enrich drilldown request by adding total number of ESNs. forward enriched drilldown requests to all adapters. 
 
 
### Batch jobs ##
 - archive T-1 raw qutoes files; 
 
### Advanced features ##
 - failover. avoid single point of failure
 

### Architecture Diagram

![architecture](/images/architecture.gif)  

### Component Diagram

![component-diagram](/images/component-diagram.gif)  

### Deployment Diagram

![deployment-diagram](/images/deployment-diagram.gif)  

### Process Flow
![process-flow-1](/images/process-flow-1.gif)  

![process-flow-2](/images/process-flow-2.gif)  

### Persistence

![persistence](/images/persistence.gif)  


### Concurrency pseudo code


![concurrency-1](/images/concurrency-1.gif)  
![concurrency-2](/images/concurrency-2.gif)  
 
## Technology Stack

![technology-stack](/images/technology-stack.gif)

## Enhancement
To enhance the speed of transferring real-time FX market data quotes to the clients, a solution is to replace messaging Queue/Topics design with Multicast socket design.

 






