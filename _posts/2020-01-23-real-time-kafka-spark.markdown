---
layout: post
title:  "How to use Kafka for ETL"
date:   2020-01-22 04:28:53 +0800
categories: python kafka 
tags: python, data integration
image: kafka.png
applause: true
short_description: A guide for how to use kafka for real time data processing with apache spark stream. 
--- 

<div markdown="1" id="text">
I choose to use Apache Kafka, one of the reasons is it provides API for user-friendly Python which easy to integrate to many other tools via Kafka Connect, also it's so sophisticated for big data integration. I had confusion when I tested with both database connector through Python code and Kafka Connect. The Kafka Connect one is hard for the first time setting up and requires strict schema registry, otherwise it broke the 'contract' between schema reads and writes that Producer and Consumer requests. Though Python DB Connect and Kafka Connect both work, but I have one friend, who taught me few big data cases last year suggested me, <strong>Kafka Connect is industrial usage,</strong> which can make life easier. So I will demonstrate both approaches for streaming integration.

- Connect with SQL db 
- Use Kafka Connect (Debezium CDC and JDBC connector for data ingestion)

Besides since KSQL was released on last November. I haven't heard it before and it's more for another topic on Kafka processing layer, same as Kafka Stream, so I exclused the details and implementations on this exploratory article. 
<!--more--> 

<strong>In general, the key to solve the problem is to frame it correctly, same to learning a new tool, is to understand the diagram of this tool well. </strong><br/>

So what is Kafka? 
there are tons of articles of high quality on Confluent <a href='https://www.confluent.io/blog'>blog</a> and Kafka official <a href='https://kafka.apache.org/uses'>website</a> and its <a href='https://kafka.apache.org/documentation/'>documentation</a>, so I won't explain here, for as a programmer one key skillset is to know how to read source and documentation. For anything you can learn is available on the Internet now, the doorway is Google. Let me just highlight key components that on this diagram, then we get our hands dirty. <br/>
<hr/>
<ul>
<li>Kafka Broker, Topic, Message, Partition, Producer, Consumer, Offset.</li>
<li>Zookeeper use for consumer leader selection.</li>
<li>Avro, use this schema to communicate through Kafka, producer encode the schema and consumer decode, Kafka provides REST API to update schema.</li>
<li>Kafka Connect (pulled from Debezium), is a framework for ingesting data with Kafka, used as source and sink.</li>
</ul>
<hr/>

<strong>Some basics need to understand </strong>
<ul>
<li>Log compaction, well-covered by Jay Kreps <a href='https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying'>link</a> <strong><i>spoiler alert: if you want find the truth, go for the log.</i></strong></li>
<li>Throughput, default mode is asynchronous/non-blocking writes increase number of partitions to scale consumers.</li>
<li>Exactly once semantic, here is one article <a href='https://medium.com/@andy.bryant/processing-guarantees-in-kafka-12dd2e30be0e'>link</a> wrote about the processing guarantee in Kafka by comparing this semantic with no guarantee, most once, effectively once.</li>
<li>Data retention policy to decide how long the data can be stored in broker.</li>
<li>CDC(change data capture), monitor the changes (insert, update and delete). Similar to RDBMS's trigger.</li>
</ul> 

<!-- Event is the log, which is the topic contains. Topic is conceptually similar to the table in database on storage layer, the producer and consumer write and read data from this layer, just like database. Data can be stored as long as needed up to data retention policy. On this concept, Kafka exactingly can be used as the database.  -->

<!-- Basic component: 
- Topic is made of messages, which is a key/value "bytes" together with the header and timestamp as a tuple, key is formated as string. 
- Consumer and Producer writes and reads: The relationship of producer and consumer is the classic one-to-many. Producer produces data and multiple consumers can read it from different offsets or seek backwards to old timestamps. It has ability to provide reliable, self-balancing, and scalable ingestion buffer. 
- Partitioning data: The topic in order to scale for comsuming, which be split into the partitions, each partition lives in each own broker. The whole principle of partition, is the log, which is immutable. 
- Groups for scalability: Consumers are titled with groupId as one Group, the same groupId can be used to consume different topics and are totally independent. -->

<!-- The typical use-case that uses Kafka to ingest, transform, load, validate, enrich and store the real time data into the DWH or data lake. The reason is Kafka supports high throughtput writes and low lantency reads, and maintainable. -->
<figure>
<img style='width:400px height:400px;' alt='data pipeline' src='https://engineeringblog.yelp.com/images/posts/2016-07-14-billions-of-messages-a-day-yelps-real-time-data-pipeline/4.jpg'/> 
<figcaption style='margin-left: 160px;'><small style='color:gray;'>the data pipeline from Yelp's website, well illustarted the whole pipeline</small></figcaption>
</figure> 

<!-- The usage of the partiotion: 
why event in the same event key may end up in different partitions?
- Topic configuration (partitioning function)
- Producer configuration 
for the partition, over partitioning is suggested. 30 partitions per topic. 

if each partition's topic has different sizes, how to write partitioning function f(event.key, event.value) to make the processing effectively?  -->
<!-- <figure>
<img style='width:400px height:400px;' src='https://cdn.confluent.io/wp-content/uploads/partitions2.png'/> 
<figcaption style='margin-left: 200px;'><small style='color: gray'>how the message used with partitions</small>
</figcaption>
</figure>  -->
<!-- Before starting to step by step set up and write producer, consumer, pay attention three main functionalities Kafka provides, 

- pub/sub to events(message bus)
- store events for as long as you want (storing)
- process and analyze events(processing)  -->

<!-- <img style='width:400px height:400px' src='https://miro.medium.com/max/1924/1*kQXkMQTrMrG4VJ3KZehaqA.png'/>  -->

<!-- Kafka Connect: Using Kafka Connect, you can use existing connector implementations for common data sources (Source Connector) and sink (Sink Connector) to move data into and out of Kafka. Kafka Connect is focused on streaming data to and from Kafka, making it simpler for you to write high quality, reliable, and high-performance connector plugins. Kafka Connect is an integral component of an ETL pipeline when combined with Kafka and a stream processing framework. Kafka Connect can run either as a standalone process for running jobs on a single machine (e.g., log collection), or as a distributed, scalable, fault tolerant service supporting an entire organisation.

The JDBC source connector allows you to import data from any relational database with a JDBC driver into Apache Kafka topics. By using JDBC, this connector can support a wide variety of databases without requiring custom code for each one. Data is loaded by periodically executing a SQL query and creating an output record for each row in the result set. The database is monitored for new or deleted tables and adapts automatically. When copying data from a table, the connector can load only new or modified rows by specifying which columns should be used to detect new or modified data. We will use this to connect with MySQL database.  -->

Additions: <br/>
The usage with Redis Cache: for data that is frequently needed or retrieved by app users, a cache would serve as a temporary data store for quick and fast retrieval without the need for extra database round trips. Note that data stored in a cache is usually data from an earlier query or copy of data stored somewhere else. This feature is vital because the more data we can fetch from a cache, the faster and more efficiently the system performs overall. 

##### Hand dirty time: 
#### Set up 
```bash
#Install Zookeeper 
$ brew install zookeeper 
$ brew services start zookeeper 

or 
$ bin/zookeeper-server-start.sh config/zookeeper.properties 
$ kafka-server-start /usr/local/etc/kafka/server.properties 
# kafka default is running on the port 9092, the zookeeper is on 2181 
$ config/server.properties 

#set up multi-broker cluster 
$ cp config/server.properties config/server-1.properties
$ cp config/server.properties config/server-2.properties

# edit on server-1.properties, for eaxmaple
config/server-1.properties:
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dirs=/tmp/kafka-logs-1
```
Next step is install Kafka, download from its official website. 

Set up multiple partitions 
```
$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor <number-of-replicas> --partitions <number-of-partitions> --topic <topicName>
$ kafka-topics --zookeeper <host>:2181 --create --topic <topicName> --partitions <number-of-partitions> --replication-factor <number-of-replicas>(old way)
```
Add partitions 
```
$ bin/kafka-topics.sh --alter --zookeeper localhost:2181 --topic <topicName> --partitions 3
```
Create topic
```
$ kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions <1> --topic <topicName>
```
Check existed topics
```
$ bin/kafka-topics.sh  --zookeeper localhost:2181 --list
```
Describe a topic 
```
$ bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic <topicName>
$ bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic <topicName>(old way)
```
Check consumer group information
```
$ kafka-consumer-groups  --bootstrap-server localhost:9092 --list
$ kafka-consumer-groups  --bootstrap-server localhost:9092 --describe --group <groupName>
```
Delete topic, make sure topic default setting is delete.topic.enable= True
``` 
$ bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic <topicName>
```
Config retention times

```
$ bin/kafka-configs.sh --zookeeper localhost:2181 --alter --entity-type topics --entity-name test_topic --add-config retention.ms=1000
$ bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic mytopic --config retention.ms=1000 (old way)
```

Publish message to Kafka
```
$ cd kafka_2.11-1.1.0 
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic <topicName>
```

Consume message 
```
$ kafka-console-consumer --bootstrap-server localhost:9092 --topic <topicName> --from-beginning
```

Get consumer offset for Topic
```
$ bin/kafka-consumer-offset-checker.sh --zookeeper=localhost:2181 --topic=<topicName> --group=<groupName>
```

Install avro, understand how to use Python and REST 

```bash 
$ pip install confluent-kafka[avro]
$ kafka-configs --zookeeper localhost:2181 --entity-type topics --entity-name _schemas --describe Configs for topic '_schemas' are cleanup.policy=compact 

# SCHEMA_REGISTRY_URL
# subject-value
# schemaId
```
<!-- #### Prototype (design)  -->
#### Implementation
##### Produce message  
- config broker, ports, zookeeper connection, default topic, producer timeout, encoder, partition, retentation time, etc
- write avro schema with REST 
- implement with python code 

```bash 
# how to know the consumer lag? consumer_offsets (group, topic, partition number)
kafka-console-consumer --consumer.config /tmp/consumer.config \
  --formatter "kafka.coordinator.GroupMetadataManager\$OffsetsMessageFormatter" \
  --zookeeper localhost:2181 \
  --topic __consumer_offsets
```
##### Read message 
- set offset, consumer timeout, etc 
- implement with python code 
#### Connect data to RDBMS
- MysqlDB 
- Kafka Connect 

##### Advanced Kafka Development:
Create multi-thread consumer 
specifying offsets
consumer rebalancing 
partitioning data 
message durability 

#### Source  
<a href='https://blog.softwaremill.com/7-mistakes-when-using-apache-kafka-44358cd9cd6'>7 mistakes when using Apache Kafka</a>
</div>