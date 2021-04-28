---
layout: post
title:  "Java-Spring Technical notes - 2020"
date:   2020-10-01 00:00:00 -0500
categories: tech-java-spring
---

# 2020 technical notes #

## spring-cloud

If you set spring.cloud.config.server property in bootstrap.xml, spring-boot app will start a embedded spring-cloud config service. No spring-cloud config client is needed in this case since spring-cloud config service is embeded in the app.

If there is no embedded spring-cloud config server is started in spring-boot app, then spring-cloud config client is needed to connect to remote spring-cloud config service.  

Embedded spring-cloud config service auto disables spring-cloud config client unless you set "spring.cloud.config.enabled=true" from a System Property (-D).  
"spring.cloud.config.enabled=true" will enable spring-cloud config client in the app.

"spring.cloud.kubernetes.enabled=true" will enable spring-cloud config kube client to connect to remote spring-cloud config service running on k8s cluster.


## Spring Cloud Context: Application Context Services

"bootstrap context" vs "application context"
 
https://cloud.spring.io/spring-cloud-commons/multi/multi__spring_cloud_context_application_context_services.html

"bootstrap context" is a parent context for the main "application context".  The two contexts share an Environment, which is the source of external properties for any Spring application.


## Guava Cache- java local cache

https://mp.weixin.qq.com/s/BV0Kcr7K3537kYIjuI740w


## Drools


