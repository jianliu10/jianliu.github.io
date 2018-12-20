---
layout: post
title:  "Technical notes - 2018"
date:   2018-10-01 13:15:42 -0500
categories: tech java-spring
---

# 2018 technical notes #

## spring-boot ##
 (spring-boot-starter-web -> spring-mvcweb -> config filter chain -> last servlet is DispatcherServerlet -> route to a controller;   
 embedded app server, tomecat or jetty;  app server conf/server.xml defines two connectors: http connector (8080), https connector (443), different ports;  

> @SpringBootApplication(scanBasePackages={,,}   
> @SpringBootApplication is a convenience annotation that adds all of the following:   
> @Configuration, @EnableAutoConfiguration, @EnableWebMvc, @ComponentScan  
> @EnableAutoConfiguration tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.  

@EnableWebMvc annotation is added automatically when it sees spring-webmvc on the classpath. This flags the app as a web app and activates key behaviors such as DispatcherServlet.  

see java classes : WebMvcConfigurer, WebMvcConfigurerAdapter, WebMvcConfigurationSupport, WebMvcAutoConfiguration, WebMvcAutoConfigurationAdapter  

@PropertySources = {,,}, 
    
## spring Rest-API ws exception handling ##
@ControllerAdvice, @ExceptionHandler, ResponseEntity<Object>, ResponseEntityExceptionHandler  

	@ControllerAdvice
	public class MyResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {
		@ExceptionHandler(value = { IllegalArgumentException.class, IllegalStateException.class })
		protected ResponseEntity<Object> handleConflict(
		  RuntimeException ex, WebRequest request) {
			String bodyOfResponse = "This should be application specific";
			return handleExceptionInternal(ex, bodyOfResponse, new HttpHeaders(), HttpStatus.CONFLICT, request);
		}
	} 
 
## spring rest ws marshalling/unmarshalling data in http request/response body ##
@EnableWebMvc will created two beans RequestMappingHandlerMapping and RequestMappingHandlerAdapter
  
>     dispatchServlet -RequestMappingHandlerMapping bean maps from URI to handler object at type level   
>     				-RequestMappingHandlerAdapter bean maps from URI to method 

**REST ws server side,**   

annotation @EnableWebMvc will pre-enable the following HttpMessageConverters instances by default:  

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
 
**Custom Converters Configuration**  

		---- class based config @Configuration class 
		@Configuration
		@EnableWebMvc
		@ComponentScan({ "org.baeldung.web" })
		public class WebConfig implements WebMvcConfigurer {
			 @Override
			 public void configureMessageConverters(
				  List<HttpMessageConverter<?>converters) { 
				  // MappingJackson2HttpMessageConverter we can now set a custom ObjectMapper on this converter and have it configured as we need to.
				  // MarshallingHttpMessageConverter – and we’re using the Spring XStream support to configure it. This allows a great deal of flexibility since we’re working with the low level APIs of the underlying marshalling framework – in this case XStream
			}
		}


		---- xml based config	  
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
	
**REST ws client side, using RestTemplate	**  

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

working with JAXB marshalling for a class, we need to annotate it with @XmlRootElement annotation. 	
Our Spring application will support both JSON as well as XML. It will even support XML request with JSON response and vice versa.  

Postman application  
Content-Type header is “application/xml” or “application/json”    
Accept header as “application/xml”  or “application/json”    
    	 
## sql injection ##

  it occurs when building a sql string by concateing parameter values.  
  how to solve it: use named parameters in sql. PreparedStatement and setXXXParameter().  

## hibernate ## 

 level 1 cache: a entity cache at session level. level 1 cache is enabled by hibernate by default.  

 level 2 cache: a entity cache at session factory level (shared by all sessions, application level cache). level 2 cache is not enabled by hibernate by default, need to be enabled.  

 early fetch, lazy fetch: early fetch hit the db and populate the entity object properties. lazy fetch initially create a proxy object with only ID in it. when a entity's property is accessed, it will then hit the DB to fetch the row data and populate the entity's properties.  

 get vs load, : 1) get(class, id) is always eager fetch, load(class,id) is always lazy fetch; 2) if ID not exists in DB, get() return null object, no exception. load() returns ObjectNotExistException when it really hits the DB to read.  

## spring annotations, difference between @component and @bean ##

@Component auto-detects and auto-configure the beans using classpath scanning. @Bean explicitly declares a single bean, rather than letting Spring do it automatically.

@Component couples bean declaration with the class definition. @Bean decouples the bean declaration of the bean from the class definition.

@Component is a class level annotation.  @Bean is a method level annotation inside a 

@Configuraton java class. The body of @Bean method bears the logic responsible for creating the instance

@Component need not to be used with the @Configuration annotation where as @Bean annotation has to be used within the class which is annotated with @Configuration.
 We cannot create a bean of a third-party class using @Component where as we can create a bean of a third-party class using @Bean. 

@Component create one bean per class (Singleton). @Bean can be used to create multiple beans with the same class type.

 
## why use @repository not @component ##

@Repository’s job is to catch checked persistence exceptions and re-throw them as one of Spring’s  unchecked DataAccessException.  
 
bean - PersistenceExceptionTranslationPostProcessor. This bean post processor adds an advisor (AOP proxy) to any bean that’s annotated with @Repository so that any checked exceptions are caught and then rethrown as one of Spring’s unchecked data access exceptions.

    <bean id="persistenceExceptionTranslationPostProcessor" class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>

## why use @Controller not @Component ##

  The dispatcher scans the classes annotated with @Controller and detects @RequestMapping annotations within them. We can only use @RequestMapping on @Controller annotated classes.
  
## spring-integration: csv file => jms queue ## 

> mri:  
> /orchestrator/src/main/resources/META-INF/spring/bcbs-svlm.xml  
> /orchestrator/src/main/java/com/tdsecurities/stars/orchestrator/bcbs/TimelinessServiceActivator.java   
         
  Step 1. source: physical files -> inbound-channel-adapter (adapt from physical file to java File object) -> input-channel -> service activator (java.io.File) 

  Step 2. service activator: read file -> foreach line, conver a csv line to to a java entity objects -> JAXB2 marshall each java entity obj to a xml String) ->  output-channel.send(GenericMessage<String> xmlString)
 
  Step 3. outbound-channel-adapter (adapt from GenericMessage<String> to Jms TextMessage) -> using JmsTemplace, send to Destination (Messaging Queue or Topic)

## spring-batch: read from database (sql or stored procedure), write to a csv file. ## 
 
> mri: /business-process-jobs/src/main/resources/com/tdsecurities/stars/bpe/file-extract

## thread pool. How will you implement a thread pool yourself ##
 
## design patterns  ##

give an example, like builder, singleton, prototype, factory, abstract factory, adapter, wrapper, facade, proxy, chain of responsibility, , iterator, mediator, observer, visitor, strategy, memento, template
 
## Thread ##

java VOLATILE variable, hashtable vs concurrentHashMap, vector vs arraylist, heap vs stack, methods in java.lang.Object.
HashMap key class must be immutable and implements hashcode() and equals()

thread context switch, process context switch, thread scheduler, cpu time slice (=preemptive scheduling), CPU registors state save/restore, cpu's L1 cache, main L2 cache, main memory

### hardware thread, software thread ### 

 https://www.codeguru.com/cpp/sample_chapter/article.php/c13533/Why-Too-Many-Threads-Hurts-Performance-and-What-to-do-About-It.htm

 number of hardware threads = (number of CPUS) * (number of cores per CPU) * (number of hardware threads per core : ususally 1 thread per core, 2 thread per core if CPU processor supports HyperThreading)

 One hardware thread can run many software threads. In modern operating systems, this is often done by time-slicing. CPU L1 cache memory, main L2 cache memory contention if too many software threads -> slow down performance
 
## task-based programming ##

task scheduling, work stealing, parallelism, forkjointask, forkjoinpool, executors.newWorkStealingThreadPool,  
 
## reactive-based programming ## 

stream programming. inside a process, using a blocking queue, use events. 
 
reactive model, reactive architecture design, messaging queue, distributed, auto scale and deploy independently, container orchestrator kubernetes (k8s) 
 
java Collectors class implements various reduction methods, such as accumulating elements into collections, summarizing elements according to various criteria.
 
## AOP, cross-cutting concern, pointcut, advise, ## 

 AOP is a way for adding behavior to existing code without modifying that code. annotation driven, non-invasiveness  
 two types of AOP programming, Spring-AOP vs AspectJ. @Aspect/@Pointcut/@Around/  @AfterThrowing can be annotation classes definted in Spring AOP lib or AspectJ lib.  

***Spring-AOP (default)***

Spring-AOP: performed by the container at container start-up. Runtime weaving through proxy using concept of jdk dynamic proxy if interface exists or cglib dynamic proxy if direct implementation provided.
 
when you write an Aspect with Spring, spring-aop will build a proxy for your objects, with a JDKDynamicProxy if your bean implements an interface or via CGLIB if your bean doesn't implement any interface.

Spring AOP is one of the essential parts of the spring framework.the spring framework is based on IoC and AOP. The AOP is one of the most important parts of the framework.

@Interface, @Aspect @Component or META-INFO/aop.xml,  @Around/@Before/@After, ProceedingJointPoint.proceed(), @Pointcut("execution(public * foo..*.*(..))"), @Pointcut("@annotation(annotation class name)"), @Pointcut("within(<package> or <interface>)")

<aop:aspectj-autoproxy/>: using it will result in the creation of Spring AOP proxies. The @AspectJ style of aspect declaration is just being used here, but the AspectJ runtime is not involved.


***AspectJ - configed by @enableAspectJAOPProxy on a @configuration class.*** 
 
coding: aspectj-maven-plugin, org.aspectj.aspectjweaver.jar, org.aspectj.aspectjrt.jar, org.aspectj.aspectjtools.jar

there are two major differences: One is related to the type of weaving. Another to the joinpoint definition.

AspectJ: through bytecode modification.   
Compile time weaving through AspectJ Java Tools(ajc compiler) if source available or post compilation weaving (using compiled class files). Compile time weaving can offer benefits of performance (in some cases).  
or Spring Framework's support for AspectJ LTW (Load Time Weaver). Load-time weaving (LTW) refers to the process of weaving AspectJ aspects into an application's class files as they are being loaded into the Java virtual machine (JVM). The focus of this section is on configuring and using LTW in the Spring application context container.

the joinpoint definition in Spring-aop is restricted to method definition only which is not the case for AspectJ.

 ***Comparison***

Spring approach is simpler and more manageable. with the Spring AOP you can't use the all power of AOP because the implementation is done through proxies and not with through modification of your code.

Spring AOP is basically a proxy created from the container, so you can use AOP only for spring beans. Spring-aop is restricted to method definition only. While with AspectJ you can use the aspect in all your beans.

If performance under high load is important, you'll want AspectJ which is 9-35x faster than Spring AOP. whether your aspects will be mission critical.

 
## @FunctionalInterface ## 

interface default method, interface static method, method reference  
functional interface contains only one abstract interface method. It can have other default and/or static interface methods.  

method references: Static method reference, instance method reference of an object of a particular type, instance method reference of an existing object  

Lambda is a single expression annonymous function that is often used as inline function. Java Lambda concept is borrowed from Python. Java internal mechanism is to create an instance of an annonymous class that implements a FunctionalInterface. This ways, java effectively treat a Lambda expression as a function object.

## Unit test ##

 @RunWith(Spring4JunitRunner.class) -- deprecated. Replace with SpringRunner.class  

 @RunWith(SpringRunner.class) -- it will create a spring app context container containing all the scanned beans. the test class is a bean in the container.   
 It can code in hybrid:  can use @Autowired to inject real(not mocked) beans in container. If we want to inject some mocked beans, we can use @InjectMocks, @Mock, @Spy,   MockitoAnnotations.initMocks(this)  
 
     @RunWith(MockitoJUnitRunner.class) -- it will not create spring app context container. the test calsss can not use @Autowired. dependency are assigned explicitly in @setup method.
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
     
## spring-mvc test ## 

using the MockMvc in spring-boot-test jar (do not need to start up web application)

    @RunWith(SpringRunner.class)  
	@SpringBootTest(classes={MyApplication.class}, webEnvironment={SpringBootTest.WebEnvironment.MOCK, WebEnvironment.RANDOM_PORT} )  
	@TestPropertySource(locations = "classpath:application-test.properties")    
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
- b-tree index (on columns with high cardinality), used in frequent DML operations.  
- bitmap index (on columns with low cadinality), used in write-once read-many application, such as overnight batch load in DW. expensive to update bitmap index.  


## microservice, design patterns, RESTful API design  ## 

**data join**  
Use two types of designs to handle data join in microservice architecture:  

- use public REST API to read data from other service, join data from other service and data from this servcie in client application,   
- use data event to publish data changes from the data's master service to other services. cache the data in the service local db. then join in the service's local database,  


**distributed transactions design**  
Use workflow or sage designs to handle distributed transactions in microservice architecture.   
 
 
 