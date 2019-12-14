---
layout: post
title:  "Case Study - Spark 2 as unified data processing platform in enterprise big data hub"
date:   2019-03-10 00:00:00 -0500
categories: tech data-integration
---

## Abstract  

The document discusss how Spark 2 is used as a unified data integration and analytics platform in enterprise big data warehouse and/or big data lake.

This document was written based on my development work experience in data integration projects in multiple enterprise data hub. 


## Background

**data lake vs data warehosue**

Big data lake is a vast pool of raw data, the purpose of which is not yet defined. It is ideal for machine learning by data scientists.

Big data warehouse stores processed structured data and semi-structured (JSON, XML) data. All of the data in a data warehouse are used for specific purposes within the organization. It is ideal for reporting and analytics by LOB business professionals.


**There are structured data, semi-structured and unstructured data**

**There are three types of data delivery channels**
- batch file API for end of day data intake
- REST API, SOAP API for synchronous real-time data intake, such as online order transactions
- event messaging API for asynchronous real-time streaming data intake. Example logs. 


## Spark data processing platform  

Spark is a general purpose computing platform, which is fast and run on cloud. Spark is used as unified data processing and analytics engine. 

Spark support :
- integrate with varioud sources and sinks, including csv, text, json, jdbc, kafka, Apache Flume
- batch and real-time data processing
- structured, semi-structured, and non-structured data


## Traditional data processing

The traditional data processing design is to modulize the data processing work based on their delivery channels. If a upstream domain system deliveries its different data using different types of channels, there are different data processing modules for each channel. These modules usually are developed with different programming languages and technologies. These different modules normally replicate the data processing business logics in their own context for loss coupling and development autonomy.

The drawback of this approach is that there are duplicated data processing business logics in different modules. There causes business logic discrepencies among modules.

The deployment process among different types of modules for the same business domain need to be coordinated.


## Benifits of unified data processing platform  

The benefits of unified data processing platform include:

- Centralize the business logics for data cleaning, validation, transformation into a single domain data integration service component, regarless of the domain data delivery channels (batch file API, REST API, event messaging API).   
  This removes the business logic discrepencies that happen when data processing codes are scattered in multiples processes including batch file jobs, REST API process instance, event messaging process instance.


## Functional Requirements

The business context is assumed to be a large global retail company with ++ billions of annual revenues. The business needs to integrate various data into a centrl master data store to support many business operation systems, finance system, report system, forcasting system, and business intelligence analytics system.

This document will use the following business service applications as sample data sources:  

- Catalog service. 
  Changes infrequently. extracted as end of day REST API JSON data.
- Marketing service, including weeky promotions, daily super sales, and hourly flash sales
  Changes weekly and adjusted daily. end of day batch file
- Orders service. online orders transactions
  real-time streaming data
- Inventory service
  real-time streaming data
- logs from all service applications
  real-time streaming data
- search service


The data source systems supply the data in the channels:  
- end of day data feed via batch file channel
- intraday data feed via REST-API channel
- real-time streaming data feed via event messaging middleware channel

The master data store data are to be fed back or used in the systems:
- operational systems
- finance system
- reporting analytics
- forcasting / predictive analytics
- machine learning system


## Non-Functional Requirements

- fault tolerant   
  automatic failure-retries to avoid sporadic network unstability.   
- fail over   
  to avoid single point of failure  
- scalability  
  start up multiple instances of same integration service to support load balancing and fail over.
- integration service instance registry  
- log aggregation and monitoring       
- application monitoring   
  monitor the runtime health, CPU, memory, disk usage of the application processes.    
- error report   
  near time error/exception notification to a support group.   


## Architecture

Diagram - enterprise big data processing architecture


## Technical Design


### batch file sensors
 

### operational system REST-API 


### real-time event messaging middleware - Kafka


### fault tolerance - failure and retry


### integration service instance scalability, load balancing, and fail over


### integration service instance registry 




### log aggregation and monitoring




### real-time error notification


## Code Snippets

the codes are proprietory


## Technology Stack

### Data integration technology stack

- Scala, Java, Python, shell scripts
- Spark, Hive, HDFS
- REST-API
- Google Cloud Platform:
	- cloud storage, 
	- cloud Pub/Sub (Kafka), 
	- cloud Dataproc cluster preinstalled with Spark and Hadoop ecosystem
	- cloud BigQuery columnar data warehouse, 
	- cloud logstash
	- cloud appengine-flex, compute engine
	- cloud kurbernetes container management
	- cloud dataflow 

	
### Report and BI technology stack

- for big data warehouse, use 
	- Tableau
	
- for big data lake, use
	- Spark SQL for structure analytics
	- Spark ML machine learning
	
	
## Deployment on Google Cloud


## Conclusion


