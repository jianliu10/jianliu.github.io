---
layout: post
title:  "Java 8 concurrency programming notes"
date:   2018-10-02 13:15:42 -0500
categories: tech java-spring
---

### hardware thread, software thread   
https://www.codeguru.com/cpp/sample_chapter/article.php/c13533/Why-Too-Many-Threads-Hurts-Performance-and-What-to-do-About-It.htm

Number of hardware threads = (number of CPUS) * (number of cores per CPU) * (number of hardware threads per core : ususally 1 thread per core, 2 thread per core if CPU processor supports HyperThreading)

One hardware thread can run many software threads. In modern operating systems, this is often done by time-slicing. CPU L1 cache memory, main L2 cache memory contention if too many software threads -> slow down performance


## thread pool

How will you implement a thread pool yourself.
Executors class is a factory class that create different types of thread pools..
Executor interface,  ExecutorService implements Executor, A ExecutorService instance is a thread pool.

	return new ThreadPoolExecutor(corePoolSize, maxPoolSize, keepAliveTime, TimeUnit.MILLISECONDS,
								  new LinkedBlockingQueue<Runnable>(processorBoundQueueCapacity), threadFactory,
								  new ThreadPoolExecutor.AbortPolicy());
 
 
## Thread 

java VOLATILE variable is NOT thread safe. VOLATILE only gurantee variable read or write operation is atomic. It can be used to get/set state without increment values.
  
java java.util.concurrency.atomic.* classes are thread safe. It uses CPU atomic operations to complete "incrementAndGet" or "getAndIncrement"

hashmap not thread safe. hashtable vs concurrentHashMap both are thread safe.   

arraylist not thread safe. vector is thread safe.  

memory space: heap vs stack

methods in java.lang.Object.  

HashMap key class must be immutable and implements hashcode() and equals()  

thread context switch, process context switch,   
thread scheduler, cpu time slice (=preemptive scheduling),   
CPU registors state save/restore, cpu's L1 cache (on CPU chip), main L2 cache (on motherboard), main memory (on RAM chip)


## thread deadlock and starvation

deadlock: lock order.

starvation:
	- lock starvation
	- cpu startvation


## task-based programming 

executors.newWorkStealingThreadPool,    
task scheduling, work stealing, parallelism, forkjointask, forkjoinpool, 
 
## java.util.Optional

 
## Java 8 - @FunctionalInterface 

A java Interface class can contains default method, static method, abstract method.
  
A functional interface contains only one abstract method. It can contains other default and/or static interface methods.  

Functional object - Implementation of a functional interface. 

A functional object can be assigned to a variable/parameter with a function interface type.

There are two category of functional objects:
- method reference: 
	- constructor reference:   .collect(Collectors.toCollection(TreeSet::new))
	- Static method reference, 
	- instance method reference of an object of a particular type,    Arrays.stream(a).map(Object::toString)
	- instance method reference of an existing object  

- lambda expression:
  Lambda is a single expression annonymous function that is often used as inline function. Java internal mechanism is to create an instance of an annonymous concrete class that implements a function interface type. This ways, java effectively treat a Lambda expression as a functional object.

  Sample - use a lambda expression with Callable:
  
		Callable<String> callable = () -> {
			// Perform some computation
			Thread.sleep(2000);
			return "Return some result";
		};


## java 8 streaming
 
### java.util.function.*
 
### java.util.stream.Collectors class

Collectors class is a factory class. It creates Collector instances that implement various reduction operations, such as accumulating elements into collections, summarizing elements according to various criteria, grouping, partitioning, etc.

	public final class Collectors extends Object

### java.util.stream.Stream class
 
Stream static factory methods:  
	of, concat, empty, generate, iterate, 
	
Stream transformation methods (a pipeline step that returns a Stream instance)  
	filter, map, flatmap, distinct, sorted, limit, skip, peek(consumer)
	
Stream action methods (a pipeline step that returns a non-Stream instance):  
	findFirst, findLast, findAny
	allMatch, anyMatch, noneMatch
	forEach(consumer)
	// statistics
	specific methods: count, max, min  
	generic methods: reduce(BinaryOperator accumulator), reduce(initialValue, BinaryOperator accumulator), reduce(initialValue, accumulator, combiner)
	// to collection
	specific methods: toArray
	generic methods: collect(collector), collect(supplier, accumulator, combiner)


The following are examples of using the predefined collectors to perform common mutable reduction tasks:

	Integer sum = integers.reduce(0, (a, b) -> a+b);
	Integer sum = integers.reduce(0, Integer::sum);		// static method reference  java.lang.Integer.sum(int a, int b)
	
     // Accumulate names into a List
     List<String> list = people.stream().map(Person::getName).collect(Collectors.toList());

     // Accumulate names into a TreeSet
     Set<String> set = people.stream().map(Person::getName).collect(
												Collectors.toCollection(TreeSet::new));

     // Convert elements to strings and concatenate them, separated by commas
     String joined = things.stream()
                           .map(Object::toString)
                           .collect(Collectors.joining(", "));

     // Compute sum of salaries of employee
     int total = employees.stream()
                          .collect(Collectors.summingInt(Employee::getSalary)));

     // Group employees by department
     Map<Department, List<Employee>> byDept
         = employees.stream()
                    .collect(Collectors.groupingBy(Employee::getDepartment));

     // Compute sum of salaries by department
     Map<Department, Integer> totalByDept
         = employees.stream()
                    .collect(Collectors.groupingBy(Employee::getDepartment,
                                                   Collectors.summingInt(Employee::getSalary)));

     // Partition students into passing and failing
     Map<Boolean, List<Student>> passingFailing =
         students.stream()
                 .collect(Collectors.partitioningBy(s -> s.getGrade() >= PASS_THRESHOLD));
		

		
## java 8 - Asynchronous programming CompletableFuture  

https://www.callicoder.com/java-8-completablefuture-tutorial/

see sample code in: C:\UserData\cap??\cardinal-prime\cardinal-channel-accounts\cardinal-accounts-srvprovider\src\main\java\com\p??\cardinal\accounts\srvprovider\AccountsServiceProvider.java

CompletableFuture<T> implements Future<T>, CompletionStage<T>

### Running asynchronous computation using runAsync() and supplyAsync():

	CompletableFuture<Void> CompletableFuture.runAsync(() -> {})
	CompletableFuture<T> CompletableFuture.supplyAsync(() -> {return a instance of type T))

You can pass your own Executor argument to the runAsync() or supplyAsync() method call. If not specified, CompletableFuture executes these tasks in a thread obtained from the global ForkJoinPool.commonPool()
	
### Transforming and acting on a CompletableFuture

- attach a callback to the CompletableFuture using thenApply(), thenAccept() and thenRun():

		CompletableFuture<S> CompletableFuture.thenApply(T -> S)
		CompletableFuture<Void> CompletableFuture.thenAccept(T -> void)
		CompletableFuture<Void> CompletableFuture.thenRun(() -> {})
	
- async callback

		CompletableFuture<S> CompletableFuture.thenApplyAsync(T -> S)
	
- CompletableFuture.get() method is blocking. It waits until the Future is completed and returns the result after its completion.

		CompletableFuture.supplyAsync(() -> {
			// Code which might throw an exception
			return "Some result";
		}).thenApply(result -> {
			return "processed result";
		}).thenApply(result -> {
			return "result after further processing";
		}).thenAccept(result -> {
			// do something with the final result
		});

		
### Combining two CompletableFutures together  

- Combine two dependent futures using thenCompose() 
  thenCompose() is used to combine two Futures where one future is dependent on the other.
  
	CompletableFuture<User> getUsersDetail(String userId) {
		return CompletableFuture.supplyAsync(() -> {
			UserService.getUserDetails(userId);
		});	
	}

	CompletableFuture<Double> getCreditRating(User user) {
		return CompletableFuture.supplyAsync(() -> {
			CreditRatingService.getCreditRating(user);
		});
	}
	CompletableFuture<Double> result = getUserDetail(userId).thenCompose(user -> getCreditRating(user));
  
- Combine two independent futures using thenCombine()  
  thenCombine() is used when you want two Futures to run independently and do something after both are complete.
  The callback function passed to thenCombine() will be called when both the Futures are complete.
  
  CompletableFuture<Double> combinedFuture = weightInKgFuture
        .thenCombine(heightInCmFuture, (weightInKg, heightInCm) -> {
			Double heightInMeter = heightInCm/100;
			return weightInKg/(heightInMeter*heightInMeter);
			});

### Combining multiple CompletableFutures together  

sample 1 - CompletableFuture.allOf():
	List<String> webPageLinks = Arrays.asList(...)	// A list of 100 web page links
	// Download contents of all the web pages asynchronously
	List<CompletableFuture<String>> pageContentFutures = webPageLinks.stream()
			.map(webPageLink -> downloadWebPage(webPageLink))
			.collect(Collectors.toList());

	// Create a combined Future using allOf()
	CompletableFuture<Void> allFutures = CompletableFuture.allOf(
			pageContentFutures.toArray(new CompletableFuture[pageContentFutures.size()])
	);

	// When all the Futures are completed, call `future.join()` to get their results and collect the results in a list -
	CompletableFuture<List<String>> allPageContentsFuture = allFutures.thenApply(v -> {
	   return pageContentFutures.stream()
			   .map(pageContentFuture -> pageContentFuture.join())
			   .collect(Collectors.toList());
	});	

	// Count the number of web pages having the "CompletableFuture" keyword.
	CompletableFuture<Long> countFuture = allPageContentsFuture.thenApply(pageContents -> {
		return pageContents.stream()
				.filter(pageContent -> pageContent.contains("CompletableFuture"))
				.count();
	});

	
sample 2 - CompletableFuture.anyOf():

	CompletableFuture<String> future1, future2, future3
	
	CompletableFuture<Object> anyOfFuture = CompletableFuture.anyOf(future1, future2, future3);
	anyOfFuture.get();

	
### CompletableFuture Exception Handling  

- Handle exceptions using exceptionally() callback
	CompletableFuture<String> maturityFuture = CompletableFuture.supplyAsync(() -> {
	}).exceptionally(ex -> {
		System.out.println("Oops! We have an exception - " + ex.getMessage());
		return "Unknown!";
	});

-  Handle exceptions using the generic handle() method
	CompletableFuture<String> maturityFuture = CompletableFuture.supplyAsync(() -> {
	}).handle((res, ex) -> {
		if(ex != null) {
			System.out.println("Oops! We have an exception - " + ex.getMessage());
			return "Unknown!";
		}
		return res;
	});  

	
