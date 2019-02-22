---
layout: post
title:  "Case Study - Retail Promotion Data Integration Automation and Enhancement in DWH"
date:   2019-2-22 00:00:00 -0500
categories: tech data-integration
---

## Background
Promotion is a key part of Merchandizing Sales and Marketing system. For a large retail company with 700+ retail stores, the look-forward promotion data need to be sent to the stores for properly goods stock preparation.

This time, the sales and marketing business wants to add 40 new promotion programs, change 10 existing promotion programs, and add 50 new promotion sub-programs.

I was tasked to enhance an 10+ years aged existing Promotion data integration component in a data warehouse. The goal of the enhancement is not only to add new and changed promotion programs and sub-programs for this one-time project, but also to automate the promotion data integration process for the future.

This document describes the as-of state of existing promotion data integration component in DWS, and its enhancement design. 


## Abbreviations  

DWS 		- Data Warehouse System  
SAM 		- the Merchandizing Sales and Marketing transactional system  
promotion DI 	- the promotion data integration component in DWS  
PromoTree 	- the retail store promotion management system  
ORM 		- Online Order Management system  


## the as-of state of existing promotion data integration component

PROMO component in DWS provides data for a few key business functions:
- provide fact-dimensional data for Reports and Business Intelligence Analysis.
- provide flat store&item level promotional data for downstream retail store promotion management system. 
- provide flat corporate&item level promotional data for downstream Online Order Management system.


There are a few issues in existing promotion data integration component 

This enhancement design addresses several issues in the existing Promotion data integration component in DWS. The Technical Debts (issues) are described in the following.

### TechnicalDebt-1: manual process to add or change dimensional data in DWS 

   The existing system adopts an approach that is commonly used by developers in data warehouse system. Since the DWS developers are good at writing SQLs, they wrote the individual SQL insert/update/delete DML statments to add/change dimensional data in DWS dimensional data. This approach is simple and works great when the number of new or changed dimensional records is small (less than 20 records). 

   However, when the number of new or changed dimensional records is big (hundreds of records), this approach is heavy coding labored, non-visualized, hard to trace the DML statments in script.

   
   ![Load dimensional data using manual process](/images/PromotionDI-EnhancementDesign/loadDimData-manualProcess.jpg)

   
### TechnicalDebt-2: Use visual ETL tool to process complex data transformation logic

   The DWS system uses Informatica jobs to extract, transform, and load Fact data. Visual ETL tools like Informatica, Talend work great when the data transformation logic is simple and straightforwd. 

   However, the visual ETL tools become unwieldly when the data transformation logic is complex.

   The visual ETL tools become unwieldly in complex data transformation logic. For a complex transformation logic, which could have been solved with several lines of Python or Java codes in clear logic flow, it will take many work-around steps in ETL tool to develped. The end result of using visual ETL tool to develop complex transformation logic is a visual flow chart that is highly complex, confusing, difficult to be undertood by other developers and thus difficult to maintain and enhance in the future. With the time going by, even the oroginal developers will have difficulty understanding the highly complex visual flow chart themselves.  
  
### TechnicalDebt-3: Use visual ETL tool to process extra large of data traffic volumn without sub-partitions 

   With today's network speed, visual ETL tool like Informatica and Talend can handle a few millions of fact records with tolerable time performance. 
   
   However, when there are a few hundreds of millions thus tens of billions of bytes of fact data, it will take hours to read the fact data from database, transform, and load them back to database. The time performance is untolerable. The issues lie in a few areas: 1) reading and writing extra large number of fact records from/to database causes IO bandwidth bottleneck; 2) the large memory cache size and concurrent multi-threaded procesing on ETL server demands a machine with high CPU capacity and high RAM capacity. This will be costly in hardware investment. 3) the fact data is not sub-partitioned for parallel processing in a grid cluster.
  
   For example, in the current promotion data integration system, it generates promotion data not only by monthly promotion turn, also by weekly, and by daily. The reason of generating weekly and daily promotion data is to provide a sales revenue weekly and daily drilldown view for report and BI analysis. In each monthly promotion turn, there are avg 170k promotion items records, avg 467k promotion locations records. By multiplying by 4, there are 680k and 1,840k weekly records per month. By multiplying by 30, there are 5,100k and 14,010k daily records per month. Assume each record avg 500 bytes, the size of promotion data generation can reach up to tens of billions of bytes in a fresh daily batch jobs run.

  
### TechnicalDebt-4: hard coded codes/types mapping logic from upstream transactional systems to DWS report system
   
   It was initially convenient to hard code codes/types mapping logic in ETL transformation when the number of mappings is small. With the upstream transactional system adds more and more codes/types, the hard coded mapping logic becomes a constant coding labor. Every time there is a new or changed promotion program or promotion location in upstream transactional system, the hard coded mapping logic need to be revisited and changed accordingly. Hard coded mapping logic also makes the visual ETL flow chart bloated with several branches of data process flows that only differs in codes/types mapping logic. 
   
   For example, promotion data integation component maps promotion location codes used in updtream SAM system to the promotion subtypes used in DWS report system. There are different codes/types mapping logics depending on the promotion types. 
    

## Design of promotion data integration automation and enhancement 

### Automate new or changed dimensional data integration into DWS 

This session proposes two solutions to TechnicalDebt-1 described in the above. 

#### Basic enhancement - Using business client CSV input files and PL/SQL Stored Procedures

When adding or chaning a large number of dimensional data records, The current approach is to write individual insert/update/delete SQL DML statements for each records to directly load into multiple inter-related final dimensional tables in DWS with surrogate IDs (integer) as foreign key references. Assume there are 50 of new or changed programs, and 5 final fact tables to populated, there will be, <number of new or changed programs> * <number of final dimensional tables>, total 250 SQL DML statments to hand written by developers.

The enhanced approach is to use business client provided CSV input files. The business client provides CSV input files containing new or changed business records, a **generic python script** is developed to read the business CSV input files, load the raw file contents to staging tables in DWS. PL/SQL stored procedures are developed to read dimensional records from staging tables, transform them, and load them into the multiple inter-related final dimensional tables in DWS.

Assume there are 50 of new or changed programs, the enhanced approach will requires business client to provide a csv file with 50 lines of records.

We can see that using this business CSV input file approach, it still invoves developers' manual work to receive a new csv file from business client, verify the input file format, run the data staging job and PL/SQL jobs.

![Load dimensional data using CSV input file + PL/SQL Stored Procedures](/images/PromotionDI-EnhancementDesign/loadDimData-csvInputFile.jpg) 


#### Advanced enhancement - Using business client notifications, actions and approval workflow

This advanced enhancement implements a fully automatic workflow that involves business client only. The approach will not involve the developers in the on-going dimensional data population, maintenance and governance. 

I have rarely seen this advanced enhancement in data warehouse systems. The rarity is mainly because the development of automatic workflow requires technical skills that go beyond a typical DWS DI team skill sets. The workflow is actually an web application development. 

The high level design of the workflow is as the following:

1. developed a standalone web application that running forever. It is not a scheduled job.

2. the scheduled ETL batch jobs will extract the dimensional information from upstream data feeds, and populate the DWS dimensional tables with as-it-is data quality or even blank fields values which are necessary in DWS report system. If the upstream data feed introduces a new programs, the ETL jobs will add a new records to  dimensional table with many key field unpopulated.

3. the web app polls the dimensional tables in DWS a few times a day. If it detects any dimensional records with invalid field values or missing required field values, it will put the invalid dimensional records into a notification queue.

4. the web app sends email notification to the business client about the new or invalid dimensional records.

5. business client go to a browser to log in to web app
	- the business client fills or corrects the dimensional records. submit and approve the changes. The changed dimensional records will be saved to DWS staging tables with FLAG column value set as "changed".
	- the business client can also query an existing valid dimensional records, change some fields to new values, submit and approve the change. The changed dimensional records will be saved to DWS staging tables with FLAG column value set as "changed".
	- the business client cal also add a new dimensional records. submit and approve the new records. The changed dimensional records will be saved to DWS staging tables with FLAG column value set as "new". 

6. upon completion of daily new or changed dimensional records, the business client clicks a button to trigger the execution of the PL/SQL stored procedures, which will read the new or changed dimensional data from staging tables, transform them, and load them into the multiple inter-related final dimensional tables in DWS.


![Automatic dimensional data governance using notification and approval workflow](/images/PromotionDI-EnhancementDesign/loadDimData-notificationApprovalWorkflow.jpg)


### Use python to process complex data transformation logic

Python has built-in language features for data processing. Python ecosystem includes data processing packages like Pandas and cx_Oracle to handle data extraction, transformation and loading. Using python, developers can write very complex transformation logic in a small chunk of codes. The benefits is an easy to understand and easy to maintain and support code base.

when the data transformation logic is complex, where visual ETL tool becomes unwieldy, I suggest writing a python script to process the transform logic.  Visual ETL tools like Informatica or Talend provides a UI command line component to invoke the python scripts.


### process extra large of data traffic volumn (hundreds of millions of fact records)  

I suggest using PL/SQL stored procedures to read/process/write extra larget of data. This avoids the IO network bottleneck, which is caused by the reading/writing extra large volumn of data to/from database to ETL client side jobs. PL/SQL will keep reading and writing data local to the database server.

Avoid using cursors in PL/SQL SP. Stored Procedure codes writing with cursors can not be optimized by database engine execution plan. Instead, writing big SQLs and using temporary tables to process the large data in PL/SQL SPs.


### Use mapping configuration tables instead of hard coded codes/types mapping logic

Data integration process always need to map codes/types from upstream transactional systems to DWS report system. The data quality of dimensional data is very import for report and BI analysis. 

Hard codes mappings of codes/types is a no. No matter how small number of the codes/types is. Always use a configuration table or file to configure the codes/types mappings.

For example, in promotion DI system, there is mapping logic to map from SAM system promotion location codes to DWS system promotion sub-type codes. The sample hard coded mapping snippets are as the following. each similar mapping logic appears in two places, one place is in corporation items transformation phase , the other place is in corporation locations transformation phase.

This is a perfect scenario to move the hard coded mapping logic to a mapping configuration table. 

We can see some mappings of codes/types are one-to-one straight forward. other mappings are based on certain field value patterns in a dimension record. To handle field value pattern based mappings, my suggestion is to use regular expressions in the mapping configuration table. In the future, we only need to add or change the mapping configuration table records without touching the ETL codes. This will greatly simplify the ETL code logic and improve the data process flow readability, maintainability, and time to production.


> TechnicalDebt-4: Sample hard coded mapping snippets in existing ETL Informatica job:
> 
> For promotionType = 'EndAisle'
> 
> - during promotion items data integration phase, 
> 
>   1. first get end aisle position
> 
> 			IIF( (INSTR(END_AISLE_NUMBER,'P',1) <>0), (SUBSTR(END_AISLE_NUMBER,0,(INSTR(END_AISLE_NUMBER,'P',1,1)-1))),   IIF( (INSTR(END_AISLE_NUMBER,'A',1) <>0), (SUBSTR(END_AISLE_NUMBER,0,(INSTR(END_AISLE_NUMBER,'A',1,1)-1))), 
> 			IIF( (INSTR(END_AISLE_NUMBER,'B',1) <>0), (SUBSTR(END_AISLE_NUMBER,0,(INSTR(END_AISLE_NUMBER,'B',1,1)-1))), IIF(UPPER(LTRIM(RTRIM(END_AISLE_NUMBER)))='HD', 'HD',END_AISLE_NUMBER))))
> 
>   2. get end aisle flight	
> 
> 			IIF( (INSTR(END_AISLE_NUMBER,'P',1) <>0 AND 
> 			INSTR(END_AISLE_NUMBER,'A',1) <>0 AND INSTR(END_AISLE_NUMBER,'B',1)<>0), 'PA+B', IIF( (INSTR(END_AISLE_NUMBER,'P',1) <>0), (SUBSTR(END_AISLE_NUMBER,(INSTR(END_AISLE_NUMBER,'P')))),
> 			IIF( (INSTR(END_AISLE_NUMBER,'A',1) <>0 AND INSTR(END_AISLE_NUMBER,'B',1)  <0 ), 'A+B',
> 			IIF( (INSTR(END_AISLE_NUMBER,'A',1) <>0), 'A',
> 			IIF( (INSTR(END_AISLE_NUMBER,'B',1) <>0), 'B',
> 			NULL)))))
> 
>   3. concate end aisle position and end aisle flight
> 
>   4. based on step 3 result, mapping the locations to subtypes:
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
> For promotionType = 'Mini Thematic', then
> 
> 		'MI'||'-'||substr(VALUE38, -1, 1)	
> 		
> 	

# Thought about migrating data integration processes to public cloud

Currently, all the data integration processes are implemented using Informatica visual ETL tool. This post a challenge to migrate the data integration processes to public cloud. 

In order to migrate the data integration processes to public cloud, I think the ETL jobs need to be written using Python or Java. This will leverage the massive scale of lower grade virtual machines in public cloud to accomplish parallel data processing. 

	
## Technology Stack 

PL/SQL Stored Procedures   
Python    
Informatica  
(Optional) business client notification and approval workflow - Java, JEE, Spring-web.   


# Code Snippets

the codes are proprietory.


# Deployment
Database server  
Informatica server  
Application server  
job scheduler   


# Conclusion  
The design ideas described in this document is applicable when the team skill sets and circumstances is ideal.

In practical, designs always have to be adapted to a team skills to some extend. At the end, the team will be the one supporting and maintaining the data integration and data warehouse system on on-going basis. 

However considering the team members turnover ratio, the architecture and design motto should still be "DO It Right".









