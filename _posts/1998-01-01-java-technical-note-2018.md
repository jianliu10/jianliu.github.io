---
layout: post
title:  "Java-Spring Technical notes - 2018"
date:   2018-10-01 13:15:42 -0500
categories: tech java-spring
---

# 2018 technical notes #

## design patterns  

builder, singleton, prototype, factory, abstract factory, adapter, wrapper, facade, proxy, chain of responsibility, , iterator, mediator, observer, visitor, strategy, memento, template
 

## Spring framework two essential features:
- IOC : Inversion of Control,  
  @Component, auto create beans in container based on declaration
- DI : Dependency Injection,  
  container engine auto search for beans with specific type or name, and assign beans to variables.
  @Autowired, @Qualifier, field injection, constructor/method parameter injection, 
  @Value - inject values from property files
- AOP: Aspect Oriented Programming

Spring bean scopes: singleton, prototype, request, session, spring-batch's step

## spring-boot ##
 (spring-boot-starter-web -> spring-mvcweb -> config filter chain -> last servlet is DispatcherServerlet -> route to a controller;   
 embedded app server, tomecat or jetty;  app server conf/server.xml defines two connectors: http connector (8080), https connector (443), different ports;  

> @SpringBootApplication(scanBasePackages={,,}   
> @SpringBootApplication is a convenience annotation that adds all of the following:   
> @Configuration, @EnableAutoConfiguration, @ComponentScan, @EnableWebMvc,   
> @EnableAutoConfiguration tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.  
> @EnableWebMvc annotation is added automatically when it sees spring-webmvc jar on the classpath. This flags the app as a web app and activates key behaviors such as DispatcherServlet.  

see java classes : WebMvcConfigurer, WebMvcConfigurerAdapter, WebMvcConfigurationSupport, WebMvcAutoConfiguration, WebMvcAutoConfigurationAdapter  

@PropertySources = {,,}, 

Spring has two types of beans:
- configuration beans 
- regular beans


## Spring REST-API WS server side 
   
### @ControllerAdvice 

#### ExceptionHandler   
classes: ResponseEntity<Object>, ResponseEntityExceptionHandler  

	@ControllerAdvice
	public class MyResponseEntityExceptionHandler  {
	
		@ExceptionHandler(value = { IllegalArgumentException.class, IllegalStateException.class })
		protected ResponseEntity<Object> handleConflict(
		  RuntimeException ex, WebRequest request) {
			String bodyOfResponse = "This should be application specific";
			return handleExceptionInternal(ex, bodyOfResponse, new HttpHeaders(), HttpStatus.CONFLICT, request);
		}
	} 
 

 #### ResponseBodyAdvice
@ControllerAdvice
public class SurrogateAdvice<T> implements ResponseBodyAdvice<T> {

    @Override
    public T beforeBodyWrite(T body, MethodParameter returnType, MediaType selectedContentType,
        Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
		...
        surrogateMapping.surrogate(fieldsWithSurrogateId, body);

        return body;
    }
 

#### RequestBodyAdvice
@ControllerAdvice
public class DecryptionAdvice implements RequestBodyAdvice {

    @Override
    public HttpInputMessage beforeBodyRead(HttpInputMessage inputMessage, MethodParameter parameter, Type targetType, Class<? extends HttpMessageConverter<?>> converterType) throws IOException {
        return inputMessage;
    }

    @Override
    public Object afterBodyRead(Object body, HttpInputMessage inputMessage, MethodParameter parameter, Type targetType, Class<? extends HttpMessageConverter<?>> converterType) {
        encryptionUtil.decryptFieldsInObject(body, CaseInsensitiveMap.forMap(inputMessage.getHeaders().toSingleValueMap()));
        return body;
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
	
	
## Spring REST-API WS client side, using RestTemplate, or use swagger-code-gen to generate feign client.	

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

import or export test case collection to a json file.

command line to run postman test case collection: newman run <path to postman-collection.json>  

    	 
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
 
## Spring @Component
In Spring @Component, @Service, @Controller, and @Repository are Stereotype annotations
@Component: generic annotation
@Controller: url request mapping
@Service: business logic
@Repository: Persistence layer(Data Access Layer) 
 
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

<aop:aspectj-autoproxy/>: It enables @AspectJ style of aspect declaration, but AspectJ runtime is not used. It still use Spring AOP proxies because of 'aop:" namespace. 


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

 
## Unit test 

### @RunWith(SpringRunner.class)  

SpringRunner.class is an alias for the SpringJUnit4ClassRunner.class. This class requires JUnit 4.12 or higher.

It will create a spring app context container containing all the scanned beans. the test class is a bean in the container.   
It can be coded in hybrid:  
- The beans can use @Autowired to inject real(not mocked) beans in container. 
- If we want to inject some mocked beans, we can use @InjectMocks, @Mock, @Spy 
- need to call MockitoAnnotations.initMocks(this) explicitly to inject mocks in @Before setup() method

@Mock - create a stub (a dummy object with no state, void method body).
@Spy - create a real object with memeber varialbes and method implementations.

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

	 
### @RunWith(MockitoJUnitRunner.class)

It will not create spring app context container. the test calsss can not use @Autowired. dependency are assigned explicitly in @setup method.
do NOT need to call 'MockitoAnnotations.initMocks(this)' explicitly to injfect mocks in @Before setup() method

     
	 
## spring-mvc unit test 

### spring-mvc test using MockMvc in spring-boot-test jar (do not need to start up web application)

use @AutoConfigureMockMvc:

    @RunWith(SpringRunner.class)  
	@SpringBootTest(classes={MyApplication.class}, webEnvironment=WebEnvironment.MOCK )  
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

	
## database 

### tables

- heap organized table (heap table), This is a standard Oracle table; 
  A heap-organized table is a table with rows stored in no particular order. the term "heap" is used to differentiate it from an index-organized table or external table. If a row is moved within a heap-organized table, the row's ROWID will also change.  
  
- index organized table (clustered index), the cluster key is the primary key, actual rows are stored in tree nodes.   

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

### index
- clustered index (index organized table)
- non-clustered index (the tree nodes store the physical locations of the actual rows in disk)  
- b-tree index (on columns with high cardinality), used in frequent DML(upsert/delete) operations.  
- bitmap index (on columns with low cadinality), used in write-once read-many application, such as overnight batch load in DW. expensive to update bitmap index.  


### performance tunning  
execute the sql -> run "explain plan" -> analyze the dumped execution plan.

Speed (hash join is faster since it access memory than b-tree index on disk:
	HASH JOIN > SORT-MERGE join > NESTED LOOPS join with b-tree index on second table. 
memory consumption in temp tablespece (hash join use more memory):
	HASH JOIN == SORT-MERGE join > NESTED LOOPS join with b-tree index on second table. 

analyze execute time, cost-based optimization (always in Oracle)


### Oracle HASH Joins

HASH joins are the usual choice of the Oracle optimizer when the memory is set up to accommodate them. In a HASH join, Oracle accesses one table (usually the smaller of the joined results) and builds a hash table on the join key in memory. It then scans the other table in the join (usually the larger one) and probes the hash table for matches to it. Oracle uses a HASH join efficiently only if the parameter PGA_AGGREGATE_TARGET is set to a large enough value. If MEMORY_TARGET is used, the PGA_AGGREGATE_TARGET is included in the MEMORY_TARGET, but you may still want to set a minimum.

If you set the SGA_TARGET, you must set the PGA_AGGREGATE_TARGET as the SGA_TARGET does not include the PGA (unless you use MEMORY_TARGET as just described). The HASH join is similar to a NESTED LOOPS join in the sense that there is a nested loop that occurs—Oracle first builds a hash table to facilitate the operation and then loops through the hash table. When using an ORDERED hint, the first table in the FROM clause is the table used to build the hash table.

HASH joins can be effective when the lack of a useful index renders NESTED LOOPS joins inefficient. The HASH join might be faster than a SORT-MERGE join, in this case, because only one row source needs to be sorted, and it could possibly be faster than a NESTED LOOPS join because probing a hash table in memory can be faster than traversing a b-tree index.

As with SORT-MERGE joins and CLUSTER joins, HASH joins work only on equijoins. As with SORT-MERGE joins, HASH joins use memory resources and can drive up I/O in the temporary tablespace if the sort memory is not sufficient (which can cause this join method to be extremely slow).

Finally, HASH joins are available only when cost-based optimization is used (which should be 100 percent of the time for your application running on Oracle 11g).

Table 1 illustrates the method of executing the query shown in the listing that follows when a HASH join is used.

	select /*+ ordered */ ename, dept.deptno
	from emp, dept
	where emp.deptno = dept.deptno

## Oracle

### Oracle 11G vs 12C

Oracle 11g was released in 2008. G stands for grid. 
- No cloud service, It Has no pluggable databases, there is no multitenant architecture, No in-memory capabilities, Has no JSON type support, Comparatively lower performance in I/O throughput and response time.

Oracle 12c was released in 2014, C stands for cloud.    
- designed for the cloud. It provides Oracle database cloud service, 
- It provides pluggable databases to support rapid provisioning and portability. It allows running multiple databases on the same hardware while maintaining the security and isolation among the databases.
- there is multitenant architecture. It enables an Oracle database to function as a multitenant container database (CDB)
- ** Has in-memory capabilities that provide real-time analytics, **
- added JSON data type support, 
- Comparatively higher performance in I/O throughput and response time.


### Oracle exadata?

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

Types of Dimensions :
- Conformed Dimension - separate dimension tables. creating consistency. The same dim table can be referenced by the multiple fact tables.
- Degenerated Dimension – the dimension attributes are stored as part of the fact table and not in a separate dimension table. Usually used when a dimension has only one attribute.
- Junk Dimension – A junk dimension is a single dimensional table with a combination of different and unrelated attributes. This is to avoid having a large number of foreign keys in the fact table. 
- Role play dimension – It is a dimension table that has multiple valid relationships with a fact table. For example, a fact table may include foreign keys for both ship date and delivery date. But the same dimension attributes apply to each foreign key so the same dimension tables can be joined to the foreign keys.

* Slowly Changing Dimensions table types:
- Type 1  is to over write the old value. (no history, overwrite the row) 
- Type 2 is to add a new row. (keep history. Add columns "active_flag, effective_start_date, effective_end_date", multiple rows) 
- Type 3 is to create a new column. (new value, old value in one rows)


### fact types & fact tabel types

Types of Fact attributes:
- Additive: Additive fact attributes can be summed up through ALL of the dimensions in the fact table.
- Semi-Additive: Semi-additive facts attributes can be summed up for some of the dimensions in the fact table, but not the others.
- Non-Additive: Non-additive facts attributes cannot be summed up for any of the dimensions present in the fact table.

Types of Fact Tables:
- Cumulative: This type of fact table describes what has happened over a period of time. For example, this fact table may describe the total sales by product by store by day. The facts for this type of fact tables are mostly additive facts. The first example presented here is a cumulative fact table.
- Snapshot: This type of fact table describes the state of things in a particular instance of time, and usually includes more semi-additive and non-additive facts. The second example presented here is a snapshot fact table.


## microservice, design patterns, RESTful API design  

### data join

Use two types of designs to handle data join in microservice architecture:  
- use public REST API to read data from other service, join data from other service and data from this servcie in the application,   
- use data event to publish data changes from the data's master service to other services. save the data in the other service local db. then join in the service's local database,  


### distributed transactions design
Always avoid distributed transactions. Instread, always use local transaction.

 
 
