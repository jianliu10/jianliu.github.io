---
layout: post
title:  "Java 8 monitoring tools"
date:   2018-01-02 13:15:42 -0500
categories: tech java-spring
---

https://programmer.help/blogs/jvm-performance-tuning-monitoring-tools-jps-jstack-jmap-jhat-jstat-hprof.html

## heap

Heap memory = younger generation + older generation + permanent generation  
Younger generation = Eden area + two Survivor areas (From area, To area)  

## stack
 

## monitoring tools

jps finds running Java processes by scanning through /tmp/hsperfdata_<username> directory. Each HotSpot-based Java process creates a file in this directory with the name equal to the process ID.

The file /tmp/hsperfdata_<username>/<pid> contains various counters exported by the JVM. These counters can be read by an external process. This is exactly how jstat works. 

So, jstat can always read counters of a local Java process, but in order to be able to monitor a remote machine, jstatd needs to be running.

jmap, jstack and jinfo use Dynamic Attach mechanism. These utilities connect to the target JVM via UNIX-domain socket and send the corresponding command to the JVM. The command is executed by the remote JVM itself.

 
## bin/jps(JVM Process Status)

jps is mainly used to list all JVM processes running in a host. The output info include main class, jvm params, application params.  
jps [options] [hostid]  

## bin/jstat, bin/jstatd (JVM Statistical Monitoring Tool)

	jstat [ generalOption | outputOptions vmid [interval[s|ms] [count]] ]
	
	// the following output is GC information, sampling time interval is 250ms, sampling number is 4:
	jstat -gc 21711 250 4


## bin/jinfo (before java 8), bin/jcmd (after java 8)

gets configuration information from a running Java process or crash dump and prints the system properties or the command-line flags that were used to start the JVM.

The release of JDK 8 introduced Java Mission Control, Java Flight Recorder, and jcmd utility for diagnosing problems with JVM and Java applications. It is suggested to use the latest utility, jcmd instead of the previous jinfo utility for enhanced diagnostics and reduced performance overhead.

	jinfo pid
	jcmd pid

   
## bin/jstack

jstack is mainly used to view thread stack information in a Java process.  

	jstack [option] pid  
	- l long listings, which prints out additional lock information, and **jstack-l pid** can be used to observe lock holdings when deadlocks occur.
   
example:

1. The first step is to find out the Java process ID. The name of the Java application I deployed on the server is mrf-center:
		ps -ef | grep mrf-center | grep -v grep
		root     21711     1  1 14:47 pts/3    00:02:10 java -jar mrf-center.jar
2. The second step is to find out the most CPU-consuming thread in the process
		top  
   The TIME column is the CPU time consumed by each Java thread. The longest CPU time is the thread with thread ID 21742.		
		printf "%x\n" 21742  
   The hexadecimal value of 21742 is 54ee, which will be used below.    
		jstack 21711 | grep 54ee  
	"PollIntervalRetrySchedulerThread" prio=10 tid=0x00007f950043e000 nid=0x54ee in Object.wait() [0x00007f94c6eda000]  
	

## bin/jmap (Heap Map) and bin/jhat (Java Heap Analysis Tool)

jmap is used to view heap memory usage, usually combined with jhat.

	jmap [option] pid
	
	// Printing process class loader and class loader loaded persistent generation object (class objects) information
	jmap -permstat pid
	
	// view the process heap memory usage, including the GC algorithm used, heap configuration parameters and heap memory usage in each generation	
	jmap -heap pid    
	
	// view the number and size statistics histogram of objects in heap memory. If you bring live, only live objects are counted, 
   jmap -histo:live 21711 | more
   
   // dump the process memory usage into the file with jmap, and then use jhat analysis to see. 
   jmap -dump:format=b,file=dumpFileName pid
   jmap -dump:format=b,file=/tmp/dump.dat 21711  
   
   // dump out of the file can be seen with MAT, Visual VM and other tools, here with jhat view
   jhat -port 9998 /tmp/dump.dat
   //if the Dump file is too large, you may need to add the parameter -J -Xmx512m to specify the maximum heap memory, 
   jhat -J-Xmx512m -port 9998 /tmp/dump.dat
   // enter the host address in the browser: 9998 to see
   
   
## hprof(Heap/CPU Profiling Tool)

	java -agentlib:hprof[=options] ToBeProfiledClass		// run time JVM params
	java -Xrunprof[:options] ToBeProfiledClass				// run time JVM params
	javac -J-agentlib:hprof[=options] ToBeProfiledClass		// compile time javac params. Its implementation uses bytecode injection technology
	
	// CPU Usage Sampling Profiling(cpu=samples). The generated profile file name is java.hprof.txt, in the current directory. 
	java -agentlib:hprof=cpu=samples,interval=20,depth=3 Hello
	// CPU Usage Times Profiling(cpu=times) can obtain finer-grained CPU consumption information than CPU Usage Sampling Profile
	javac -J-agentlib:hprof=cpu=times Hello.java
	// Heap Allocation Profiling(heap=sites)
	javac -J-agentlib:hprof=heap=sites Hello.java
	// Heap Dump(heap=dump) can generate more detailed Heap Dump information than the previous Haip Allocation Profiling
	javac -J-agentlib:hprof=heap=dump Hello.java
	
Although adding - Xrunprof:heap=sites parameter to JVM startup parameter can generate CPU/Heap Profile file, it has a great impact on JVM performance and is not recommended for online server environment.	
	
## bin/jconsole 

Jconsole is a JMX-compliant monitoring tool.  It uses the extensive JMX instrumentation of the Java virtual machine to provide information on performance and resource consumption of applications running on the Java platform.

### The jconsole interface (old util, new util is bin/jvisuamvm)

The jconsole interface is composed of six tabs:
- Summary tab: displays summary information on the JVM and monitored values.
- Memory tab: displays information on memory use.
- Threads tab: displays information on thread use.
- Classes tab: displays information on class loading
- MBeans tab: displays information on MBeans
- VM tab: displays information on the JVM


## bin/jvisualvm  (since java 6)

provides a visual interface for viewing detailed information about Java applications while they are running on a Java Virtual Machine (JVM), and for troubleshooting and profiling these applications. 

most of the previously standalone tools JConsole, jstat, jinfo, jstack, and jmap are part of Java VisualVM. Java VisualVM federates these tools to obtain data from the JVM software, then re-organizes and presents the information graphically, to enable you to view different data about multiple Java applications uniformly, whether they are running locally or on remote machines. 

Java VisualVM can be used by Java application developers to troubleshoot applications and to monitor and improve the applications' performance. Java VisualVM can allow developers to generate and analyse heap dumps, track down memory leaks, browse the platform's MBeans and perform operations on those MBeans, perform and monitor garbage collection, and perform lightweight memory and CPU profiling.

		