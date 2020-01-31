---
layout: post
title:  "How to use kafka with Spark for ETL"
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
I choose to use Apache Kafka, I think one reason is that it provides API for user-friendly Python and also it's easy to integrate to many other tools via Kafka Connect. I had confusion when I tested with both use database connector through the code and Kafka Connect, they both worked but I have one mentor told me, Kafka Connect is more industrial usage, at least which can make my life easier.<br/> So on this article I will demonstrate both approaches. 
- Connect with the SQL db 
- Use Kafka Connect 

Besides since the KSQL was released on November, last year. It's been new use case that I have read from some tech blogs too. I was considering if I need to add KSQL too on this article for the time I spend for learning and practicing Kafka, but I think it's more for another topic on the Kafka processing layer, so I exclused the details and implementation on this article. 
<!--more--> 
So what is the Kafka? <br/>
Kafka as the cluster of broker, which connected with the low-level producer and consumer APIs, it well known as a general distribut event stream processing platform. Event is a key/value tuple including the timestamp. It can be used as a horizontally scable and high performance data ingestion system in the data engineering field. <br/> 

The typical use case that use kafka is to read, transform, load, validate, enrich and store the real time data into the DWH. The reason is Kafka supports high throughtput writes and low lantency reads.  

The data model in kafka is message, a key/value pair, key can be the string. Topic are the named space(kafka storage layer, part of the kafka 'filesystem' powered by the brokers) that let producer and consumer write and read the message, which can be stored as long as the project expected. Conceptually the topic is an unbounded sequence of serialized events and partitioned in different brokers. Producer produce the data and multiple consumers can read it from different offsets or seek backwards to old timestamps.
It has ability to provide reliable, self-balancing, and scalable ingestion buffer. The topic in order to scale for comsumptition, which be split into the partitions, each partition lives in each own broker. The whole principle of partition, is the log, which is immutable. <img src='https://cdn.confluent.io/wp-content/uploads/partitions2.png'> 
if each partition's topic has different sizes, how to well write the partitioning function f(event.key, event.value) to make the processing effectively? 

Kafka's three key functionalities in a scalable, fault-tolerant, and reliable manner:
- publish and subscribe to events
- store events for as long as you want
- process and analyze events

why event in the same event key may end up in different partitions?
- Topic configuration (partitioning function)
- Producer configuration 
for the partition, over partitioning is suggested. 30 partitions per topic. 

Kafka has two main functionalities, storing to processing, the storing happens in the kafka cluster and the processing is the kafka stream and ksql etc. 

data retentation policy 
<img src='https://miro.medium.com/max/1924/1*kQXkMQTrMrG4VJ3KZehaqA.png'> 

The usage with Redis Cache 
For data that is frequently needed or retrieved by app users, a cache would serve as a temporary data store for quick and fast retrieval without the need for extra database round trips. Note that data stored in a cache is usually data from an earlier query or copy of data stored somewhere else. This feature is vital because the more data we can fetch from a cache, the faster and more efficiently the system performs overall.

#### Related Concept 
- Zookeeper, Kafka’s sidekick used for managing consumers 
- throughput, the default mode is asynchronous/non-blocking writes 
increase number of partitions to scale the consumers. 
- data retention policy to decide how long the data can be stored in broker. 
- a distributed commit log, processing event and store the event, intergrate with the legacy relational database which is not the envent system. 
- exactly once semantic 
- Kafka’s schema registry: AVRO format
- Kafka Connect (pulled from Debezium), which will source and sink data back and forth to/from Postgres through Kafka

#### Source  
recently read one post 7 mistakes when using Apache Kafka <a href='https://blog.softwaremill.com/7-mistakes-when-using-apache-kafka-44358cd9cd6'>[link]</a>

#### Set up 
#### Produce message  
#### Read message 
#### Connect the data to the DWH 
#### Way more, to do the data processing with the Spark Stream 
</div>