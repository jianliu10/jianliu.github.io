---
layout: post
title:  "(Draft) Bigdata technical notes - 2019"
date:   2019-12-15 00:00:00 -0500
categories: tech-data-integration
---

# 2019 bigdata technical notes #

## Hadoop HDFS


## HIVE data warehouse

### partitions 

By default, a simple HQL query scans the whole table. This slows down the performance
when querying a big table. This issue could be resolved by creating partitions. 

In Hive, each partition corresponds to a predefined 
partition column(s), which maps to subdirectories in the table's directory in HDFS. When
the table gets queried, only the required partitions (directory) of data in the table are being
read, so the I/O and time of the query is greatly reduced. Using partition is a very easy and
effective way to improve performance in Hive.

The use case for static and dynamic partition is quite different. 

- Static partition
  Static partition is often used for an external table containing data newly landed
in HDFS. In this case, it often uses the date, such as yyyyMMdd, as the
partition column. Whenever the data of the new day arrives, we add the
day-specific static partition (by script) to the table, and then the newly
arrived data is queryable from the table immediately. 

- dynamic partition
  dynamic partition is often being used for data transformation between internal
tables with partition columns derived from data itself; 


### buckets

Besides partition, the bucket is another technique to cluster datasets into more manageable
parts to optimize query performance. Different from a partition, a bucket corresponds to
segments of files in HDFS. For example, the employee_partitioned table from the
previous section uses year and month as the top-level partition. If there is a further request
to use employee_id as the third level of partition, it creates many partition directories. 

For instance, we can bucket the employee_partitioned table using employee_id as a bucket
column. The value of this column will be hashed by a user-defined number of buckets. The
records with the same employee_id will always be stored in the same bucket (segment of
files). 

The bucket columns are defined by CLUSTERED BY keywords. It is quite different
from partition columns since partition columns refer to the directory, while bucket columns
have to be actual table data columns. 

		CLUSTERED BY (employee_id) INTO 2 BUCKETS

### HQL engine		
		
yarn jobs

		
## hadoop YARN resouece manager and node manager

yarn job

AM: Application Master



		
## mapreduce computing engine

		
## Spark computing engine

### csv / json data sources

you can read CSV / JSON files in single-line or multi-line mode. In single-line mode, a file can be split into many parts and read in parallel.

If a JSON object occupies multiple lines, you must enable multi-line mode for Spark to load the file. Files will be loaded as a whole entity and cannot be split.

### database data source

https://docs.databricks.com/spark/latest/data-sources/index.html
https://docs.databricks.com/spark/latest/data-sources/sql-databases.html

If running within the spark-shell use the --jars option and provide the location of your JDBC driver jar file on the command line.
spark-shell --jars ./mysql-connector-java-5.0.8-bin.jar

Once the spark-shell has started, we can now insert data from a Spark DataFrame into our database


## databrick - IAAS for spark development	

