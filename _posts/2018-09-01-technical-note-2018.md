---
layout: post
title:  "Technical notes - 2018"
date:   2018-10-01 13:15:42 -0500
categories: tech java-spring
---

# 2018 technical notes #

## Spring framework two essential features:
- IOC : Inversion of Control,  
  @Component, auto create beans in container based on declaration
- DI : Dependency Injection,  
  auto search for beans and assign beans to variables.
  @Autowired, @Qualifier, field injection, constructor/method parameter injection, 
  @Value - inject values from property files
- AOP: Aspect Oriented Programming

Spring bean scopes: singleton, prototype, request, session, spring-batch's step

## spring-boot ##
 (spring-boot-starter-web -> spring-mvcweb -> config filter chain -> last servlet is DispatcherServerlet -> route to a controller;   
 embedded app server, tomecat or jetty;  app server conf/server.xml defines two connectors: http connector (8080), https connector (443), different ports;  

> @SpringBootApplication(scanBasePackages={,,}   
> @SpringBootApplication is a convenience annotation that adds all of the following:   
> @Configuration, @EnableAutoConfiguration, @EnableWebMvc, @ComponentScan  
> @EnableAutoConfiguration tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.  
> @EnableWebMvc annotation is added automatically when it sees spring-webmvc on the classpath. This flags the app as a web app and activates key behaviors such as DispatcherServlet.  

see java classes : WebMvcConfigurer, WebMvcConfigurerAdapter, WebMvcConfigurationSupport, WebMvcAutoConfiguration, WebMvcAutoConfigurationAdapter  

@PropertySources = {,,}, 

Spring has two types of beans:
- configuration beans 
- regular beans


## Spring REST-API WS server side 
   
### Exception handling 

annotations: @ControllerAdvice, @ExceptionHandler   
classes: ResponseEntity<Object>, ResponseEntityExceptionHandler  

	@ControllerAdvice
	public class MyResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {
		@ExceptionHandler(value = { IllegalArgumentException.class, IllegalStateException.class })
		protected ResponseEntity<Object> handleConflict(
		  RuntimeException ex, WebRequest request) {
			String bodyOfResponse = "This should be application specific";
			return handleExceptionInternal(ex, bodyOfResponse, new HttpHeaders(), HttpStatus.CONFLICT, request);
		}
	} 
 

 ### maps from URI to controller object and methods

@EnableWebMvc will auto created two beans RequestMappingHandlerMapping and RequestMappingHandlerAdapter
  
DispatcherServlet uses two beans to map URI:
-RequestMappingHandlerMapping bean maps from URI to handler object at type level   
-RequestMappingHandlerAdapter bean maps from URI to method 


### marshall/unmarshall data from POJO to/from http request/response body

HttpMessageConverters is used to marshall/unmarshall data in http request/response body 

#### default HttpMessageConverters
@EnableWebMvc will pre-enable the following HttpMessageConverters instances by default:  

		ByteArrayHttpMessageConverter – converts byte arrays
		StringHttpMessageConverter – converts Strings
		ResourceHttpMessageConverter – converts org.springframework.core.io.Resource for any type of octet stream
		SourceHttpMessageConverter – converts javax.xml.transform.Source
		FormHttpMessageConverter – converts form data to/from a MultiValueMap<String, String>.
		Jaxb2RootElementHttpMessageConverter – converts Java objects to/from XML (added only if JAXB2 is present on the classpath)
		MappingJackson2HttpMessageConverter – converts JSON (added only if Jackson 2 is present on the classpath)
		MappingJacksonHttpMessageConverter – converts JSON (added only if Jackson is present on the classpath)
		AtomFeedHttpMessageConverter – converts Atom feeds (added only if Rome is present on the classpath)
		RssChannelHttpMessageConverter – converts RSS feeds (added only if Rome is present on the classpath)
 
### Customize HttpMessageConverters

class based configuration sample:

		@Configuration
		@EnableWebMvc
		@ComponentScan({ "org.baeldung.web" })
		public class WebConfig implements WebMvcConfigurer {
			 @Override
			 public void configureMessageConverters(List<HttpMessageConverter<?>converters) { 
				  // MappingJackson2HttpMessageConverter - we can now set a custom ObjectMapper on this converter and have it configured as we need to.
				  // MarshallingHttpMessageConverter – and we’re using the Spring XStream support to configure it. This allows a great deal of flexibility since we’re working with the low level APIs of the underlying marshalling framework – in this case XStream
			}
		}


xml based configuration sample:
	  
		<beans:bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
			<beans:property name="messageConverters">
				<beans:list>
					<beans:ref bean="jsonMessageConverter"/>
					<beans:ref bean="xmlMessageConverter"/>
					<beans:ref bean="stringMessageConverter"/> 
				</beans:list>
			</beans:property>
		</beans:bean>
		<!-- Configure bean to convert JSON to POJO and vice versa -->
		<beans:bean id="jsonMessageConverter" class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
		</beans:bean>	
		<!-- Configure bean to convert XML to POJO and vice versa -->
		<beans:bean id="xmlMessageConverter" class="org.springframework.http.converter.xml.Jaxb2RootElementHttpMessageConverter">
		</beans:bean>
		<bean class="org.springframework.http.converter.xml.MarshallingHttpMessageConverter">
				   <property name="marshaller" ref="xstreamMarshaller" />
				   <property name="unmarshaller" ref="xstreamMarshaller" />
		</bean> 	
		<bean id="xstreamMarshaller" class="org.springframework.oxm.xstream.XStreamMarshaller" />
	
	
## Spring REST-API WS client side, using RestTemplate	

	@Autowired RestTemplate restTemplate
	
	msgConverters.add(new StringHttpMessageConverter())
	restTemplate.setMessageConverters(msgConverters);
	
	String auth = username + ":" + password;
	byte[] encodedAuth = Base64.encodeBase64(auth.getBytes(Charset.forName("US-ASCII")));
	String authHeader = "Basic " + new String(encodedAuth);
	HttpHeaders requestHeaders = new HttpHeaders();
	requestHeaders.set("Authorization", authHeader);
	requestHeaders.setContentType(MediaType.APPLICATION_JSON);
	requestHeaders.set("Accept", MediaType.TEXT_PLAIN_VALUE);

	HttpEntity<List<? extends PublishData>> requestEntity =	new HttpEntity<List<? extends PublishData>>(publishDatas, headers);
	ResponseEntity<String> response = restTemplate.exchange(restResourceUrl, HttpMethod.POST, requestEntity, String.class);

working with JAXB marshalling for a class, we need to annotate the class with @XmlRootElement annotation. 	

Spring application supports both JSON as well as XML. It will even support XML request with JSON response and vice versa.  


## Postman tool  
Content-Type header is “application/xml” or “application/json”    
Accept header as “application/xml”  or “application/json”    

    	 
## SQL injection ##

SQL injection occurs when building a sql string by concateing parameter values.   
Solution: use SQL parameters in sql, named parameter or positional parameter. PreparedStatement and setXXXParameter().  


## hibernate ## 

- level 1 cache  
  a entity cache at session level. level 1 cache is enabled by hibernate by default.  

- level 2 cache  
  a entity cache at session factory level (shared by all sessions, application level cache). level 2 cache is not enabled by hibernate by default, need to be enabled.  

- early fetch, lazy fetch  
  early fetch hit the db and populate the entity object properties. lazy fetch initially create a proxy object with only ID in it. when a entity's property is accessed, it will then hit the DB to fetch the row data and populate the entity's properties.  

- get() vs load()  
	- get(class, id) is basic programming. It always eager fetch. get() return null object, no exception. 
	- load(class,id) is advanced programming. It always lazy fetch. If ID not exists in DB,  load() returns ObjectNotExistException when it really hits the DB to read.  

- @OnetoOne, @OneToMany, @ManyToMany
 
## Spring annotations @component vs @bean difference

- @Component annotation need NOT to be used with @Configuration class where as @Bean annotation has to be used within @Configuration class.

- @Component is a class level annotation. the constructor bears the logic responsible for creating the bean. @Bean is a method level annotation inside a @Configuraton class. The body of @Bean method bears the logic responsible for creating the bean

- @Component create a single bean of a class. @Bean can be used to create multiple beans with the same class type.

- We cannot create a bean of a third-party class using @Component where as we can create a bean of a third-party class using @Bean. 

 
## why use @repository not @component ##

@Repository’s job is to catch checked persistence exceptions and re-throw them as one of Spring’s unchecked DataAccessException.  
 
PersistenceExceptionTranslationPostProcessor bean. This bean post processor adds an advisor (AOP proxy) to any bean that’s annotated with @Repository so that any checked exceptions are caught and then rethrown as one of Spring’s unchecked DataAccessException.

	<bean id="persistenceExceptionTranslationPostProcessor" class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>

	
## why use @Controller not @Component ##

We can only use @RequestMapping on @Controller annotated classes.  
The DispatcherServlet scans the classes annotated with @Controller and detects @RequestMapping annotations within them. 
  
  
## spring-integration 

MRI sample - /orchestrator/src/main/resources/META-INF/spring/bcbs-svlm.xml   
MRI sample - /orchestrator/src/main/java/com/tdsecurities/stars/orchestrator/bcbs/TimelinessServiceActivator.java   
	  
Case Study: read a csv file, convert each line to a JMS XmlMessage, send them to a middleware queue 	  
Reference Implementation Steps:	  
1. source is inbound-channel (OS physical file) -> inbound-channel-adapter (adapt from OS file to java File object) -> input-channel(java File object) -> service activator (java.io.File) 

2. service activator: read java File object -> foreach line, conver a csv line to to a java entity objects -> JAXB2 marshall each java entity obj to a xml String) ->  output-channel.send(GenericMessage<String> xmlString)
 
3. output-channel (GenericMessage<String> xmlString) -> outbound-channel-adapter (adapt from GenericMessage<String> to JMS XmlMessage), using JmsTemplace, send JMS messages to middleware Queue -> Destination is outbound-channel(JMS messages in middleware Queue)  


## spring-batch

Case Study: read from database (sql or stored procedure), write to a csv file.  
MRI sample - /business-process-jobs/src/main/resources/com/tdsecurities/stars/bpe/file-extract


## design patterns  

builder, singleton, prototype, factory, abstract factory, adapter, wrapper, facade, proxy, chain of responsibility, , iterator, mediator, observer, visitor, strategy, memento, template
 

## thread pool. 
How will you implement a thread pool yourself.
Executors class is a factory class that create different types of thread pools..
Executor interface,  ExecutorService implements Executor, A ExecutorService instance is a thread pool.
 
 
## Thread 

java VOLATILE variable is thread safe.   
hashmap not thread safe. hashtable vs concurrentHashMap both are thread safe.   
arraylist not thread safe. vector is thread safe.  
memory space: heap vs stack
methods in java.lang.Object.  

HashMap key class must be immutable and implements hashcode() and equals()  

thread context switch, process context switch,   
thread scheduler, cpu time slice (=preemptive scheduling),   
CPU registors state save/restore, cpu's L1 cache, main L2 cache, main memory


### hardware thread, software thread   
https://www.codeguru.com/cpp/sample_chapter/article.php/c13533/Why-Too-Many-Threads-Hurts-Performance-and-What-to-do-About-It.htm

Number of hardware threads = (number of CPUS) * (number of cores per CPU) * (number of hardware threads per core : ususally 1 thread per core, 2 thread per core if CPU processor supports HyperThreading)

One hardware thread can run many software threads. In modern operating systems, this is often done by time-slicing. CPU L1 cache memory, main L2 cache memory contention if too many software threads -> slow down performance

 
## task-based programming 

executors.newWorkStealingThreadPool,    
task scheduling, work stealing, parallelism, forkjointask, forkjoinpool, 
 
 
## AOP  

AOP is a way for adding behavior to existing code without modifying that code. annotation driven, non-invasiveness.    

AOP solves cross-cutting concern
an aspect = pointcuts + advisors, 

Two types of AOP programming:
- Spring-AOP 
- AspectJ AOP

Both spring-AOP lib and AspectJ lib defines @Aspect/@Pointcut/@Around/@Before/@After/@AfterReturnning/@AfterThrowing annotations.  


### Spring-AOP (default)

Spring-AOP creates proxies at spring container starts up. It is Runtime weaving using proxy.

Spring-AOP will build a proxy for your objects, with a JDKDynamicProxy if your bean implements an interface or via CGLIB dynamic proxy to implementation class  if your bean doesn't implement any interface.

Spring-AOP is one of the essential parts of the spring framework. the spring framework is based on IoC and AOP. The AOP is one of the most important parts of the framework.

@Interface, @Aspect @Component or META-INFO/aop.xml,  @Around/@Before/@After, ProceedingJointPoint.proceed(), @Pointcut("execution(public * foo..*.*(..))"), @Pointcut("@annotation(annotation class name)"), @Pointcut("within(<package> or <interface>)")

<aop:aspectj-autoproxy/>: It enables @AspectJ style of aspect declaration, but AspectJ runtime is not used. It still use Spring AOP proxies. 


### AspectJ - configed by @enableAspectJAOPProxy on a @configuration class.
 
libs: aspectj-maven-plugin, org.aspectj.aspectjweaver.jar, org.aspectj.aspectjrt.jar, org.aspectj.aspectjtools.jar

AspectJ modifies class bytecodes at compilation time or spring container loading time. 

- Compile time weaving. It is done through AspectJ Java Tools(ajc compiler) if source available or post compilation weaving (using compiled class files). Compile time weaving can offer benefits of performance (in some cases).  

- Spring Framework's support for AspectJ LTW (Load Time Weaver). Load-time weaving (LTW) refers to the process of weaving AspectJ aspects into the class files as they are being loaded into the Java virtual machine (JVM). The focus of this section is on configuring and using LTW in the Spring application context container.


### Comparison

there are two major differences btwn Spring AOP and AspectJ AOP: 
- the type of weaving. 
  Spring approach is simpler and more manageable. But with the Spring AOP you can't use the all power of AOP because the implementation is done through proxies and not with modification of your bytecode.
  
  Spring AOP is basically a proxy created from the container, so you can use AOP only for spring beans. AspectJ AOP can be used to any java instances (beans or not-beans)

- joinpoint definition.
  the joinpoint definition in Spring-aop is restricted to method definition. While with AspectJ you can use the aspect in both methods and fields. 

- runtime performance
  If performance under high load is important, you'll want AspectJ which is 9-35x faster than Spring AOP. whether your aspects will be mission critical.

 
## @FunctionalInterface 

A java Interface class can contains default method, static method, abstract method  
A functional interface contains only one interface abstract method. It can contains other default and/or static interface methods.  

Functional object - Implementation of a @FunctionalInterface interface. 
A functional object can be assigned to a variable/parameter with a @FunctionalInterface interface type.

There are two category of functional objects:
- method reference: 
	- constructor reference:   .collect(Collectors.toCollection(TreeSet::new))
	- Static method reference, 
	- instance method reference of an object of a particular type,    Arrays.stream(a).map(Object::toString)
	- instance method reference of an existing object  

- lambda expression:
  Lambda is a single expression annonymous function that is often used as inline function. Java Lambda concept is borrowed from Python. Java internal mechanism is to create an instance of an annonymous class that implements a @FunctionalInterface interface. This ways, java effectively treat a Lambda expression as a function object.

  
## java 8 streaming
 
Collectors class is a factory class. It creates Collector instances that implement various reduction operations, such as accumulating elements into collections, summarizing elements according to various criteria, grouping, partitioning, etc.

filter, map, flatmap, sort, findfirst, findlast, collect

** public final class Collectors extends Object

The following are examples of using the predefined collectors to perform common mutable reduction tasks:

     // Accumulate names into a List
     List<String> list = people.stream().map(Person::getName).collect(Collectors.toList());

     // Accumulate names into a TreeSet
     Set<String> set = people.stream().map(Person::getName).collect(Collectors.toCollection(TreeSet::new));

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
		

## Unit test 

### @RunWith(SpringRunner.class)  

SpringRunner.class is an alias for the SpringJUnit4ClassRunner.class. This class requires JUnit 4.12 or higher.

It will create a spring app context container containing all the scanned beans. the test class is a bean in the container.   
It can be coded in hybrid:  
- The beans can use @Autowired to inject real(not mocked) beans in container. 
- If we want to inject some mocked beans, we can use @InjectMocks, @Mock, @Spy 
 
 
### @RunWith(MockitoJUnitRunner.class)

It will not create spring app context container. the test calsss can not use @Autowired. dependency are assigned explicitly in @setup method.

Sample:

     @InjectMocks  
     private AccountsController accountsController;  
     @Spy  
     private TransactionTrendRequestRestAPIMapper transactionTrendRequestRestAPIMapper;  
     @Mock  
     private AccountsService accountsService;  
     @Before  
     public void setup() {  
    	MockitoAnnotations.initMocks(this);  
     }  
     
	 
## spring-mvc unit test 

### spring-mvc test using MockMvc in spring-boot-test jar (do not need to start up web application)

use @AutoConfigureMockMvc:

    @RunWith(SpringRunner.class)  
	@SpringBootTest(classes={MyApplication.class}, webEnvironment={SpringBootTest.WebEnvironment.MOCK, WebEnvironment.RANDOM_PORT} )  
	@TestPropertySource(locations={"classpath:application-test.properties"})    
    @AutoConfigureMockMvc     
		
    @Autowired    
    private MockMvc mockMvc;  
    
or use manual setup:  

    org.springframework.test.web.servlet.MockMvc mockMvc =   					
		org.springframework.test.web.servlet.setup.MockMvcBuilders.standaloneSetup(accountsController)  
			.setMessageConverters(new StringHttpMessageConverter(), new MappingJackson2HttpMessageConverter())  
			.setControllerAdvice(advice)
			.build();

test web request-response:
			
    mockMvc.perform(post("/load/input?input_name=YDMD_SP_LDR_AB_Part1&bid=b2");       

	mockMvc.perform(get("/accounts/1/transactions/filterOptions")  
			.accept(MediaType.APPLICATION_JSON))
			.andDo(print())
			.andExpect(status().isOk())
		    .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8_VALUE))
			.andExpect(jsonPath("$.cardholderFilterOption[1]").exists())
            .andExpect(jsonPath("$.cardholderFilterOption[0].accountCustomerId").value("1"))

			
### spring-mvc test using io.restassured lib (need to start up web application first)		  

	@Test
    public void whenAddnewBook_thenSuccess() {
        final Book book = new Book();
        book.setTitle("How to spring cloud");
        book.setAuthor("Baeldung");
        // request the protected resource
        final Response bookResponse = RestAssured.
            .contentType(ContentType.JSON)
            .body(book)
            .post(ROOT_URI + "/book-service/books");
        final Book result = bookResponse.as(Book.class);
        Assert.assertEquals(HttpStatus.OK.value(), bookResponse.getStatusCode());
        Assert.assertEquals(book.getAuthor(), result.getAuthor());
        Assert.assertEquals(book.getTitle(), result.getTitle());
    }

	
## oracle ##

- heap table(== heap organized table),   
- clustered index (index organized table, the cluster key is the primary key, actual rows are stored in tree nodes),   
- non-clustered index (the tree nodes store the physical locations of the actual rows in disk)  
- b-tree index (on columns with high cardinality), used in frequent DML(upsert/delete) operations.  
- bitmap index (on columns with low cadinality), used in write-once read-many application, such as overnight batch load in DW. expensive to update bitmap index.  


## microservice, design patterns, RESTful API design  

### data join

Use two types of designs to handle data join in microservice architecture:  
- use public REST API to read data from other service, join data from other service and data from this servcie in the application,   
- use data event to publish data changes from the data's master service to other services. save the data in the service local db. then join in the service's local database,  


### distributed transactions design
Use workflow or sage designs to handle distributed transactions in microservice architecture.   
 
 
