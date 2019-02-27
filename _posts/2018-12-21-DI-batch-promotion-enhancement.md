---
layout: post
title:  "Case Study - Retail Data Integration Automation and Enhancement in DWH"
date:   2019-2-22 00:00:00 -0500
categories: tech data-integration
---

## Background
I was tasked to enhance an 10+ years aged existing data integration component PROMO-DI in a data warehouse. The goal of the enhancement is not only to add new business features for this one-time project, but also to automate the data integration process for the future.

This document describes the as-of state of PROMO-DI data integration component in DWH, and its automation & enhancement design proposals.


## Abbreviations  

DWH 		- Data Warehouse   
SAM 		- the Merchandizing Sales and Marketing transactional system  
PROMO-DI 	- the promotion data integration component in DWH  
PromoTree 	- the retail store promotion management system  
ORM 		- Online Order Management system  


## As-of state of existing data integration component

PROMO-DI component in DWH provides data for a few key business functions:
- provide fact-dimensional data for Reports and Business Intelligence Analysis.
- provide flat store&item level promotional data for downstream retail store promotion management system. 
- provide flat corporate&item level promotional data for downstream Online Order Management system.


In the first half of this document, I will explain four challenges in PROMO-DI data integration component. These challenges are common to most of data warehouse systems. In the second half of the document, I will propose solutions to these challenges. 

The challenges are described in the following.

### Challenge-1: manual process to add or change dimensional data in DWH 

   The existing system adopts an approach that is commonly used by developers in data warehouse system. Since the DWH developers are good at writing SQLs, they wrote the individual SQL insert/update/delete DML statments to add/change dimensional data in DWH dimensional tables. This approach is simple and works great when the number of new or changed dimensional records is small (less than 20 records). 

   However, when the number of new or changed dimensional records is big (hundreds of records), this approach is heavy coding labored, non-visualized, hard to trace the DML statments in script.

   
   ![Load dimensional data using manual process](/images/PromotionDI-EnhancementDesign/loadDimData-manualProcess.jpg)

   
### Challenge-2: Use visual ETL tool to process complex data transformation logic

   The DWH system uses Informatica jobs to extract, transform, and load Fact data. Visual ETL tools like Informatica, Talend work great when the data transformation logic is simple and straightforwd. 

   However, the visual ETL tools become unwieldly when the data transformation logic is complex.

   The visual ETL tools become unwieldly in complex data transformation logic. For a complex transformation logic, which could have been solved with several lines of Python or Java codes in clear logic flow, it will take many work-around steps in ETL tool to develped. The end result of using visual ETL tool to develop complex transformation logic is a visual flow chart that is highly complex, confusing, difficult to be undertood by other developers and thus difficult to maintain and enhance in the future. With the time going by, even the oroginal developers will have difficulty understanding the highly complex visual flow chart themselves.  
  
### Challenge-3: Use visual ETL tool to process extra large of data traffic volumn without sub-partitions 

   With today's network speed, visual ETL tool like Informatica and Talend can handle a few millions of fact records with tolerable time performance. 
   
   However, when there are a few hundreds of millions thus tens of billions of bytes of fact data, it will take hours to read the fact data from database, transform, and load them back to database. The time performance is untolerable. The issues lie in a few areas: 1) reading and writing extra large number of fact records from/to database causes IO bandwidth bottleneck; 2) the large memory cache size and concurrent multi-threaded procesing on ETL jobs demands a machine with high CPU capacity and high RAM capacity. This will be costly in hardware investment. 3) the fact data is not sub-partitioned for parallel processing in a grid cluster.
  
   For example, in the current promotion data integration system, it generates promotion data not only by monthly promotion turn, also by weekly, and by daily. The reason of generating weekly and daily promotion data is to provide a sales revenue weekly and daily drilldown view for report and BI analysis. In each monthly promotion turn, there are avg 170k promotion items records, avg 467k promotion locations records. By multiplying by 4, there are 680k and 1,840k weekly records per month. By multiplying by 30, there are 5,100k and 14,010k daily records per month. Assume each record avg 500 bytes, the size of promotion data generation can reach up to tens of billions of bytes in a fresh daily batch job run which will generate not only look-forward monthly promotion turn data, but look-forward 4 weeks and 30 days drilldown promotion data.

  
### Challenge-4: hard coded codes/types mapping logic from upstream transactional systems to DWH report system
   
   It was initially convenient to hard code codes/types mapping logic in ETL transformation when the number of mappings is small. With the upstream transactional system adds more and more codes/types, the hard coded mapping logic becomes a constant coding labor. Every time there is a new or changed promotion program or promotion location in upstream transactional system, the hard coded mapping logic need to be revisited and changed accordingly. Hard coded mapping logic also makes the visual ETL flow chart bloated with several branches of data process flows that only differs in codes/types mapping logic. 
   
   For example, promotion data integation component maps promotion location codes used in updtream SAM system to the promotion subtypes used in DWH report system. There are different codes/types mapping logics depending on the promotion types. 
    

## Design of promotion data integration automation and enhancement 

### Automate the integration of new and changed dimensional data into DWH 

This session proposes two solutions to Challenge-1 described in the above. 

#### Basic enhancement - Using business client CSV input files and PL/SQL Stored Procedures

When adding or chaning a large number of dimensional data records, The current approach is to write individual insert/update/delete SQL DML statements for each records to directly load into multiple inter-related final dimensional tables in DWH with surrogate IDs (integer) as foreign key references. Assume there are 50 of new or changed programs, and 5 final fact tables to populated, there will be, <number of new or changed programs> * <number of final dimensional tables>, total 250 SQL DML statments to hand written by developers.

The enhanced approach is to use business client provided CSV input files. The business client provides CSV input files containing new or changed business records, a **generic python script** is developed to read the business CSV input files, load the raw file contents to staging tables in DWH. PL/SQL stored procedures are developed to read dimensional records from staging tables, transform them, and load them into the multiple inter-related final dimensional tables in DWH.

Assume there are 50 of new or changed programs, the enhanced approach will requires business client to provide a csv file with 50 lines of records.

We can see that using this business CSV input file approach, it still invoves developers' manual work to receive a new csv file from business client, verify the data format, and run a shell script that executes the python script and the PL/SQL SPs.

![Load dimensional data using CSV input file + PL/SQL Stored Procedures](/images/PromotionDI-EnhancementDesign/loadDimData-csvInputFile.jpg) 


#### Advanced enhancement - Using business client notifications, actions and approval workflow

This advanced enhancement implements a fully automatic workflow that involves business client only. The approach will not involve the developers in the on-going dimensional data population, maintenance and governance. 

I have rarely seen this advanced enhancement in data warehouse systems. The rarity is mainly because the development of automatic workflow requires technical skills that go beyond a typical DWH DI team skill sets. The workflow is actually an web application development. 

The high level design of the workflow is as the following:

1. developed a standalone web application that running forever. It is not a scheduled job.

2. the scheduled ETL batch jobs will extract the dimensional information from upstream data feeds, and populate the DWH dimensional tables with as-it-is data quality or even blank fields values which are necessary in DWH report system. If the upstream data feed introduces a new programs, the ETL jobs will add a new records to  dimensional table with many key field unpopulated.

3. the web app polls the dimensional tables in DWH a few times a day. If it detects any dimensional records with invalid field values or missing required field values, it will put the invalid dimensional records into a notification queue.

4. the web app sends email notification to the business client about the new or invalid dimensional records.

5. business client go to a browser to log in to web app
	- the business client fills or corrects the dimensional records. submit and approve the changes. The changed dimensional records will be saved to DWH staging tables with FLAG column value set as "changed".
	- the business client can also query an existing valid dimensional records, change some fields to new values, submit and approve the change. The changed dimensional records will be saved to DWH staging tables with FLAG column value set as "changed".
	- the business client cal also add a new dimensional records. submit and approve the new records. The changed dimensional records will be saved to DWH staging tables with FLAG column value set as "new". 

6. upon completion of daily new or changed dimensional records, the business client clicks a button to trigger the execution of the PL/SQL stored procedures, which will read the new or changed dimensional data from staging tables, transform them, and load them into the multiple inter-related final dimensional tables in DWH.


![Automatic dimensional data governance using notification and approval workflow](/images/PromotionDI-EnhancementDesign/loadDimData-notificationApprovalWorkflow.jpg)


### Use python to process complex data transformation logic

Python has built-in language features for data processing. Python ecosystem includes data processing packages like Pandas and cx_Oracle to handle data extraction, transformation and loading. Using python, developers can write very complex transformation logic in a small chunk of codes. The benefits is an easy to understand and easy to maintain and support code base.

when the data transformation logic is complex, where visual ETL tool becomes unwieldy, I suggest writing a python script to process the transform logic.  Visual ETL tools like Informatica or Talend provides a command line UI component to invoke python scripts.


### process extra large of data traffic volumn (hundreds of millions of fact records)  

I suggest using PL/SQL stored procedures to read/process/write extra larget of data. This avoids the IO network bottleneck, which is caused by the reading/writing extra large volumn of data to/from database to ETL client side jobs. PL/SQL will keep reading and writing data local to the database server.


### Use mapping configuration tables instead of hard coded codes/types mapping logic

Data integration process always need to map codes/types from upstream transactional systems to DWH report system. The data quality of dimensional data is very import for report and BI analysis. 

For example, in promotion DI system, there is mapping logic to map from SAM system promotion location codes to DWH system promotion sub-type codes. The sample hard coded mapping snippets are as the following. each similar mapping logic appears in two places, one place is in corporation items transformation phase , the other place is in corporation locations transformation phase. 

We can see some mappings of codes/types are one-to-one straight forward. other mappings are based on certain field value patterns in a dimension record. 

To handle field value pattern based mappings, patterned based regular expressions can be configured in the mapping configuration table. However, this belongs to advanced level software engineering. I hope ETL tool vendor complany like Informatia and Talend can provide built-in feature for pattern based mapping configurations in their product in the future. Before Informatica or Talend provodes built-in feature for this need, hard coded mappings might be a simpler feasible solution since it avoids over-enginnering.


> Challenge-4: Sample hard coded mapping snippets in existing ETL Informatica job:
> 
> For promotionType = 'EndAisle'
> 
> - during promotion items data integration phase, 
> 
> 			IIF(VALUE31 = 'HD', 'HD',
> 				 IIF(VALUE31 = 'GL', 'GL',
> 				 IIF(VALUE31 = '12P', 'EA12P',
> 				 IIF((INSTR(VALUE31,'FEM',1,1)) <0 , VALUE31,
> 				 IIF((INSTR(VALUE31,'PA+B',1,1)) <0,'EA' || LPAD(VALUE31,6,'0'),
> 				 IIF((INSTR(VALUE31,'PA',1,1)) <0, 'EA'||LPAD(VALUE31,4,'0'),
> 				  IIF((INSTR(VALUE31,'PB',1,1)) <0,'EA'||LPAD(VALUE31,4,'0'),
> 				  IIF((INSTR(VALUE31,'A+B',1,1)) <0, 'EA'||LPAD(VALUE31,5,'0'),
> 				  IIF((INSTR(VALUE31,'A',1,1)) <0, 'EA'||LPAD(VALUE31,3,'0'),
> 				  IIF((INSTR(VALUE31,'B',1,1)) <0, 'EA'||LPAD(VALUE31,3,'0'),
> 				  IIF((INSTR(VALUE31,'VL',1,1)) <0, 'EA'||VALUE31,
> 			IIF((INSTR(VALUE31,'CE',1,1)) = 1, VALUE31,
> 			IIF((INSTR(VALUE31,'EZ',1,1)) =1, VALUE31,
> 			IIF((INSTR(VALUE31,'S',1,1)) =1, VALUE31,
> 			IIF((INSTR(VALUE31,'W',1,1)) =1, VALUE31,
> 			'EA'||LPAD(VALUE31,2,'0'))))))))))))))))
> 
> - during the promotion locations data integration phase:
> 
> 			DECODE(TRUE, 
> 			INSTR(LOCATION_CODE31,'P') <0, lpad(LOCATION_CODE31,3,'0'),
> 			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'GL')  = 1, LOCATION_CODE31,
> 			LTRIM(RTRIM(UPPER(LOCATION_NAME31))) = 'GO LOCAL','GL',
> 			UPPER(LTRIM(RTRIM(LOCATION_CODE31))) = 'HERO','HD',
> 			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'VL') = 1, 'EA'||LOCATION_CODE31,
> 			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'FEM') 0, LOCATION_CODE31, 
> 			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'CE')  = 1, LOCATION_CODE31,
> 			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'EZ')  = 1, LOCATION_CODE31,
> 			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'S')  = 1, LOCATION_CODE31,
> 			INSTR(UPPER(LTRIM(RTRIM(LOCATION_CODE31))),  'W') = 1, LOCATION_CODE31,
> 			'EA'||lpad(LOCATION_CODE31,2,'0'))
> 
> 			DECODE(TRUE,
> 				INSTR(LOCATION_CODE34,'P') <0, 'EA'||lpad(LOCATION_CODE34,3,'0')||'B', IN (LOCATION_CODE34 ,'4','8','9','12',0) = 1, 'EA'||lpad(LOCATION_CODE34,2,'0')||'B',
> 				'EA'||lpad(LOCATION_CODE34,2,'0'))
> 
> 			DECODE(TRUE,
> 			INSTR(LOCATION_CODE5,'P') <0, 'EA'||lpad(LOCATION_CODE5,3,'0')||'A+B',
> 			'EA'||lpad(LOCATION_CODE5,2,'0')||'A+B')
> 
> For promotionType = 'ProductExtender', then
> 
> 	DECODE(UPPER(LTRIM(RTRIM(DESCRIPTION))),
> 	       'SHELF EXTENDERS - COMMUNITY', 'PECM',
> 	       'SHELF EXTENDERS - GREEN','PEGR',
> 		   'SHELF EXTENDERS - REGULAR', 'PERG', 
> 		   'DISCOVERY','PEDI')
> 
> For promotionType = 'AirMiles', then 	
> 
> 		IIF(ACRONYM34='BAM', 'AMRG',IIF(ACRONYM34='BBAM', 'AMBB','AMSB'))
> 		
> For promotionType = "LimitedSales", then
> 
> 		IIF(VALUE39 = 'Super Sale LTO', 'LS',
> 		IIF(VALUE39 = 'Flash Sale', 'LF', NULL))
> 		
> 		
> 	

## Technology Stack 

PL/SQL Stored Procedures   
Python    
Informatica  
Shell Script
(Optional) business client notification and approval workflow - Java, JEE, Spring-web.   


# Code Snippets

the codes are proprietory.


# Deployment
Database server  
Application server  
Informatica server  
job scheduler   


# Conclusion  
The design ideas described in this document is applicable when the team skill sets and circumstances is ideal.

In practical, designs have to be adapted to a team skills to some extend. At the end, the team will be the one supporting and maintaining the data integration and data warehouse system on on-going basis. 

However considering the team members turnover, the architecture and design motto should always be **"DO It Right"**.








