---
layout: post
title:  "Java log frameworks | Java build tools"
date:   2018-10-01 13:15:42 -0500
categories: tech-java-spring
---

## Java log architecture

https://mp.weixin.qq.com/s/O2i-_q4l5J8V8cBQ_RCwYA

解决日志框架共存/冲突问题其实很简单，只要遵循几个原则：

统一使用一套日志实现
删除多余的无用日志依赖
如果有引用必须共存的话，那么就移除原始包，使用“over”类型的包（over类型的包复制了一份原始接口，重新实现）
不能over的，使用日志抽象提供的指定方式，例如jboss-logging中，可以通过org.jboss.logging.provider环境变量指定一个具体的日志框架实现
项目里统一了日志框架之后，无论用那种日志框架打印，最终还是走向我们中转/适配后的唯一一个日志框架。

![log architecture](/images/java/log-archer.png)


## Gradle

[gradle](https://mp.weixin.qq.com/s/A8kIeo-yjyVCs9EhYdLzwQ)