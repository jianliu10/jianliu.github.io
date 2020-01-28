---
layout: post
title:  "Event Messaging vs Data Streaming in distributed system"
date:   2019-12-16 00:00:00 -0500
categories: tech-architecture-design
---

# Event Messaging and Data Streaming in distributed system

## Event Messaging

the message payload does NOT contain the source data. Its payload contains the metadata of the source data, including souece data physical location, size, creation date, originating application, event type, etc.

- AWS SNS topic, SQS queue. use case: Finra FileX project, HERD project.

- ActiveMQ messaging middleware. use case: TDS MRI project.


## Data Streaming

the message payload contains the source data.

Streaming data platforms include: Solace, Kafka, AWS Kienes.

- Solace. enterprise service bus. use case: TD stars Midas project

- Kafka. Streaming data platform

- AWS Kienes. AWS streaming data platform. use case: Finra logs delivery stream to DynamoDB no-sql database for Splunk system.