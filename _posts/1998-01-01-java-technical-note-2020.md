---
layout: post
title:  "Java-Spring Technical notes - 2020"
date:   2020-10-01 00:00:00 -0500
categories: tech-java-spring
---

# 2020 technical notes #

## spring-cloud

If you set spring.cloud.config.server property in bootstrap.xml, spring-boot app will start a embedded spring-cloud service. No spring-cloud client is needed in this case since spring-cloud service is embeded in the app.

If there is no embedded spring-cloud config server is started in spring-boot app, then spring-cloud client is needed to connect to remote sprong-cloud config service.  

Embeded spring-cloud config service disables spring-cloud config client explicitly unless you set "spring.cloud.config.enabled=true" from a System Property (-D).  

"spring.cloud.kubernetes.enabled: true" will enable spring-cloud kube client to connect to remote spring-cloud config service running on kube cluster.


## junit5

JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

Adding Jupiter (for JUnit 5 tests only) and Vintage (for Junit4/Junit3 compatibility - to run legacy JUnit4 tests from JUnit5) to pom.xml is like this (just for future reference):

JUnit Jupiter is the combination of the new programming model and extension model for writing tests and extensions in JUnit 5. The Jupiter sub-project provides a TestEngine for running Jupiter based tests on the platform.

JUnit Vintage provides a TestEngine for running JUnit 3 and JUnit 4 based tests on the platform.

<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.4.0</version>
    <scope>test</scope>
</dependency>

<!-- Vintage Module to run JUnit4 from JUnit 5 -->
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>5.4.0</version>
    <scope>test</scope>
</dependency>

### migrate Junit4 to Junit5

If we want to migrate this test to JUnit5 we need to replace the @RunWith annotation with the new @ExtendWith:
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = { SpringTestConfiguration.class })
OR
@SpringJUnitConfig({ServiceConfig.class, DaoConfig.class, SpringTestConfiguration})



 
