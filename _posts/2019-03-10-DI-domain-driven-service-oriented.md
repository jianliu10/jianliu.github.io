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
- SQL database or columnar data warehouse to stored structured data
- NoSQL database to stored unstructured data
- A data service for reporting and BI Analysis

At the time I wrote this document, Google Cloud Platform is my study interest. Thus you will find Google Cloud specific products and services in this document.


## Functional Requirements

The business context is assumed to be a large global retail company with ++ billions of annual revenues. The business needs to integrate various data into a centrl master data store to support many business operation systems, finance system, report system, and business intelligence analysis system.

This document will use the following business operational systems as sample data sources:  
SAM 		   : the Merchandizing Sales and Marketing transactional system  
PromoLocation  : the retail store promotion location management system  
OMS 		   : Online Order Management system  
WMS			   : Warehouse Management System
web clicks

The data source systems supply the data in the channels:
- end of day data channelled via batch files
- intraday channelled via REST-API
- real-time streaming data channelled via event messaging middleware


The data are to be fed back or used in the systems:
- operational systems
- finance system
- reporting system
- business intelligence system


## Non-Functional Requirements

fault tolerant - automatic failure-retries to avoid sporadic network unstability.   
failover - to avoid single point of failure  
scale out - to support load balancing and failover.
service registry
log aggregation and monitoring     
application monitoring - monitor the runtime health, CPU, memory, disk usage of the application processes.    
error report - near time error/exception notification to a support group.   


# Architecture

Diagram - enterprise data integation SOA architecture

# Design

## batch file sensor 

## REST-API 
### OpenAPI 2.0 Protocol

### Postman test tool

## event messaging middleware - Kafka

## fault tolerance - failure and retry

## scale out

## failover

## service registry

## log aggregation and monitoring

## near-time error notification



# Code Snippets

the codes are proprietory

# Technology Stack
Java, JEE, Spring-boot, Spring, Python, REST-API, OpenAPI, JSON, Shell Script
Google Cloud Platform: cloud storage, cloud Pub/Sub (Kafka), appengine-flex, BigQuery columnar data warehouse, 

# Deployment

# Conclusion

