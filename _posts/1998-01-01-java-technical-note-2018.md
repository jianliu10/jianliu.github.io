---
layout: post
title:  "Java-Spring Technical notes - 2018"
date:   2018-10-01 13:15:42 -0500
categories: tech-java-spring
---

# 2018 technical notes #

## OO

PEI:  
- polymorphism
- Encapsulation (Information hiding)
- Inheritance


## design patterns  

builder, singleton, prototype, factory, abstract factory, adapter, wrapper, facade (a interface of a subsystem), proxy (stub to a remote object), chain of responsibility, iterator, mediator (broadcast), observer, visitor (visit a tree), strategy (algorithm switch), memento (state persistence), template
 
adapter vs wrapper:
adapter is used to convert a external interface provided by a third party lib to a internal interface expected by the app code. The external interface declaration and interface interface declaration are different.
wrapper and wrappee provides similar interface declaration. A wrapper is used to modified, enrich, extend a wrappee's behavior. 

chain of responsibility: a request is passed down a chain of objects. each object in the chain is responsible of processing a specific task.


## Spring framework two essential features

- IOC : Inversion of Control,  
  Spring automatically creates beans in a container by using declarations. There is no need to write code to explicitly create objects.  
    xml based declaration,  class based declaration
    
- DI : Dependency Injection,  
  Given annonation declarations like @Autowired, @Qualifier, @Value, Spring automatically searches for beans with specific type or name, and assign beans to variables like class fields, method params. There is no need to write code to explicitly assigned value to variables.  
  
  @Autowired, @Qualifier - inject beans to class fields, constructor/method parameters.   
  
  @Value - inject configuration values from property files  
  
  @EnableConfigurationProperties, @ConfigurationProperties(prefix="..."), inject configuration properties from property files
  
- AOP: Aspect Oriented Programming
  @EnableAspectJAutoProxy - use ApectJ style AOP declarations like @Apect, @Pointcut, @Advise, @Around, @Before, @After, @AfterThrowing, @AfterReturning etc. while runtime still using Spring's proxy based AOP. 
  need to include spring-aspects.jar when using @EnableAspectJAutoProxy, which handles components marked with AspectJ's @Aspect annotation

Spring bean scopes: singleton, prototype, request, session, spring-batch's step

Spring has two types of beans:
- configuration beans 
- regular beans


## spring-5 configuration injection

see DECAF xml-json-converter project.

	@EnableConfigurationProperties
	@PropertySources = {,,}, 
	@TestPropertySource("classpath:application-test.properties")

	@Bean
	@ConfigurationProperties(prefix = "app.aws.s3")
	
	@Value("")


## spring-boot app

embedded app server, tomcat or jetty;  app server conf/server.xml defines two connectors: http connector (8080), https connector (443), different ports;  

> @SpringBootApplication(scanBasePackages={,,}   
> @SpringBootApplication is a convenience annotation that adds all of the following:   
> @Configuration, @EnableAutoConfiguration, @ComponentScan   
> @EnableAutoConfiguration tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.  


## spring-security

@EnableWebSecurity

when spring-boot-starter-security jar or spring-security jar is in classpath, spring-boot app automatically adds @EnableWebSecurity annotation.   
@EnableWebSecurity will activate key behaviors such as building HttpSecurity instance for SpringSecurityFilterChain (a DelegatingFilterProxy)

		@EnableWebSecurity
		public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
			@Override
			protected void configure(HttpSecurity http) throws Exception {
				http.authorizeRequests()
					.requestMatchers(protectedRequestsMatcher)
					.access("hasAuthority('" + ISSO_USER.getAuthority() + "')")
					.anyRequest().permitAll()

					.and()
					.cors()
					.and()
					.csrf().disable()
					.httpBasic().disable()
					.anonymous().disable()
					.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)	
			}

			@Override
			public void configure(WebSecurity web) throws Exception {
				super.configure(web);
				web.ignoring().antMatchers(errorPath).antMatchers("/health", "/info");
			}	
		}


## Spring REST-API WS service side

### @EnableWebMvc

when spring-boot-starter-web jar or spring-webmvc jar is in classpath, spring-boot automatically adds @EnableWebMvc annotation.    
@EnableWebMvc flags the app as a web app and activates key behaviors such as DispatcherServlet.

spring-boot-starter-web -> spring-webmvc -> config filter chain -> last servlet is DispatcherServerlet -> route to a controller;   

java classes:   
WebMvcConfigurer interface, WebMvcConfigurerAdapter abstract class, WebMvcConfigurationSupport impl class,   
WebMvcAutoConfiguration, WebMvcAutoConfigurationAdapter  


### maps from URI to controllers and methods

@EnableWebMvc will auto created two beans RequestMappingHandlerMapping and RequestMappingHandlerAdapter
  
DispatcherServlet uses the two beans to map URI:

-RequestMappingHandlerMapping bean maps from URI to handler object at type level   
-RequestMappingHandlerAdapter bean maps from URI to handler method 


### HttpMessageConverters

HttpMessageConverters marshall/unmarshall data from POJO to/from http request/response body

#### default HttpMessageConverters. 

@EnableWebMvc will pre-enable the following HttpMessageConverters instances by default:  

	ByteArrayHttpMessageConverter – converts java byte arrays to/from octet-stream
	StringHttpMessageConverter – converts Java Strings to/from text/plain
	ResourceHttpMessageConverter – converts org.springframework.core.io.Resource to/from octet-stream
	SourceHttpMessageConverter – converts javax.xml.transform.Source
	FormHttpMessageConverter – converts MultiValueMap<String, Collection> to/from form data
	Jaxb2RootElementHttpMessageConverter – converts Java objects to/from XML (added only if JAXB2 is present on the classpath)
	MappingJackson2HttpMessageConverter – converts Java objects to/from JSON (added only if Jackson2 is present on the classpath)
	MappingJacksonHttpMessageConverter – converts Java objects to/from JSON (added only if Jackson is present on the classpath)
	AtomFeedHttpMessageConverter – converts Java objects to/from Atom feeds (added only if Rome is present on the classpath)
	RssChannelHttpMessageConverter – converts Java objects to/from RSS feeds (added only if Rome is present on the classpath)
 
#### Customize HttpMessageConverters. 

class based configuration sample:

		@Configuration
		@EnableWebMvc
		@ComponentScan({ "org.baeldung.web" })
		public class WebConfig implements WebMvcConfigurer {
			 @Override
			 public void configureMessageConverters(List<HttpMessageConverter<?> converters) { 
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
	
	
### ControllerAdvice component class

@ControllerAdvice annotation

classes: ResponseEntity<Object>, ResponseEntityExceptionHandler  

1. @ExceptionHandler annotation  
	
		@ControllerAdvice  
		public class MyResponseEntityExceptionHandler  {
		
			@ExceptionHandler(value = { IllegalArgumentException.class, IllegalStateException.class })
			protected ResponseEntity<Object> handleConflict(
			  RuntimeException ex, WebRequest request) {
				String bodyOfResponse = "This should be application specific";
				return handleExceptionInternal(ex, bodyOfResponse, new HttpHeaders(), HttpStatus.CONFLICT, request);
			}
		} 
 
2. RequestBodyAdvice interface
	
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

3. ResponseBodyAdvice interface
	
		@ControllerAdvice
		public class SurrogateAdvice<T> implements ResponseBodyAdvice<T> {

			@Override
			public T beforeBodyWrite(T body, MethodParameter returnType, MediaType selectedContentType,
				Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
				...
				surrogateMapping.surrogate(fieldsWithSurrogateId, body);

				return body;
			}
		} 

	
## Spring REST-API WS client side  

1. use swagger-code-gen maven plugin

use maven plugin to generate ApiClient and api model classes from swagger2 API.   
ApiClient use apache http client package to do remote http request/response.

2. use Spring RestTemplate, 

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

Spring application supports both JSON as well as XML. It will even support XML request with JSON response and vice versa, using content-type and Accept http headers, using @RestMapping(consumes="", produces="")


## Postman tool  
Content-Type header is “application/xml” or “application/json”    
Accept header as “application/xml”  or “application/json”  

steps
1. import or export test case collection to a json file.
2. command line to run postman test case collection: "newman run &lt;path to postman-collection.json&gt;"  

    	 
## SQL injection ##

SQL injection occurs when building a sql string by concateing parameter values.   
Solution: use parameterized SQL statment, either named parameter or positional parameter. PreparedStatement and setXXXParameter().  


## Spring @Component

In Spring @Component, @Service, @Controller, and @Repository are Stereotype annotations

- @Component: generic annotation. It can also be used in lieu of @Configuration for creating beans w/o reference to other beans.
- @Controller: url request mapping
- @Service: business logic
- @Repository: Persistence layer(Data Access Layer) 
 
## Spring annotations @component vs @bean difference

- @Component annotation need NOT to be used with @Configuration class where as @Bean annotation has to be used within @Configuration class.

- @Component is a class level annotation. the constructor bears the logic responsible for creating the bean. @Bean is a method level annotation inside a @Configuraton class. The body of @Bean method bears the logic responsible for creating the bean

- @Component create a single bean of a class. @Bean can be used to create multiple beans with the same class type.

- We cannot create a bean of a third-party class using @Component where as we can create a bean of a third-party class using @Bean. 


## @Controller

**why use @Controller not @Component?** We can only use @RequestMapping on @Controller annotated classes.    

spring-webmvc scans the classes annotated with @Controller and create RequestMappingHandlerMapping for the controllers.   
spring-webmvc scans @RequestMapping annotations within the controller classes, and create RequestMappingHandlerAdapter for the methods.   
  
 
## persistence entity model classes

@OnetoOne, @OneToMany, @ManyToMany  

		import javax.persistence.Column;
		import javax.persistence.EntityListeners;
		import javax.persistence.GeneratedValue;
		import javax.persistence.GenerationType;
		import javax.persistence.Id;
		import javax.persistence.MappedSuperclass;
		import javax.persistence.Temporal;
		import javax.persistence.TemporalType;
		import javax.persistence.Version;
		import javax.persistence.JoinColumn;
		import javax.persistence.OneToOne;
		import javax.persistence.Table;


		import org.springframework.data.annotation.CreatedBy;
		import org.springframework.data.annotation.CreatedDate;
		import org.springframework.data.annotation.LastModifiedBy;
		import org.springframework.data.annotation.LastModifiedDate;
		import org.springframework.data.jpa.domain.support.AuditingEntityListener;

		/**
		 * A base entity class that provides auditing properties.
		 */
		@MappedSuperclass
		@EntityListeners(AuditingEntityListener.class)
		public abstract class BaseEntity<U> {

			@Id
			@Column(name = "id", nullable = false)
			@GeneratedValue(strategy = GenerationType.IDENTITY)
			private Long id;

			@CreatedBy
			@Column(name = "crtd_by")
			private U createdBy;

			@CreatedDate
			@Temporal(TemporalType.TIMESTAMP)
			@Column(name = "crtd_date")
			private Date createdDate;

			@LastModifiedBy
			@Column(name = "last_mdfd_by")
			private U lastModifiedBy;

			@LastModifiedDate
			@Temporal(TemporalType.TIMESTAMP)
			@Column(name = "last_mdfd_date")
			private Date lastModifiedDate;

			@Version
			@Column(name = "vrsn")
			private Integer version;
		}

		
		@Entity
		@Table(name = "aplcn")
		public class ApplicationEntity extends BaseEntity<String> {

			@Basic
			@Column(name = "code", nullable = false, updatable = false, length = 255)
			@NotNull @NotEmpty
			private String code;

			@Basic
			@Column(name = "name", nullable = false, length = 30)
			@NotNull @NotEmpty
			private String name;

			@ManyToOne(fetch = FetchType.LAZY)
			@JoinColumn(name = "dept_id")
			private DepartmentEntity department;

			@OneToMany(fetch = FetchType.LAZY, mappedBy = "application", cascade = CascadeType.ALL, orphanRemoval = true)
			private Collection<ApplicationSpaceEntity> directories = new ArrayList<>();
		}	
	
	
		@Entity
		@Table(name = "aplcn_space")
		public class ApplicationSpaceEntity extends BaseEntity<String> {
		
			@ManyToOne
			@JoinColumn(name = "aplcn_id", referencedColumnName = "id")
			private ApplicationEntity application;
			
			@OneToOne(fetch = FetchType.LAZY, mappedBy = "space", cascade = CascadeType.ALL, orphanRemoval = true)
			private MetadataEntity metadata;
		}
	
 
		@Entity
		@Table(name = "aplcn_space_mtdt")
		public class MetadataEntity extends BaseEntity<String> {

			@OneToOne(fetch = FetchType.LAZY)
			@JoinColumn(name = "aplcn_space_id", referencedColumnName = "id", nullable = false)
			private ApplicationSpaceEntity space;
		}

	
## why use @repository not @component 

- @EnableJpaRepositories(basePackages="...")  
  with @EnableJpaRepositories, Spring-data automatically creates repository implementation classes for each of the scanned Repository interfaces. 

		@Configuration
		@EnableTransactionManagement
		@EnableJpaAuditing
		@EnableJpaRepositories(basePackages = "org.finra.filex.dal.newdal")
		public class DaoConfig {

			@Primary
			@Bean(name = "dataSource")
			@ConfigurationProperties(prefix = "spring.datasource.hikari")
			public DataSource dataSource() {
				return DataSourceBuilder.create().type(HikariDataSource.class).build();
			}

			@Primary
			@Bean(name = "entityManagerFactory")
			public LocalContainerEntityManagerFactoryBean entityManagerFactory(EntityManagerFactoryBuilder builder, @Qualifier("dataSource") DataSource dataSource, JpaProperties jpaProperties) {
				LocalContainerEntityManagerFactoryBean factoryBean = new LocalContainerEntityManagerFactoryBean();
				factoryBean.setDataSource(dataSource);
				factoryBean.setPackagesToScan("org.finra.filex.model.newjpa");
				factoryBean.setPersistenceUnitName("dao");

				HibernateJpaVendorAdapter adapter = new HibernateJpaVendorAdapter();
				adapter.setDatabasePlatform(jpaProperties.getDatabasePlatform());
				adapter.setGenerateDdl(jpaProperties.isGenerateDdl());
				adapter.setShowSql(jpaProperties.isShowSql());
				factoryBean.setJpaVendorAdapter(adapter);

				factoryBean.setJpaPropertyMap(jpaProperties.getProperties());
				Map<String, Object> hibernateProperties = jpaProperties.getHibernateProperties(new HibernateSettings());
				factoryBean.setJpaPropertyMap(hibernateProperties);
				return factoryBean;
			}

			@Primary
			@Bean(name = "transactionManager")
			public PlatformTransactionManager transactionManager(@Qualifier("entityManagerFactory") EntityManagerFactory entityManagerFactory) {
				JpaTransactionManager manager = new JpaTransactionManager(entityManagerFactory);
				return manager;
			}
		}	

	
- one EntityManagerFactory bean for one persistence unit
  One persistence unit includes: data source, entity model classes, JPA properties, JPA vendor properties.
	
- @Repository not only create a singleton bean, but create a Aspect.   

  @Repository will create a AOP proxy for a repository singleton bean. The AOP proxy addes a AfterThrowing advice (PersistenceExceptionTranslationPostProcessor bean) to pointcuts(repository bean methods ). The AfterThrowing advice catches vendor specific persistence exceptions and re-throw them as oSpring’s unchecked persistence Exception.  
 
	<bean id="persistenceExceptionTranslationPostProcessor" class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>

	
## hibernate - JPA implementation provider

- level 1 cache  
  a entity cache at session level. level 1 cache is enabled by hibernate by default.  

- level 2 cache  
  a entity cache at session factory level (shared by all sessions, application level cache). level 2 cache is not enabled by hibernate by default, need to be enabled.  

- early fetch, lazy fetch  
  early fetch hit the db and populate the entity object properties. lazy fetch initially create a proxy object with only ID in it. when a entity's property is accessed, it will then hit the DB to fetch the row data and populate the entity's properties.  

- get() vs load()  
	- get(class, id) is basic programming, eager and lenient. It always eager fetch. if ID is not found in db, get() return null object, no exception. 
	- load(class,id) is advanced programming, lazy and strict. It always lazy fetch. If ID is not found in DB,  load() returns ObjectNotExistException when it really hits the DB to read.  


## spring-data for JPA

		public interface ApplicationRepository extends JpaRepository<ApplicationEntity, Long>   
		public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T>
		public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID>  
		public interface CrudRepository<T, ID> extends Repository<T, ID>

		public interface ApplicationRepository extends JpaRepository<ApplicationEntity, Long> {
			Optional<ApplicationEntity> findByCodeIgnoreCase(String appCode);
			Collection<ApplicationEntity> findAllByDepartment_code(String deptCode);
			Optional<ApplicationEntity> findByDepartment_codeIgnoreCaseAndCodeIgnoreCase(String deptCode, String appCode);
			Optional<ApplicationEntity> findByDirectories_trackings_TrackingId(String trackingId);
		}

	
## spring-data for mongodb

		@Configuration
		@EnableMongoRepositories(basePackages = "intact.clf.dc.dal")
		//@EnableTransactionManagement
		@EnableMongoAuditing
		public class DaoConfig { }
		
		@Document(collection="DomPolicyCollection")
		@CompoundIndex(name="dataSetIdx", def="{agent: 1, sourceSystem: 1, sourceDataSetId: -1}")
		@CompoundIndex(name="businessKeyIdx", def="{agent: 1, sourceSystem: 1, policyBusinessKey: 1}")
		@CompoundIndex(name="policyTypeIdx", def="{agent: 1, sourceSystem: 1, policyType: 1}")
		public class DomPolicyEntity extends AbstractPolicyEntity { 
			@Id
			private String id;
			...
		}
		
		public interface DomPolicyEntityDao extends MongoRepository<DomPolicyEntity, String> { }
		public interface MongoRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> { }
		public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID>  
		public interface CrudRepository<T, ID> extends Repository<T, ID>


## spring-batch

spring-boot-starter-batch.jar uses spring-batch.jar

Case Study: read from database (sql or stored procedure), write to a csv file.  
MRI sample - /business-process-jobs/src/main/resources/com/tdsecurities/stars/bpe/file-extract
  
  
## spring-integration (deprecated)

spring-integration is deprecated. The new tech is Apache Camel for service orchestration and data channel integration. 

MRI sample - /orchestrator/src/main/resources/META-INF/spring/bcbs-svlm.xml   
MRI sample - /orchestrator/src/main/java/com/tdsecurities/stars/orchestrator/bcbs/TimelinessServiceActivator.java   
	  
Case Study: read a csv file, convert each line to a JMS XmlMessage, send them to a middleware queue 	  

Reference Implementation Steps:	  

1. source is inbound-channel (OS physical file) -> inbound-channel-adapter (adapt from source model a OS file to java model a File object) -> input-channel(java File object) -> service activator (java.io.File) 

2. service activator: read java File object -> foreach line, conver a csv line to to a java model objects -> JAXB2 marshall each java model obj to a xml String) ->  output-channel.send(GenericMessage<String> xmlString)
 
3. output-channel (GenericMessage<String> xmlString) -> outbound-channel-adapter (adapt from java model a GenericMessage<String> to destination model a JMS XmlMessage), using JmsTemplace, send JMS messages to middleware Queue -> Destination is outbound-channel(JMS messages in middleware Queue)  

## JMS


## AOP  

AOP is a way for adding behavior to existing code without modifying that code. annotation driven, non-invasiveness.    

AOP solves cross-cutting concern.

a aspect = pointcuts + advisors, 

Two types of AOP programming:
- Spring-AOP 
- AspectJ AOP

Both spring-AOP lib and AspectJ lib defines: @Aspect, @Pointcut, @Around, @Before, @After, @AfterReturnning, @AfterThrowing annotations.  


### Spring-AOP (default)

Spring-AOP is one of the essential parts of the spring framework. the spring framework is based on IoC, DI and AOP. The AOP is one of the most important parts of the framework.

Spring-AOP creates proxies at spring context container loading time. It is Runtime weaving using proxy.

@enableAspectJAutoProxy on a @configuration class, OR, <aop:aspectj-autoproxy/> in xml config, it enables @AspectJ style of aspect declaration, but AspectJ runtime is not used. It still use Spring AOP proxies. 

Spring-AOP will build proxies for the beans, using a JDKDynamicProxy if your bean implements an interface, OR, using CGLIB dynamic proxy to subclass an implementation class  if your bean doesn't implement any interface.

@EnableAspectJAutoProxy(proxyTargetClass = true) will force Spring container to always use CGLIB style subclass proxy.

		@Interface, @Aspect @Component or META-INFO/aop.xml,   
		@Around/@Before/@After,  
		ProceedingJointPoint.proceed(),  
		@Pointcut("execution(public * foo..*.*(..))"), 
		@Pointcut("@annotation(\<annotation class name>)"),  
		@Pointcut("within(\<package> or \<interface> or \<annotation class name>)")

for every class annotated with @Aspect, a singleton aspect bean is created in Spring context container. Then spring will use the aspect beans to create the proxy instances for those target beans.

For Spring-AOP you do not need the AspectJ compiler, but then you are stuck with the proxy-based "AOP lite" approach which comes at the cost of internal calls not being intercepted by aspects because they do not go through proxies but through this (the original object).


### AspectJ 
 
libs: aspectj-maven-plugin, org.aspectj.aspectjtools.jar, org.aspectj.aspectjweaver.jar, org.aspectj.aspectjrt.jar, 

AspectJ modifies class bytecodes at compilation time. Compile time weaving can offer benefits of performance (in some cases).  

- Compile time weaving. It is done through AspectJ Tools(ajc compiler) if java source files are available
- OR LTW (Load Time Weaver). Which is post compilation weaving (using compiled class files). Spring Framework's support for LTW (Load Time Weaver). 

For full-blown AspectJ you can configure Spring to use LTW (load-time weaving) as described in manual chapter Using AspectJ with Spring applications. Alternatively, you can also use compile-time weaving, but this is not necessary unless you have performance problems during application start-up.


### Comparison

there are two major differences btwn Spring-AOP and AspectJ-AOP: 

- the type of weaving     
  Spring-aop approach is simpler and more manageable. But with the Spring AOP you can't use the all power of AOP because the implementation is done through proxies and not with modification of your bytecode.
  
  Spring AOP is basically proxy instances created by the Spring container, so spring-AOP can only be applied to Spring beans.   
  AspectJ AOP can be used to any java instances (beans or not-beans)

- joinpoint definition  
  1. spring-AOP can only be applied to Spring beans. AspectJ AOP can be used to any java instances (beans or not-beans).  
  2. the joinpoint definition in Spring-aop is restricted to method definition. While with AspectJ you can use the aspect in both methods and fields. 

- runtime performance
  If performance under high load is important, you'll want AspectJ which is 9-35x faster than Spring AOP. whether your aspects will be mission critical.




 
 
