---
layout: post
title:  "Apache Kafka Developers training- module 1"
date:   2020-09-23 18:00 +0800
categories: kafka, distributed system
tags: 
image: kafka_producer.png
applause: true
short_description: this is the first module of Confluent Kafka training.
--- 

<div markdown="1" id="text">
I'm preparing the Kafka Developer Certification, so taking Confluent training <a href="https://assets.confluent.io/m/a47e1a7a6ecd2fb/original/20200403_DS_Confluent-Developer-Skills-for-Apache-Kafka.pdf">Confluent Developer Skills for Building Apache Kafka</a>. To me, I write notes down for they’re a useful resource, so if when I want to refresh my memory on Kafka topic that I learnt, I can go here to review. In the meanwhile, it gives me a motivation to record this developer training.

This training consists of 12 modules, so this post is notes of the first module I started for this training.

- [Introduction](#Introduction)
- Fundamentals of Apache Kafka 
    - [2.1 Fundamentals of Apache Kafka - 1 of 2](#fundamentals-of-apache-kafka---1-of-2)
    - [2.2 Fundamentals of Apache Kafka - 2 of 2](#fundamentals-of-apache-kafka---2-of-2)
    - 2.3 Lab - Using Kafka’s Command-Line Tools

## 1. Introduction

Course Intro (skipped)<br/>
General training outlines:
<ul>
<li>- Write <code>Producers</code> and <code>Consumers</code> to send data to and read data from Kafka</li>
<li>- Integrate Kafka with external systems using <code>Kafka Connect</code></li>
<li>- Write streaming applications with <code>Kafka Streams & ksqlDB</code></li>
<li>- Integrate a Kafka client application with <code>Confluent Cloud</code></li>
</ul>

<hr/>

## 2. Fundamentals of Apache Kafka 
#### 2.1 Fundamentals of Apache Kafka - 1 of 2
Kafka, a Distributed Streaming Platform, is a data architecture for data distributing, storage and processing. It is used for building real-time data pipelines and streaming apps, its main characteristics are horizontally scalable, fault-tolerant, wicked fast in production.

#### Kafka Client (API)
From its built-in API, we can directly use high level code to `send()` and `subscribe()` data. Producer to write data to one or many topics,
Consumer (Groups) to subscribe data. <br/>
<small>(*Kafka cluster: cluster of brokers, store events.)</small>
<img src="/assets/kafka_cluster.png" height= "400" width= "800">

#### Data Structure => Log 
The immutable, append only, and once written, they are never changed data structure. Elements that are added to the log are strictly ordered in time. 

#### Policy 
They have two ways to retain messages, the default is **retention policy**, another one is by key, named **compact policy**. 
```
- retention policy 
- compact policy
#For example, if use case that is mutable state data, data can be modified, deleted or updated,
#Kafka only need the most recent messages, then `compact policy` is an option. 
```
<small>*Log compaction is a granular retention mechanism that retains the last update for each key. A log compacted topic log contains a full snapshot of final record values for every record key not just the recently changed keys. </small>


#### *Brokers, Topics, Partitions, and Segments
<img src="/assets/architecture_kafka_topics.png" height= "400" width= "800">
<ul>
    <li><strong>- Broker</strong>: think it as VM, which provides you memory, storage etc.</li>
    <li><strong>- Topic</strong>: comprises all messages to a given category. </li>
    <li><strong>- Partition</strong>: split a single topic into many partitions to parallelize work. The default algorithm for partition is <code>hash code of the message key</code>.</li>
    <li><strong>- Segment</strong>: a single log file, or commit log. </li>
    <li><strong>- Replica</strong>: *replica of each partition is always store in different brokers, to produces high availability of data</li>
</ul>

##### Replication
To keep around more than one copy of the log for reliability. This process is called **replication**.
Replication and partitioning are two core concepts in distributed system, 
<blockquote>
In any distributed system,
high availability (HA) through *replication*; 
scalability through *partitioning*.
</blockquote>
One possible way of doing a log replication is to define a (single) leader and a number of followers. The leader `writes` and keeps the principal copy of the log. The followers `read` from the leader’s log (managed by the leader) and update their own local log accordingly.
Adjust the parameter `replica.lag.time.max.ms` to change the default `lag time`.
Only the leader writes data originating from a data source to the log. The followers only ever
read from the leader.

The followers whose log is up to date with the leader are called in-sync replicas. Any of those
followers can take over the role of a leader if the current leader fails.


## Data elements
<img src="/assets/data_eles.png" height= "360" width= "580">

A record in Kafka consists of Metadata and a Body. The metadata contains offset, compression, magic byte, timestamp etc, Body part is `<key, value>`, 
`value` part is usually containing the business relevant data, 
`key` by default is used to decide into which partition a record is written to. Identical key of records go into 
the same partition, it also provides ordering in partition level. 

## Kafka Connect
Kafka Connect is a standard framework for source and sink connectors. It makes it easy to
import data from and export data to standard data sources such as relational DBs, HDFS,
cloud based blob storage, etc. It is a data integration tool for connecting with external systems.
<hr/>

#### 2.2  Fundamentals of Apache Kafka - 2 of 2
## High Level Architecture of Kafka Producer
<img src="/assets/kafka_producer.png" height= "400" width= "800">

## Serializer 
Kafka doesn't care about what you send to it as long as it's been converted to byte stream beforehand. That is also the format Kafka stores data in broker, the **binary**, it can store any format data, to keep Producer and Consumer communicate through this data format, but independently. So make sure the connection between Producer and Consumer clients, use serialization and deserialization to convert bytes to high-level data format.
In practice, message before `send()` to broker, should pass Interceptor, **Serializer, Partitioner**.

###### Batching 
Batching happens on a per topic partition level (for batch only happens in partition level, only messages written to the same partition are batched together).
Producer API write batch to `Kafka Broker`, by using either **ACK** or **NACK**.
<ul>
<li> - If <code>ACK</code> then <code>all is good</code> and success metadata is returned to the producer</li>
<li> - If <code>NACK</code> then the producer transparently retries until the <code>max number of retries</code> is reached, in which case an exception is returned to the producer</li>
</ul>

## Partitioning 
<img src="/assets/partitioining.png" height= "400" width= "800">

How to calculate partitions? => `partition= hash(key) % numPartitions`
if key is not provided, then partition will be done in `round robin` basis. 

#### Key Cardinality
For key based partitioning, it will have key skew etc, so will cause key cardinality, which affects the amount of work done by the individual consumers in a group.
So poor key choice can lead to uneven workloads.

## Consumer Group
First why need Consumer Group? => high throughput, we are using distributed system now. For single consumer will consume all the records form partitions, 
use multiply consumers will run assigned partitions at the same time.

<small>*note: multiple consumer groups can consume from the same topic(s), that means consumers can be as many as user wants;
each consumer in a group can be assigned zero, one, or many partitions, but will not assign the same partition to more than one 
consumer in the same group at the same time, so if number of (consumer > partitions), then the extra consumer will be idle.</small>

<hr/>

## Questions?

a. Explain the relationship between topics and partitions? <br>
<small>=> Topics are logical containers in Kafka and (usually) contain records or data of same type.
Partitions are used to parallelized work with topics. Thus one topic has 1…n partitions</small>
<hr>

b. How does Kafka achieve high availability? <br>
<small>=> Kafka uses replication to achieve HA. Each partition of a topic is available in the cluster on
say 3 distinct brokers. In case of failure of one broker, still two additional copies of the
data are available.</small>
<hr>

c. What is the easiest way to integrate your RDBMS with Kafka? <br>
<small>=> By using Kafka Connect with a plugin/connector that suits your needs, e.g. a JDBC source
or sink connector, depending of the direction the data has to flow.</small>

</div>