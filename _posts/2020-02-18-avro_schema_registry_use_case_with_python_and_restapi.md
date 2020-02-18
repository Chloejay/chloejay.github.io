---
layout: post
title:  "avro schema registry use case with Kafka REST"
date:   2020-02-18 22:28:53 +0800
categories: Kafka
tags: Python, Kafka, Zookeeper 
image: avro.jpeg
applause: true 
short_description: A short set up and use case with python for Kafka and Kafka REST.  
--- 

<div markdown="1" id="text">
I plan to make a separate article to do avro and its usage with Kafka REST, which I don't know before. I will set up and run a simplified exploratory, so that I can understand. Confluent’s schema-registry provides a RESTful interface for storing and retrieving Apache Avro schema, is an HTTP-based proxy for your Kafka cluster. Goal: use schema registery to store all the schema, to send and receive messages in Python using Avro. 

First let's jump into the Confluent website and install Kafka REST API. After install, the top-level folder structure will like below, 
<table>
  <tr>
    <th>Folder</th>
    <th>DESC</th>
  </tr>
  <tr>
    <th>/bin/</th>
    <th>Driver scripts for starting and stopping services</th>
  </tr>
  <tr>
    <th>/etc/</th> 
    <th>Configuration files</th> 
  </tr>
  <tr>
    <th>/lib/</th>
    <th>Systemd services</th>
  </tr>
  <tr>
    <th>/logs/</th>
    <th>Log files</th> 
  </tr>
  <tr>
    <th>/share/</th>
    <th>Jars and licenses</th> 
  </tr>
  <tr>
    <th>/src/</th>
    <th>Source files that require a platform-dependent build</th> 
  </tr>
</table>

<!--more-->

###### Config on /etc/ 
```
$ vim /etc/kafka/zookeeper.properties

tickTime=2000
dataDir=/var/lib/zookeeper/
clientPort=2181
initLimit=5
syncLimit=2
#need 3 zookeeper servers running, on this case only on localhost
reference: https://stackoverflow.com/questions/15842553/zookeeper-network-ensemble-does-not-start-appropiately

server.1=localhost:2888:3888
server.2=localhost:2888:3888
server.3=localhost:2888:3888
autopurge.snapRetainCount=3
autopurge.purgeInterval=24

creating a file named myid, one for each server on /var/lib/zookeeper folder 
```
```
$ vim /etc/kafka/server.properties

zookeeper.connect=localhost:2181 #hostname is local at dev stage 
#comment default broker.id=0
#add 
broker.id.generation.enable=true
advertised.listeners=PLAINTEXT://localhost:9092
```

```
$ vim /etc/confluent-control-center/control-center-production.properties

bootstrap.servers=kafka1:9092 #which is hostname: port 
#change confluent.controlcenter.data.dir=/tmp/confluent/control-center
#to  
confluent.controlcenter.data.dir=/var/lib/confluent/control-center
zookeeper.connect=zookeeper1:2181
```

```
$ vim /etc/kafka/server.properties
#uncomment 
metric.reporters=io.confluent.metrics.reporter.ConfluentMetricsReporter 
confluent.metrics.reporter.bootstrap.servers=localhost:9092

# Uncomment the following line if the metrics cluster has a single broker
confluent.metrics.reporter.topic.replicas=1
```
```
$ vim /etc/kafka/connect-distributed.properties

consumer.interceptor.classes=io.confluent.monitoring.clients.interceptor.MonitoringConsumerInterceptor
producer.interceptor.classes=io.confluent.monitoring.clients.interceptor.MonitoringProducerInterceptor
```
###### Schema Registry 
```
$ vim /etc/schema-registry/schema-registry.properties

listeners=http://0.0.0.0:8081
kafkastore.bootstrap.servers=PLAINTEXT://hostname:9092,SSL://hostname2:9092
```
###### Config Zookeepr
```
$ <path-to-confluent>/bin/zookeeper-server-start <path-to-confluent>/etc/kafka/zookeeper.properties
```
###### Run kafka-rest 
```
$ <path-to-confluent>/bin/kafka-server-start <path-to-confluent>/etc/kafka/server.properties
```

###### Start Schema Registry
```
$ <path-to-confluent>/bin/schema-registry-start <path-to-confluent>/etc/schema-registry/schema-registry.properties
```
Probably you will have `miss myid` error, &#128531;
go to check var/lib/zookeeper/ to create myid file 
```
$ sudo sh -c "echo '1' > /var/lib/zookeeper/myid" #but really depend on your zookeeper installization, and stuck here to debug if need. 
$ sudo sh -c "echo '2' > /var/lib/zookeeper/myid"
$ sudo sh -c "echo '3' > /var/lib/zookeeper/myid"
```
Also another common bug is `ERROR Unexpected exception, exiting abnormally  (org.apache.zookeeper.server.quorum.QuorumPeerMain)
java.net.BindException: Address already in use` &#128531;

```
$ brew services stop zookeeper

#if you not install zookeeper by brew, do this either 
$ ./zkServer.sh stop
then 
$ ./zkServer.sh start 
```
```
$ vim /usr/local/Cellar/zookeeper #zoo.cfg to config the file especially the hostname 
```
Here is another bug, `INFO Unable to read additional data from server sessionid 0x0, likely server has closed socket, closing socket connection and attempting reconnect (org.apache.zookeeper.ClientCnxn) 
` &#128531; 
```
# when run successfully, you will see: 
[2020-02-18 21:24:48,481] INFO Server environment:user.dir=/Users/chloeji/confluentKafka/confluent-5.3.2 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-02-18 21:24:48,484] INFO tickTime set to 2000 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-02-18 21:24:48,484] INFO minSessionTimeout set to -1 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-02-18 21:24:48,484] INFO maxSessionTimeout set to -1 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-02-18 21:24:48,498] INFO Using org.apache.zookeeper.server.NIOServerCnxnFactory as server connection factory (org.apache.zookeeper.server.ServerCnxnFactory)
[2020-02-18 21:24:48,511] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)
[2020-02-18 21:25:05,894] INFO Accepted socket connection from /0:0:0:0:0:0:0:1:50467 (org.apache.zookeeper.server.NIOServerCnxnFactory)
[2020-02-18 21:25:05,900] INFO Client attempting to establish new session at /0:0:0:0:0:0:0:1:50467 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-02-18 21:25:05,905] INFO Creating new log file: log.1 (org.apache.zookeeper.server.persistence.FileTxnLog)
[2020-02-18 21:25:05,926] INFO Established session 0x100406808490000 with negotiated timeout 6000 for client /0:0:0:0:0:0:0:1:50467 (org.apache.zookeeper.server.ZooKeeperServer)
[2020-02-18 21:25:05,994] INFO Got user-level KeeperException when processing sessionid:0x100406808490000 type:create cxid:0x2 zxid:0x3 txntype:-1 reqpath:n/a Error Path:/brokers Error:KeeperErrorCode = NoNode for /brokers (org.apache.zookeeper.server.PrepRequestProcessor)
[2020-02-18 21:25:06,034] INFO Got user-level KeeperException when processing sessionid:0x100406808490000 type:create cxid:0x6 zxid:0x7 txntype:-1 reqpath:n/a Error Path:/config Error:KeeperErrorCode = NoNode for /config (org.apache.zookeeper.server.PrepRequestProcessor)
[2020-02-18 21:25:06,056] INFO Got user-level KeeperException when processing sessionid:0x100406808490000 type:create cxid:0x9 zxid:0xa txntype:-1 reqpath:n/a Error Path:/admin Error:KeeperErrorCode = NoNode for /admin (org.apache.zookeeper.server.PrepRequestProcessor)
[2020-02-18 21:25:06,353] INFO Got user-level KeeperException when processing sessionid:0x100406808490000 type:create cxid:0x15 zxid:0x15 txntype:-1 reqpath:n/a Error Path:/cluster Error:KeeperErrorCode = NoNode for /cluster (org.apache.zookeeper.server.PrepRequestProcessor)
[2020-02-18 21:25:07,264] INFO Got user-level KeeperException when processing sessionid:0x100406808490000 type:multi cxid:0x39 zxid:0x1d txntype:-1 reqpath:n/a aborting remaining multi ops. Error Path:/admin/preferred_replica_election Error:KeeperErrorCode = NoNode for /admin/preferred_replica_election (org.apache.zookeeper.server.PrepRequestProcessor)
[2020-02-18 21:25:07,523] INFO Got user-level KeeperException when processing sessionid:0x100406808490000 type:setData cxid:0x45 zxid:0x1e txntype:-1 reqpath:n/a Error Path:/config/topics/__confluent.support.metrics Error:KeeperErrorCode = NoNode for /config/topics/__confluent.support.metrics (org.apache.zookeeper.server.PrepRequestProcessor)
```

```
$ <path-to-confluent>/bin/kafka-server-start <path-to-confluent>/etc/kafka/server.properties

#when run successfully, you will see: 
[2020-02-18 21:25:07,675] INFO [Log partition=__confluent.support.metrics-0, dir=/tmp/kafka-logs] Loading producer state till offset 0 with message format version 2 (kafka.log.Log)
[2020-02-18 21:25:07,681] INFO [Log partition=__confluent.support.metrics-0, dir=/tmp/kafka-logs] Completed load of log with 1 segments, log start offset (merged: 0, local: 0) and log end offset 0 in 48 ms (kafka.log.Log)
[2020-02-18 21:25:07,684] INFO Kafka version: 5.3.2-ce (org.apache.kafka.common.utils.AppInfoParser)
[2020-02-18 21:25:07,685] INFO Kafka commitId: 231a0ebf45e66e60 (org.apache.kafka.common.utils.AppInfoParser)
[2020-02-18 21:25:07,685] INFO Kafka startTimeMs: 1582032307684 (org.apache.kafka.common.utils.AppInfoParser)
[2020-02-18 21:25:07,686] INFO [Log partition=__confluent.support.metrics-0, dir=/tmp/kafka-logs] Loading producer state till offset 0 with message format version 2 (kafka.log.Log)
[2020-02-18 21:25:07,710] INFO Completed load of log with 1 segments containing 1 local segments and 0 tiered segments, tier start offset 0, first untiered offset 0, local start offset 0, log end offset 0 (kafka.log.MergedLog)
[2020-02-18 21:25:07,712] INFO Created log for partition __confluent.support.metrics-0 in /tmp/kafka-logs with properties {compression.type -> producer, min.insync.replicas -> 1, message.downconversion.enable -> true, segment.jitter.ms -> 0, cleanup.policy -> [delete], flush.ms -> 9223372036854775807, confluent.tier.local.hotset.ms -> 86400000, confluent.tier.local.hotset.bytes -> -1, segment.bytes -> 1073741824, retention.ms -> 31536000000, flush.messages -> 9223372036854775807, confluent.tier.enable -> false, message.format.version -> 2.3-IV1, max.compaction.lag.ms -> 9223372036854775807, file.delete.delay.ms -> 60000, max.message.bytes -> 1000012, min.compaction.lag.ms -> 0, message.timestamp.type -> CreateTime, preallocate -> false, index.interval.bytes -> 4096, min.cleanable.dirty.ratio -> 0.5, unclean.leader.election.enable -> false, retention.bytes -> -1, delete.retention.ms -> 86400000, segment.ms -> 604800000, message.timestamp.difference.max.ms -> 9223372036854775807, segment.index.bytes -> 10485760}. (kafka.log.LogManager)
[2020-02-18 21:25:07,713] INFO [Partition __confluent.support.metrics-0 broker=1001] No checkpointed highwatermark is found for partition __confluent.support.metrics-0 (kafka.cluster.Partition)
[2020-02-18 21:25:07,714] INFO Replica loaded for partition __confluent.support.metrics-0 with initial high watermark 0 (kafka.cluster.Replica)
[2020-02-18 21:25:07,719] INFO [Partition __confluent.support.metrics-0 broker=1001] __confluent.support.metrics-0 starts at Leader Epoch 0 from offset 0. Previous Leader Epoch was: -1 (kafka.cluster.Partition)
[2020-02-18 21:25:07,737] WARN [Producer clientId=producer-1] Error while fetching metadata with correlation id 1 : {__confluent.support.metrics=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)
[2020-02-18 21:25:07,739] INFO [Producer clientId=producer-1] Cluster ID: _0nGBcLqTzSvB_HF-Vc2UA (org.apache.kafka.clients.Metadata)
[2020-02-18 21:25:07,904] INFO [Producer clientId=producer-1] Closing the Kafka producer with timeoutMillis = 9223372036854775807 ms. (org.apache.kafka.clients.producer.KafkaProducer)
[2020-02-18 21:25:07,909] INFO Successfully submitted metrics to Kafka topic __confluent.support.metrics (io.confluent.support.metrics.submitters.KafkaSubmitter)
[2020-02-18 21:25:11,516] INFO Successfully submitted metrics to Confluent via secure endpoint (io.confluent.support.metrics.submitters.ConfluentSubmitter)
```

###### Start Schema Registry
```
$ <path-to-confluent>/bin/schema-registry-start <path-to-confluent>/etc/schema-registry/schema-registry.properties

# when run successfully, you will see: 
[2020-02-18 21:29:38,742] INFO HV000001: Hibernate Validator 5.1.3.Final (org.hibernate.validator.internal.util.Version:27)
[2020-02-18 21:29:38,853] INFO Started o.e.j.s.ServletContextHandler@62d0ac62{/,null,AVAILABLE} (org.eclipse.jetty.server.handler.ContextHandler:855)
[2020-02-18 21:29:38,863] INFO Started o.e.j.s.ServletContextHandler@91c4a3f{/ws,null,AVAILABLE} (org.eclipse.jetty.server.handler.ContextHandler:855)
[2020-02-18 21:29:38,878] INFO Started NetworkTrafficServerConnector@6b5f8707{HTTP/1.1,[http/1.1]}{0.0.0.0:8081} (org.eclipse.jetty.server.AbstractConnector:292)
[2020-02-18 21:29:38,879] INFO Started @3455ms (org.eclipse.jetty.server.Server:410)
[2020-02-18 21:29:38,879] INFO Server started, listening for requests... (io.confluent.kafka.schemaregistry.rest.SchemaRegistryMain:44)
```
###### Connect with Confluent REST Proxy
```
$ <path-to-confluent>/bin/kafka-rest-start <path-to-confluent>/etc/kafka-rest/kafka-rest.properties

# when run successfully, you will see: 
[2020-02-18 21:32:07,654] INFO HV000001: Hibernate Validator 5.1.3.Final (org.hibernate.validator.internal.util.Version:27)
[2020-02-18 21:32:07,829] INFO Started o.e.j.s.ServletContextHandler@7a791b66{/,null,AVAILABLE} (org.eclipse.jetty.server.handler.ContextHandler:855)
[2020-02-18 21:32:07,839] INFO Started o.e.j.s.ServletContextHandler@40f1be1b{/ws,null,AVAILABLE} (org.eclipse.jetty.server.handler.ContextHandler:855)
[2020-02-18 21:32:07,858] INFO Started NetworkTrafficServerConnector@53dacd14{HTTP/1.1,[http/1.1]}{0.0.0.0:8082} (org.eclipse.jetty.server.AbstractConnector:292)
[2020-02-18 21:32:07,861] INFO Started @2587ms (org.eclipse.jetty.server.Server:410)
[2020-02-18 21:32:07,861] INFO Server started, listening for requests... (io.confluent.kafkarest.KafkaRestMain:56)
```
<hr/>
Above is how to set up and debug the zookeeper with Kafka platform. 
Let's now dive into code and to see how to write this with Python. 
&#128522;


```
SCHEMA_REGISTRY_HOST_NAME=schema-registry
SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL=localhost:2181
KAFKA_HEAP_OPTS=-Xms32M -Xmx32M -verbose:gc
```

###### Some basic infor for Avro usage between Consumer and Producer: 
###### Avro Consumer: <br/>
<small>
The Avro consumer is responsible for pulling a message from Kafka and decoding the content. A message is polled from the Kafka and unpacked to get the <strong>schema_id</strong> for decoding. The Avro consumer also has a cache, which manages schema_id to schema mappings. The Avro consumer looks into the cache for the stored schema. If schema is not present, Avro consumer makes an API call to the schema-registry server to get the schema for the corresponding schema_id present in the message pack. Then it decodes the message against the Avro schema to get the original message structure.
</small><br/>

###### Avro Producer: <br/>
<small>
The schema-registry server returns a unique <strong>schema_id</strong> when a new schema is registered. The Avro producer client takes a message and a schema as input. The client first checks the cache for schema_id for the corresponding schema. If the schema is already registered with schema-registry server, then that schema_id is picked from cache. The message is then encoded with the Avro schema and the schema_id is attached to it and published to Kafka topic. If the schema_id is not found in cache, then the schema is registered with schema-registry server and the schema_id is stored back in cache for future use.
</small>

Message Structure: 
In order to interoperate with other Kafka ecosystem consumers and producers that use the Schema registry, the standard message format payload is followed. Every message published to Kafka topic through this client will have three parts

<ul>
<li>Magic byte: Used to store the version of the schema for the payload being sent</li>
<li>Schema_id: Unique id associated with the Avro schema with which the message is been encoded</li>
<li>Message: Actual bytes of the encoded data</li>
</ul> 


#### read the doc, which is the ability I should to build 
what the HTTP header mean? 
what the payload exactingly mean? 
The payload contains a value schema to specify the format of the data (records with a single field username) and a set of records.


Schemas:
GET /schemas/ids/{int: id} 
```
GET /schemas/ids/1 HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```
Response 
```
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
  "schema": "{\"type\": \"string\"}"
}
```

Subject: name under which the schema is registered. It refers to the '<topic>-key' or '<topic>-value', depending on topic being registrred by the key schema or value schema.
```
GET /subjects HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```
Response: 
```
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

["subject1", "subject2"]
```

GET /subjects/(string: subject)/versions
Get a list of versions registered under the specified subject.
```
GET /subjects/test/versions HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```
Response: 
```
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

[
  1, 2, 3, 4
]
```

GET /subjects/(string: subject)/versions/(versionId: version)
Get a specific version of the schema registered under this subject
```
GET /subjects/test/versions/1 HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

Response:
```
HTTP/1.1 200 OK
Content-Type: application/vnd.schemaregistry.v1+json

{
  "name": "test",
  "version": 1,
  "schema": "{\"type\": \"string\"}"
}
```

POST and DELETE 
curl -i -X POST -H "Content-Type: application/vnd.kafka.avro.v1+json" --data
</div>
