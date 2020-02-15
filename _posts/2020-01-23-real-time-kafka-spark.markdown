---
layout: post
title:  "How to use Kafka with Spark for ETL"
date:   2020-01-22 04:28:53 +0800
categories: python kafka 
tags: python 
image: kafka.png
applause: true
short_description: A guide for how to use kafka for real time data processing with apache spark stream. 
--- 

<div markdown="1" id="text">
<span style='color:red' >draft!</span>
<br/> 
I choose to use Apache Kafka, one of the reasons is that it provides API for user-friendly Python and it's easy to integrate to many other tools via Kafka Connect. I had confusion when I tested with both using database connector through Python code and Kafka Connect, the second one is hard for the first setting up and required the strict schema registry, otherwise it failed when consuming real-time sample data. Though they both worked but I have one mentor who taught me few big data cases suggested me, Kafka Connect is more industrial usage, at least which can make my life easier.<br/> So on this article I will demonstrate both approaches for streaming integration.

- Connect with SQL db 
- Use Kafka Connect(Debezium CDC and JDBC connector) 

Besides since the KSQL was released on last November. It's been a new use case that I have read from latest tech blogs. I was considering if I need to add KSQL too on this article based on time I spend for learning and practicing Kafka. But I think it's more for another topic on Kafka processing layer, so I exclused the details and implementations on this article. 
<!--more--> 

So what is Kafka? <br/>
Kafka as the cluster of broker, which connected with the low-level producer and consumer APIs, it well known as a general distribut event stream processing platform, a core part for the ETL as a message bus.

Event is the log, which is the topic contains. Topic is conceptually similar to the table in database on stoarage layer, the producer and consumer write and read data from this layer, just like database. Data can be stored as long as needed up to data retention policy. 

Topic is made of messages, which is a key/value "bytes" together with the header and timestamp as a tuple, key is formated as string. The relationship of producer and consumer is the classic one-to-many. Producer produces data and multiple consumers can read it from different offsets or seek backwards to old timestamps. It has ability to provide reliable, self-balancing, and scalable ingestion buffer. The topic in order to scale for comsuming, which be split into the partitions, each partition lives in each own broker. The whole principle of partition, is the log, which is immutable. Consumer can group to group, titled with groupId, the same groupId can be used to consume different topics and are totally independent.

The typical use-case that uses Kafka to ingest, transform, load, validate, enrich and store the real time data into the DWH or data lake. The reason is Kafka supports high throughtput writes and low lantency reads, and maintainable.
<figure>
<img style='width:400px height:400px;' alt='data pipeline' src='https://engineeringblog.yelp.com/images/posts/2016-07-14-billions-of-messages-a-day-yelps-real-time-data-pipeline/4.jpg'/> 
<figcaption style='margin-left: 160px;'><small style='color:gray;'>the data pipeline from Yelp's website, well illustarted the whole pipeline</small></figcaption>
</figure> 

The usage of the partiotion: 
why event in the same event key may end up in different partitions?
- Topic configuration (partitioning function)
- Producer configuration 
for the partition, over partitioning is suggested. 30 partitions per topic. 

if each partition's topic has different sizes, how to write partitioning function f(event.key, event.value) to make the processing effectively? 
<figure>
<img style='width:400px height:400px;' src='https://cdn.confluent.io/wp-content/uploads/partitions2.png'/> 
<figcaption style='margin-left: 200px;'><small style='color: gray'>how the message used with partitions</small>
</figcaption>
</figure> 

Kafka's three key functionalities in a scalable, fault-tolerant, and reliable manner:
- pub/sub to events(message bus)
- store events for as long as you want (storing)
- process and analyze events(processing) 

Advantages: 
- Data Centric Pipeline — use meaningful data abstractions to pull or push data to Kafka.
- Flexibility and Scalability — run with streaming and batch-oriented systems on a single node or scaled to an organization-wide service.
- Reusability and Extensibility — leverage existing connectors or extend them to tailor to your needs and lower time to production. 

<img style='width:400px height:400px' src='https://miro.medium.com/max/1924/1*kQXkMQTrMrG4VJ3KZehaqA.png'/> 

Kafka Connect: is a framework for connecting Kafka with external systems such as databases, key-value stores, search indexes, and file systems. Using Kafka Connect, you can use existing connector implementations for common data sources and sinks to move data into and out of Kafka. Kafka Connect is focused on streaming data to and from Kafka, making it simpler for you to write high quality, reliable, and high-performance connector plugins. Kafka Connect is an integral component of an E-T-L pipeline when combined with Kafka and a stream processing framework. Kafka Connect can run either as a standalone process for running jobs on a single machine (e.g., log collection), or as a distributed, scalable, fault tolerant service supporting an entire organisation. This allows it to scale down to development, testing, and small production deployments with a low barrier to entry and low operational overhead, and to scale up to support a large organisation’s data pipeline.

Source Connector<br/>
A source connector ingests entire databases and streams table updates to Kafka topics. It can also collect metrics from all your application servers into Kafka topics, making the data available for stream processing with low latency. 

Sink Connector<br/>
A sink connector delivers data from Kafka topics into secondary indexes such as Elastic search or batch systems such as Hadoop for offline analysis.

The JDBC source connector allows you to import data from any relational database with a JDBC driver into Apache Kafka topics. By using JDBC, this connector can support a wide variety of databases without requiring custom code for each one. Data is loaded by periodically executing a SQL query and creating an output record for each row in the result set. The database is monitored for new or deleted tables and adapts automatically. When copying data from a table, the connector can load only new or modified rows by specifying which columns should be used to detect new or modified data.

The usage with Redis Cache: for data that is frequently needed or retrieved by app users, a cache would serve as a temporary data store for quick and fast retrieval without the need for extra database round trips. Note that data stored in a cache is usually data from an earlier query or copy of data stored somewhere else. This feature is vital because the more data we can fetch from a cache, the faster and more efficiently the system performs overall.

#### Related Concept 
- Log compaction, well-covered by Jay Kreps <a href='https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying'>[link]</a> spolier: if you want find the truth, go for the log. 
- Zookeeper, Kafka’s sidekick used for managing consumers 
- Throughput, the default mode is asynchronous/non-blocking writes 
increase number of partitions to scale the consumers. 
- Data retention policy to decide how long the data can be stored in broker.
- Exactly once semantic, here has one good article <a href='https://medium.com/@andy.bryant/processing-guarantees-in-kafka-12dd2e30be0e'>[link]</a> wrote about the processing guarantee in Kafka by comparing this semantic with no guarantee, most once, effectively once. 
- Schema registry: <a href='https://avro.apache.org/'>AVRO</a> format, to decoupled data formats, for all messages are 'just bytes' that send to Kafka, so need the schema store that stored the pre defined the schema to encode the communication between producer(write) and consumer(read). 
- Kafka Connect (pulled from Debezium), which can used as both source and sink. 
- CDC(change data capture), monitor the changes (insert, update and delete). 

#### Source  
recently read one post 7 mistakes when using Apache Kafka <a href='https://blog.softwaremill.com/7-mistakes-when-using-apache-kafka-44358cd9cd6'>[link]</a>

#### Set up 
#### Prototype (design) 
#### Implementation 
##### Produce message  
- config broker, ports, zookeeper connection, default topic, producer timeout, encoder, partition, retentation time, etc
- write avro schema 
- implement with python code 
##### Read message 
- set offset, consumer timeout, etc 
- implement with python code 
#### Connect data to DWH 
- MysqlDB 
- Kafka Connect 
#### Way more, to do the data processing with the Spark Stream 
</div>

know how our consumer application behaves
--group mygroup \
  --new-consumer \
  --describe

Note: some offsets are unknown (therefore the lag also) because the consumers did not consume all the partitions yet.

watch -n1 -t (use watch) 

how to know the consumer lag? 

kafka-console-consumer --consumer.config /tmp/consumer.config \
  --formatter "kafka.coordinator.GroupMetadataManager\$OffsetsMessageFormatter" \
  --zookeeper localhost:2181 \
  --topic __consumer_offsets

__consumer_offsets (group, topic, partition number)
