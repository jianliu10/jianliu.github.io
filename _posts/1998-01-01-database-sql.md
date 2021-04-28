---
layout: post
title:  "SQL databases - 2018"
date:   2018-10-01 13:15:42 -0500
categories: tech-database
---

# SQL databases

## tables

- heap organized table (heap table)  
  This is a standard Oracle table; 
  A heap-organized table is a table with rows stored in no particular order. the term "heap" is used to differentiate it from an index-organized table or external table. If a row is moved within a heap-organized table, the row's ROWID will also change.  
  
- clustered index organized table    
  A clustered key is the primary key, actual rows are stored in tree nodes.   

- external table  
  An external table is a table whose data is NOT stored within the Oracle database. Data is loaded from a file via an access driver (normally ORACLE_LOADER) when the table is accessed. One can think of an external table as a view that allows running SQL queries against files on a filesystem without the need to first loaded the data into the database.
  
			CREATE TABLE t1 
			( c1 NUMBER,  
			  c2 VARCHAR2(30)
			)
			ORGANIZATION EXTERNAL
			( default directory my_data_dir
			  access parameters
			  ( records delimited by newline
				fields terminated by ','
			  )
			  location ('report.csv')  
			);


## index

Type 1 :
- clustered index (index organized table)
- non-clustered index (the tree nodes store the physical locations of the actual rows in disk)  

Type 2:
- b-tree index (index on a column with high cardinality), used in frequent DML(upsert/delete) operations.  
- bitmap index (index on a column with low cadinality for millions of rows ). Since it is expensive to update bitmap index, bitmap index is used in write-once read-many application, such as overnight batch load of fact table in DW.

### cadinality

A lot of distinct values is called high cardinality; a few of distinct values is called low cardinality.


## performance tunning  

execute the sql -> run "explain plan" -> analyze the dumped execution plan.

Speed (hash join is faster since it access memory, while b-tree index join access disk)
	HASH JOIN > SORT-MERGE join > NESTED LOOPS join with b-tree index on second table. 
	
memory consumption in temp tablespece (hash join use more memory):
	HASH JOIN == SORT-MERGE join > NESTED LOOPS join with b-tree index on second table. 

analyze execute time, cost-based optimization (always in Oracle)


## Oracle partitions

partition and sub-partition

- date partitions (1 day, 1 month, 1 quarter)
- range partitions
- list partitions
- reference partitions (child table uses the same partition config as its parent table if child table has a foreigh key reference to parent table partitioning column). since oracle 11g


## Oracle HASH Joins, SORT-MERGE joins

HASH joins are the usual choice of the Oracle optimizer when the memory is set up to accommodate them. In a HASH join, Oracle accesses one table (usually the smaller of the joined results) and builds a hash table on the join key in memory. It then scans the other table in the join (usually the larger one) and probes the hash table for matches to it. Oracle uses a HASH join efficiently only if the parameter PGA_AGGREGATE_TARGET is set to a large enough value. If MEMORY_TARGET is used, the PGA_AGGREGATE_TARGET is included in the MEMORY_TARGET, but you may still want to set a minimum.

If you set the SGA_TARGET, you must set the PGA_AGGREGATE_TARGET as the SGA_TARGET does not include the PGA (unless you use MEMORY_TARGET as just described). The HASH join is similar to a NESTED LOOPS join in the sense that there is a nested loop that occurs. Oracle first builds a hash table to facilitate the operation and then loops through the hash table. When using an ORDERED hint, the first table in the FROM clause is the table used to build the hash table.

HASH joins can be effective when the lack of a useful index renders NESTED LOOPS joins inefficient. The HASH join might be faster than a SORT-MERGE join, in this case, because only one row source needs to be sorted, and it could possibly be faster than a NESTED LOOPS join because probing a hash table in memory can be faster than traversing a b-tree index.

As with SORT-MERGE joins and CLUSTER joins, HASH joins work only on equal joins. As with SORT-MERGE joins, HASH joins use memory resources and can drive up I/O in the temporary tablespace if the sort memory is not sufficient (which can cause this join method to be extremely slow).

Finally, HASH joins are available only when cost-based optimization is used (which should be 100 percent of the time for your application running on Oracle 11g).

Table 1 illustrates the method of executing the query shown in the listing that follows when a HASH join is used. Normally hash table is created on smaller table 'dept'. but here /*+ ordered */ hint instructs Oracle to create a hash table on first table 'emp'.

	select /*+ ordered */ ename, dept.deptno
	from emp, dept
	where emp.deptno = dept.deptno

	
## NESTED LOOPS joins

optimization: create index on joined columns. the second table are a smaller table with high cardinality on joined column.


## Oracle

### Oracle 11G vs 12C

- Oracle 11g was released in 2008. G stands for grid.  
	- 11g has no cloud service, It Has no pluggable databases, there is no multitenant architecture, No in-memory capabilities, Has no JSON type support, Comparatively lower performance in I/O throughput and response time.

- Oracle 12c was released in 2014, C stands for cloud.    
	- 12c is designed for the cloud. It uses container tech. It provides Oracle database service on cloud. 
	- It provides pluggable databases to support rapid provisioning and portability. It allows running multiple databases on the same hardware while maintaining the security and isolation among the databases.
	- there is multitenant architecture. It enables an Oracle database to function as a multitenant container database (CDB)
	- **Has in-memory capabilities that provide real-time analytics**
	- added JSON data type support, 
	- Comparatively higher performance in I/O throughput and response time.


### Oracle exadata

The Oracle Exadata is engineered to deliver **dramatically better performance**, cost effectiveness, and high availability for Oracle databases. 

Oracle exadata is  
- cloud-based architecture, running on private cloud or public cloud.
- scale-out high performance database servers
- scale-out storage servers with state-of-art PCI flash
- ultra-fast InfiniBan networking that connect all servers and storage
- Unique software algorithms implement database intelligence in compute, storage, and networking to deliver higher performance and capacity at lower costs  

Exadata runs all types of database workloads including 
- Online Transaction Processing (OLTP), 
- Data Warehousing (DW), 
- In-Memory Analytics 

Exadata can be purchased and deployed 
- on premises as the foundation for a private database cloud
- or using a subscription model and deployed in the Oracle 'Public Cloud' or 'Cloud at Customer' with all infrastructure management performed by Oracle.


## data warehouse 

### dimension types & dimension table types

- Types of Dimensions :
	- Conformed Dimension - separate dimension tables. creating consistency. The same dim table can be referenced by the multiple fact tables.
	- Degenerated Dimension – the dimension attributes are stored as part of the fact table and not in a separate dimension table. Usually used when a dimension has only one attribute.
	- Junk Dimension – A junk dimension is a single dimensional table with a combination of different and unrelated attributes. This is to avoid having a large number of foreign keys in the fact table. 
	- Role play dimension – It is a dimension table that has multiple valid relationships with a fact table. For example, a fact table may include foreign keys for both ship date and delivery date. But the same dimension attributes apply to each foreign key so the same dimension tables can be joined to the foreign keys.

- Slowly Changing Dimensions table types:
	- Type 1  is to over write the old value. (no history, overwrite the row) 
	- Type 2 is to add a new row. (keep history. Add columns "active_flag, effective_start_date, effective_end_date", multiple rows) 
	- Type 3 is to create a new column. (new value, old value in one rows)


### fact attribute types & fact table types

- Types of Fact attributes:  
	- Additive: Additive fact attributes can be summed up through ALL of the dimensions in the fact table.
	- Semi-Additive: Semi-additive facts attributes can be summed up for some of the dimensions in the fact table, but not the others.
	- Non-Additive: Non-additive facts attributes cannot be summed up for any of the dimensions present in the fact table.

- Types of Fact Tables:  
	- Cumulative: This type of fact table describes what has happened over a period of time. For example, this fact table may describe the total sales by product by store by day. The facts for this type of fact tables are mostly additive facts. The first example presented here is a cumulative fact table.
	- Snapshot: This type of fact table describes the state of things in a particular instance of time, and usually includes more semi-additive and non-additive facts. The second example presented here is a snapshot fact table.


