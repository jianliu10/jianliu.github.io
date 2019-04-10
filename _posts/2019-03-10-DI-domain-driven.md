---
layout: post
title:  "Case Study - unified data integration platform in enterprise data hub"
date:   2019-03-10 00:00:00 -0500
categories: tech data-integration
---

## Abstract  

This research and design article was written based on my development work experience in data integration projects in multiple enterprise data management.  

* data lake vs data warehosue *

Big data lake is a vast pool of raw data, the purpose of which is not yet defined. It is ideal for machine learning by data scientists.

Big data warehouse stores processed structure data. All of the data in a data warehouse are used for specific purposes within the organization. It is ideal for line of business (LOB) reporting and analytics by  business professionals.


* There are structured data, semi-structured and unstructured data. *

* There are three types of data delivery channels *
- batch API   
  end of day batch files and their sensors.
- REST API    
  intraday request-response JSON data
- event messaging API    
  real-time streaming data integration. example web order transactions, capital market trades.

The design document tries to probe into the following design areas:  

- use business domain driven service oriented design in enterprise data integration. A specific domain integration service integrates all types of data APIs in its business domain.
- SQL database or columnar data warehouse to store structured data  
- NoSQL database to store unstructured data  
- A back-end data service to serve data for front-end reporting and BI Analysis  

This document uses Google Cloud Platform as cloud provider.


## Traditional data processing design

The traditional data processing design is to modulize the data processing work based on their delivery channels. If a upstream domain system deliveries its different data using different types of channels, there are different data processing modules for each channel. These modules usually are developed with different programming languages and technologies. These different modules normally replicate the data processing business logics in their own context for loss coupling and development autonomy.

The drawback of this approach is that there are duplicated data processing business logics in different modules. There causes business logic discrepencies among modules.

The deployment process among different types of modules for the same business domain need to be coordinated.

## Benifits of business domain driven data integration service design  

The benefits of business domain driven data intergration service design include:

- Centralize the business logics for data cleaning, validation, transformation into a single domain data integration service component, regarless of the domain data delivery channels (batch file API, REST API, event messaging API).   
  This removes the business logic discrepencies that happen when data processing codes are scattered in multiples processes including batch file jobs, REST API process instance, event messaging process instance.

- development autonomy  
  A small group of developers working on a single business domain data integration is allowed to use the technologies that they are most productive with. 
  They can pioneer into some cutting edges technologies in  the domain data integration service without impacting other domain data integration services that are owned by other groups of developers.

- deployment agility  
  the centralization of domain business data processing logics into one place leads to the decoupling from other domain data integration services.   
  The bug fix is quick since the codes are centralized in one domain data integration service component regardless of the domain data delivery channels.   
  The deployment is agile. There will be no deployment coordination among ETL jobs instance (Talend, Informatica), Java/python REST API process instance, java/python event messaging process instance.

- The above benefits will lead to better data quality in data integration result.


## Functional Requirements

The business context is assumed to be a large global retail company with ++ billions of annual revenues. The business needs to integrate various data into a centrl master data store to support many business operation systems, finance system, report system, forcasting system, and business intelligence analytics system.

This document will use the following business operational systems as sample data sources:  

- Sales system, including store sales transactions and online orders transactions.
- Marketing Tracking system, including promotions and advertising
- Inventory Management System
- Shipment tracking system
- logs


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

Diagram - enterprise data integation SOA architecture


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


