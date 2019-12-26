---
layout: post
title:  "Event Messaging and Data Streaming in distributed system"
date:   2019-12-16 00:00:00 -0500
categories: tech-architecture-design
---

# Event Messaging and Data Streaming in distributed system

## Event Messaging

the message payload does NOT contain the source data. Its payload contains the physical location of the source data.

#### AWS SNS topic, SQS queue

use case: Finra FileX project, HERD project.


## Data Streaming

the message payload contains the source data.

#### Solace ESB 

use case: TD stars Midas project

#### Kafka 

#### AWS Kienes

use case: Finra logs delivery stream to DynamoDB no-sql database for Splunk system.