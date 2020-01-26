---
layout: post
title:  "How to use kafka with Spark for the ETL"
date:   2020-01-22 04:28:53 +0800
categories: python kafka 
tags: python 
# applause: true
short_description: A guide for how to use kafka for real time data processing with apache spark stream. 
--- 

<div markdown="1" id="text">
draft! 
Kafka as the cluster of broker, connect with the low-level producer and consumer APIs, a general distributed data stream processing platform. The data model in kafka is message, a key/value pair, key can be the string. Topic are the named space that let producer and consumer write and read the message. The topic in order to scale for comsumptition, which be split into the partitions, each partition live in each own broker. The whole principle of partition, it's the log, which is immutable. Producer produce the data and multiple consumers can read it from different offsets or seek backwards to old timestamps. 
<img src='https://miro.medium.com/max/1924/1*kQXkMQTrMrG4VJ3KZehaqA.png'> 
throughput, the default mode is asynchronous/non-blocking writes 
increase number of partitions to scale the consumers. 
data retention policy to decide how long the data can be stored in broker. 
a distributed commit log, processing event and store the event, intergrate with the legacy relational database which is not the envent system. 

exactly once semantic 
I choose to use kafka, I think one reason is it provides API for user-friendly Python and also easy to integrate to many other tools via its Kafka Connect. I have confusion when I tested with both use database connector through the code and Kafka Connect, they both works but I have one mentor told me, Kafka Connect is more industrial usage, at least which can make my life easier. 

architecture: 
data ingesters
do it fast, cheap, scale 
complex compute algorithm 

recently read one post 7 mistakes when using Apache Kafka <a href='https://blog.softwaremill.com/7-mistakes-when-using-apache-kafka-44358cd9cd6'>[link]</a>

Zookeeper, Kafka’s sidekick used for managing consumers
Kafka’s schema registry
Kafka Connect (pulled from Debezium), which will source and sink data back and forth to/from Postgres through Kafka
PostgreSQL (also pulled from Debezium and tailored for use with Connect)

## Set up 

## Produce message 

## Read message 

## Connect the data to the DWH 

## Way more, to do the data processing with the Spark Stream 


</div>