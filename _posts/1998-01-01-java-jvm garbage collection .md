---
layout: post
title:  "JVM garbage collection"
date:   2018-01-02 13:15:42 -0500
categories: tech-java-spring
---

## heap

Heap memory = younger generation + older generation + permanent generation  
Younger generation = Eden area + two Survivor areas (From area, To area) 

- scavenge GC (on Young generation only)
- Full GC 

## GC implementation types

JVM has four types of GC implementations:

- Serial Garbage Collector
- Parallel Garbage Collector
- CMS Garbage Collector
- G1 Garbage Collector

### serial gc

	java -XX:+UseSerialGC -jar Application.java
		
This is the simplest GC implementation, as it basically works with a single gc thread.

this GC implementation freezes all application threads when it runs. Hence, it is not a good idea to use it in multi-threaded applications like server environments.

The Serial GC is the garbage collector of choice for most applications that do not have small pause time requirements and run on client-style machines. 

		
### parallel gc

	java -XX:+UseParallelGC -XX:ParallelGCThreads=<N> -XX:MaxGCPauseMillis=<N> -XX:GCTimeRatio=<N> -jar Application.java
	
	-XX:MaxGCPauseMillis=<N>	// gap [in milliseconds] between two GC
	-XX:GCTimeRatio=<N>	//measured regarding the time spent doing garbage collection versus the time spent outside of garbage collection
	
It's the default GC of the JVM and sometimes called Throughput Collectors. this uses multiple gc threads for managing heap space.   
it also freezes other application threads while performing GC.		

### CMS gc

	java -XX:+UseParNewGC -jar Application.java

The Concurrent Mark Sweep (CMS) implementation uses multiple garbage collector threads for garbage collection. It's designed for applications that prefer shorter garbage collection pauses, and that can afford to share processor resources with the garbage collector while the application is running.

applications still respond while performing garbage collection but reponse slower on average.

note here is that since this GC is concurrent, an invocation of explicit garbage collection such as using System.gc() while the concurrent process is working, will result in Concurrent Mode Failure / Interruption.

If more than 98% of the total time is spent in CMS garbage collection and less than 2% of the heap is recovered, then an OutOfMemoryError is thrown by the CMS collector.

	
### G1 gc

	java -XX:+UseG1GC -jar Application.java
	
G1 (Garbage First) Garbage Collector is designed for applications running on:

- multi-processor machines 
- large heap size

G1 collector will replace the CMS collector since it's more performance efficient. applications still respond while performing garbage collection but reponse slower on average, but respond faster than using CMS gc.

Unlike other collectors, G1 collector partitions the heap into a set of equal-sized heap regions, each a contiguous range of virtual memory. When performing garbage collections, G1 shows a concurrent global marking phase (i.e. phase 1 known as Marking) to determine the liveness of objects throughout the heap.

After the mark phase is completed, G1 knows which regions are mostly empty. It collects in these areas first, which usually yields a significant amount of free space (i.e. phase 2 known as Sweeping). It is why this method of garbage collection is called Garbage-First.
	
### deduplication of strings
	
	-XX:+UseStringDeduplication

Java 8u20 has introduced one more JVM parameter for reducing the unnecessary use of memory by creating too many instances of same String. This optimizes the heap memory by removing duplicate String values to a global single char[] array.


## GC profile and tunning
	
### GC log file

**To diagnose any memory problems, the Garbage Collection log file is the best place to start**. It provides several interesting statistics:

- When the scavenge (or Young generation) GC ran?
- When the full GC ran?
- How many scavenge GCs and Full GCs ran? Did they run repeatedly? In what interval?
- After the GC process ran, how much memory was reclaimed in Young, Old, and Permanent/Metaspace generations?
- How long did the GC run?
- How long did JVM pause when Full GC run?
- What was the total allocated memory in each generation?
- How many objects were promoted to old generation?


### How to Generate GC Log File

	-XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:<file-path>
	-XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:/opt/app/gc.log
	
### Anatomy of GC Log Statement

example:

2014-11-18T16:39:37.728-0800: 88.808: [Full GC [PSYoungGen: 116544K->12164K(233024K)] [PSOldGen: 684832K->699071K(699072K)] 801376K->711236K(932096K) [PSPermGen: 2379K->2379K(21248K)], 3.4230220 secs] [Times: user=3.40 sys=0.02, real=3.42 secs]	

[Times: user=3.40 sys=0.02, real=3.42 secs] – Real is wall clock time (time from start to finish of the call).  
User is the amount of CPU time spent in user-mode code (outside the kernel) within the process.   
Sys is the amount of CPU time spent in the kernel within the process.   
if the CPU time (3.4 sec) is considerably higher than the real time passed (3.42 Sec), we can conclude that the GC was run using multiple threads.   

[PSOldGen: 684832K->699071K(699072K)] – After the GC ran the old generation space increased from 684832k (i.e. 669mb) to 699071k (i.e. 682mb) and total allocated old generation space is 669072k (i.e. 682mb). In this case after the GC event, old generation's space increased and didn't decrease, which isn't the case always. Here size has increased because all objects in Old generation are actively referenced + objects from young generation are promoted to old generation. Thus you are seeing the increase in the old generation size.

### Tools to Analyze GC Logs

using **http://gceasy.io/**, which is a universal online Garbage collection analysis tool.   
It's a free tool that analyzes the Garbage collection logs and provides telemetrics, potential Garbage Collection problems, and Memory problems, and it provides solutions to these problems, as well.

