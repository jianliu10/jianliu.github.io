---
layout: post
title:  "Big data technical note - Kafka"
date:   2019-12-15 00:00:00 -0500
categories: tech-big-data
---

# Getting Started

## introduction

Kafka is a distributed publish-subscribe messaging system that maintains feeds of messages in partitioned and replicated topics. In the simplest way there are three players in the Kafka ecosystem: producers, topics (run by brokers) and consumers.

kafka documentation here:   
https://kafka.apache.org/documentation

## Kafka installation 

https://kafka.apache.org/documentation/#gettingStarted  
https://medium.com/@shaaslam/installing-apache-kafka-on-windows-495f6f2fd3c8

Setting up Apache Kafka:
1. Go to config folder in Apache Kafka and edit “server.properties” using any text editor.
2. Find log.dirs and repelace after “=/tmp/kafka-logs” to “=C:\\Tools\\kafka_2.10–0.10.1.1\\kafka-logs” (change your version number).
3. Leave other setting as is. If your Apache Zookeeper on different server then change the “zookeeper.connect” property.

By default Apache Kafka will run on port 9092 and Apache Zookeeper will run on port 2181.


## Kafka server

Open two console terminals to start zookeeper server and kafka server:

	bin\zookeeper-server-start.sh config/zookeeper.properties
	bin\kafka-server-start.bat .\config\server.properties

## topic

create a topic:

	bin/kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic numtest
	bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe
	
## console Producer & Consumer commands

open two console terminals, run producer.bat cna consumer.bat commands to test Kafka server and Topic:

	bin\kafka-console-producer.bat — broker-list localhost:9092 — topic sql-insert
	bin\kafka-console-consumer.bat -bootstrap-server localhost:2181 -topic sql-insert
	
Eenter messages from the producer’s terminal and see them appearing in the subscribed consumer’s terminal.
	
# Configs

		3.1 Broker Configs
			3.1.1 Updating Broker Configs
		3.2 Topic-Level Configs
		3.3 Producer Configs
		3.4 Consumer Configs
		3.5 Kafka Connect Configs
			3.5.1 Source Connector Configs
			3.5.2 Sink Connector Configs
		3.6 Kafka Streams Configs
		3.7 Admin Configs	
	
# APIs

Kafka exposes all its functionality over a language independent protocol which has clients available in many programming languages. However only the Java clients are maintained as part of the main Kafka project, the others are available as independent open source projects. 

## admin api (in kafka-clients.jar)


The Admin API supports managing and inspecting topics, brokers, acls, and other Kafka objects.
	
## Kafka procuder api (in kafka-clients.jar)


## kafka consumer api (in kafka-clients.jar)


## kafka Connect api	

For many systems, instead of writing custom integration code you can use Kafka Connect to import or export data.

The Connect API allows implementing connectors that continually pull from source data system into Kafka, or push from Kafka into sink data system. 

A KafkaConnect is an application, which contains one or more connectors.

Many users of Connect won't need to use this API directly, though, they can use pre-built connectors without needing to write any code.

The connectors included in kafka-clients jar:
- FileSystemSource, FileSystemSink
- database
- SOAP ws
- REST ws

## kafka Streams api (in kafka-streams.jar)

https://kafka.apache.org/24/documentation/streams/

Kafka Streams is a client library for building real-time applications and microservices, where the input and output data are stored in Kafka clusters. It combines the simplicity of writing and deploying standard Java and Scala applications on the client side with the benefits of Kafka's server-side cluster technology.

A KafkaStreams is an application, which include the stream data transformations.

Functions:
- Producer / Consumer Applications are programmed with "Kafka Streams API". Kafka Streams API lib interact with Kafka server.
- Kafka Streams API transforms and enriches data
- Write standard Java applications and microservices to process the stream data in real time. 
	- No seperate processing cluster is required.  
	- deployed to container, VM, bare metal, cloud, on prem.
	- elastic, scalable, fault tolerant.
	- support Exactly-Once sematics.

### KStream and KTable

leverage the duality between a table and a changelog stream (here: table = the KTable, changelog stream = the downstream KStream): you can publish every change of the table to a stream, and if you consume the entire changelog stream from beginning to end, you can reconstruct the contents of the table.

ktable: is a table. it support upsert operations (insert new record, update existing record)

ktable.toStream(): convert upsert changes of a ktable to a kstream.

### Transforming data 

- kstream -> kstream :   
	kstream.filter, map, flatmap, flatmapvalues, join, leftJoing, rightJoin
- kstream -> kgroupedtable :   
	kstream.groupby
- kgroupedtable -> ktable  
	  count("state-store-name"), aggregate(...)
	  /* windowing by event-time */ count(TimeWindows.of(TimeUnit), "state-store-name")    
- kstream -> ktable
	kstream.reduce
- ktable -> ktable : 
	ktable.filter, mapValue, join
- ktable -> kstream change logs: 
	ktable.toStream()

e.g.

	kstream<songId, playevent> stream2 = kstream<userId, playevent>.filter((u,p) -> p.duration > 2sec).map((u,p) -> new spring.util.Pair(p.getSongId(), p));
	
	kstream<songId, song> stream3 = stream2.leftJoin(songTable, (playevent, song) -> song, SerDes.long(), songSerde);

	
## Examples

### Creating a Streams Application in Java using Kafka Stream API

	import org.apache.kafka.common.serialization.Serdes;
	import org.apache.kafka.common.utils.Bytes;
	import org.apache.kafka.streams.KafkaStreams;
	import org.apache.kafka.streams.StreamsBuilder;
	import org.apache.kafka.streams.StreamsConfig;
	import org.apache.kafka.streams.kstream.KStream;
	import org.apache.kafka.streams.kstream.KTable;
	import org.apache.kafka.streams.kstream.Materialized;
	import org.apache.kafka.streams.kstream.Produced;
	import org.apache.kafka.streams.state.KeyValueStore;

- StreamsConfig class

		Properties props = new Properties();
		props.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-application");
		props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka-broker1:9092");
		
- StreamBuilder class. builder.stream("TextLinesTopic")

		StreamsBuilder builder = new StreamsBuilder();
		KStream<String, String> textLines = builder.stream("TextLinesTopic", Consumed.with(stringSerde, stringSerde));
		KTable<String, Long> wordCounts = textLines
			.flatMapValues(textLine -> Arrays.asList(textLine.toLowerCase().split("\\W+")))
			.groupBy((key, word) -> word)
			.count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as("counts-store"));
		wordCounts.toStream().to("WordsWithCountsTopic", Produced.with(Serdes.String(), Serdes.Long()));
		
- Serdes (serializers and deserializers) for key and value

- KStreamBuilder.stream(keySerde, valueSerde, topicName) returns a KStream<k, v>. A stream contains change logs for keys

- KStreamBuilder.table(keySerde, valueSerde, topicName) returna s KTable<k, v>. A table contains the current records for keys

- KafkaStreams class. 

		KafkaStreams streams = = new KafkaStreams(builder.build, props); 
		streams.start();
	
- state data store (materialize)	

		kgroupedtable.count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as("counts-store"));

- starts the WordCount demo application:
	
		bin/kafka-run-class.sh org.apache.kafka.streams.examples.wordcount.WordCountDemo	
	
As one can see, outputs of the Wordcount application is actually a continuous stream of records upserts.

	
### Creating a Streams Application in Python using Kafka Producer API and Consumer API

https://towardsdatascience.com/kafka-python-explained-in-10-lines-of-code-800e3e07dad1

- install kafka client module:

		pip install kafka-python
		conda install -c conda-forge kafka-python

- producer app:

		from time import sleep
		from json import dumps
		from kafka import KafkaProducer

		producer = KafkaProducer(bootstrap_servers=['localhost:9092'],
								 value_serializer=lambda x: 
								 dumps(x).encode('utf-8'))
		for e in range(1000):
			data = {'number' : e}
			producer.send('numtest', value=data)
			sleep(5)						 

- comsumer app:

		from kafka import KafkaConsumer
		from pymongo import MongoClient
		from json import loads

		consumer = KafkaConsumer(
			'numtest',
			 bootstrap_servers=['localhost:9092'],
			 auto_offset_reset='earliest',
			 enable_auto_commit=True,
			 group_id='my-group',
			 value_deserializer=lambda x: loads(x.decode('utf-8')))
			 
		client = MongoClient('localhost:27017')
		collection = client.numtest.numtest	 

		for message in consumer:
			message = message.value
			collection.insert_one(message)
			print('{} added to {}'.format(message, collection))

		
	
	




