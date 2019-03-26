---
layout: post
title:  "Case Study - domain driven service oriented design in data integration"
date:   2019-03-10 00:00:00 -0500
categories: tech data-integration
---

## Abstract  

This research and design paper was written based on my development work experience in data integration projects in multiple enterprise master data management.  

There are structured data and unstructured data.

There are three types of data APIs:
- batch API - end of day batch files and their sensors.
- REST API - intraday request-response
- event messaging API - real-time streaming data integration. example web clicks, capital market trades.

The design document tries to probe into the following design areas:
- use domain driven service oriented design in data integration. A service integrates all types of data APIs in one domain.
- SQL database or columnar data warehouse to store structured data
- NoSQL database to store unstructured data
- A data service for reporting and BI Analysis

At the time I wrote this document, I used Google Cloud Platform as PAAS and SAAS. Thus you will find Google Cloud specific products and services in this document.


## Functional Requirements

The business context is assumed to be a large global retail company with ++ billions of annual revenues. The business needs to integrate various data into a centrl master data store to support many business operation systems, finance system, report system, forcasting system, and business intelligence analysis system.

This document will use the following business operational systems as sample data sources:  

- MPTS
  The Merchandizing Promotion Tracking system. Both end of day data feed for weekly promotions, and intraday data feed for time limited flash sales and super sales  
- SOD
  Sales of Day. Intraday data feed.
- OMS
  Order Management system. real-time order data integration  
- WMS
  Warehouse Management System. real-time inventory data integration


The data source systems supply the data in the channels:
- end of day data feed via batch file channel
- intraday data feed via REST-API channel
- real-time streaming data feed via event messaging middleware channel


The master data store data are to be fed back or used in the systems:
- operational systems
- finance system
- reporting system
- forcasting system
- business intelligence system


## Non-Functional Requirements

- fault tolerant 
  automatic failure-retries to avoid sporadic network unstability.   
- failover 
  to avoid single point of failure  
- scale out 
  to support load balancing and failover.
- integration service instance registry
- log aggregation and monitoring     
- application monitoring 
  monitor the runtime health, CPU, memory, disk usage of the application processes.    
- error report 
  near time error/exception notification to a support group.   


# Architecture

Diagram - enterprise data integation SOA architecture


# Technical Design


## batch file sensors
 

## REST-API 

### OpenAPI 2.0 Protocol


### Postman test tool


## real-time event messaging middleware - Kafka


## fault tolerance - failure and retry


## integration service instance scalability, load balancing, and fail over


## integration service instance registry 




## log aggregation and monitoring




## real-time error notification


# Code Snippets

the codes are proprietory


# Technology Stack

## Data integration technology stack

- Python, Java
- REST-API, OpenAPI2, JSON, 
- Shell Script
- Google Cloud Platform and Services
	- cloud storage, 
	- cloud Pub/Sub (Kafka), 
	- cloud BigQuery columnar data warehouse, 
	- cloud logstash
	- cloud appendine-flex
	- cloud kurbernetes cluster of compute engines
	- cloud dataflow or Spark

	
## Report and BI technology stack

- Tableau
- Java, Spring-boot, Spring for data service back-end
- AngularJS for front-end customized reports that cannot be handled by tableau

	
# Deployment on Google Cloud


# Conclusion


