---
layout: post
title:  "Case Study - batch data integration using Java + Spring-boot + Spring-batch on Google Cloud"
date:   2018-12-01 00:00:00 -0500
categories: data-integration batch
---

# Functional Requirement

I was tasked to architect, design, program and implement a batch data integration framework that loads upstream data into Google Cloud BigQuery data warehouse. This document explains the key aspects of my development work.

The upstream system will extract data into files and upload into Google Cloud Storage.

The data integration batch job will read the data files from Cloud Storage, transform the data, load the data into Cloud BigQuery staging tables and data warehouse tables, and finally generate reports into report tables.

The data integration batch job will be deployed and run on Google Cloud Platform. 

# Non-Functional Requirement
fault tolerant (auto fail-retry),  
failover,  
log monitoring,  
process metrics,  
error report    


# Architecture

# Design

# Code Snippets

# Deployment

# Conclusion

