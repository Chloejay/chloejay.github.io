---
layout: post
title:  "How to use kafka with Spark for the ETL"
date:   2020-01-22 04:28:53 +0800
categories: python kafka 
tags: python 
applause: true
short_description: A guide for how to use kafka for real time data processing with apache spark stream. 
--- 

<div markdown="1" id="text">
draft! 
<hr>

I choose to use Apache Kafka, I think one reason is that it provides API for user-friendly Python and also it's easy to integrate to many other tools via Kafka Connect. I had confusion when I tested with both use database connector through the code and Kafka Connect, they both worked but I have one mentor told me, Kafka Connect is more industrial usage, at least which can make my life easier.<br/> So on this article I will demonstrate both approaches. 
- Connect with the SQL db 
- Use Kafka Connect 

So what is the Kafka? <br/>
Kafka as the cluster of broker, which connected with the low-level producer and consumer APIs, it well known as a general distributed data stream processing platform. It can be used as a horizontally scable and high performance data ingestion system in the data engineering field. <br/> 
<!--more-->  

The typical use case that use kafka is to read, transform, load, validate, enrich and store the real time data into the DWH. The reason is Kafka supports high throughtput writes and low lantency reads. 

The data model in kafka is message, a key/value pair, key can be the string. Topic are the named space that let producer and consumer write and read the message. It has ability to provide reliable, self-balancing, and scalable ingestion buffer. The topic in order to scale for comsumptition, which be split into the partitions, each partition lives in each own broker. The whole principle of partition, is the log, which is immutable. Producer produce the data and multiple consumers can read it from different offsets or seek backwards to old timestamps. 
<img src='https://miro.medium.com/max/1924/1*kQXkMQTrMrG4VJ3KZehaqA.png'> 

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