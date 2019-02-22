---
layout: post
title:  "Case Study - batch data integration using Java + REST-API + Spring-boot + Spring-batch on Google Cloud"
date:   2018-12-28 00:00:00 -0500
categories: tech data-integration
---

## Functional Requirement

I was tasked to architect, design, program and implement a batch data integration framework that loads upstream retail business data into Google Cloud BigQuery columnar data warehouse. This document explains the key aspects of my development work.

The upstream system will extract data into files and upload into Google Cloud Storage.

The data integration batch job will read the data files from Cloud Storage, transform the data, load the data into Cloud BigQuery staging tables and data warehouse tables, and finally generate reports into report tables.

The data integration batch job will be deployed and run on Google Cloud Platform. 


## Non-Functional Requirement

fault tolerant - automatic failure-retries    
failover - to avoid single point of failure  
log monitoring     
batch process metrics - monitor the health, CPU, memory, disk usage states of the data integration processes.    
error report - near time error/exception notification to a support group.   


# Architecture 

# Design

# Code Snippets

# Task Scheduler

# Deployment

# Conclusion

