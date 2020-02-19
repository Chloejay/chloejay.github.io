---
layout: post
title:  "avro schema registry with Kafka REST - Part 1"
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
```
port 2181 is used by ZooKeeper clients to connect to the ZooKeeper servers
port 2888 is used by peer ZooKeeper servers to communicate with each other
port 3888 is used for leader election
folder structure pattern: /var/lib/zookeeper/zookeeper-[id]/data
```
$ mkdir -p /var/lib/zookeeper/zookeeper-1 /var/lib/zookeeper/zookeeper-2 /var/lib/zookeeper/zookeeper-3
$ sudo cp -rp /usr/local/Cellar/zookeeper/3.4.14/* /var/lib/zookeeper/zookeeper-1
$ sudo cp -rp /usr/local/Cellar/zookeeper/3.4.14/* /var/lib/zookeeper/zookeeper-2
$ sudo cp -rp /usr/local/Cellar/zookeeper/3.4.14/* /var/lib/zookeeper/zookeeper-3
```
###### config  /var/lib/zookeeper/[zookeeper-id]/conf/zoo.cfg
creating a file named myid, one for each server on /var/lib/zookeeper folder and add its own unique id. 
```
$ sudo sh -c "echo '1' > /var/lib/zookeeper/zookeeper-1/data/myid" 
#but really depend on your zookeeper installization, and stuck here to debug if need. 
$ sudo sh -c "echo '2' > /var/lib/zookeeper/zookeeper-2/data/myid"
$ sudo sh -c "echo '3' > /var/lib/zookeeper/zookeeper-3/data/myid"
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

###### Bug you will have, maybe

- Probably you will have `miss myid` error, &#128531;
go to check var/lib/zookeeper/ to see if correctly create myid file and have at least three Zookeeper servers started respectively 
- Also another common bug is `ERROR Unexpected exception, exiting abnormally  (org.apache.zookeeper.server.quorum.QuorumPeerMain)
java.net.BindException: Address already in use`
solution is varies, you can solve this by running command to kill Zookeeper pid 
```
$ lsof -n -i :9092 | grep LISTEN
# kill the pid you saw
# or solve by stopping Zookeeepr if you install by brew. 
$ brew services stop zookeeper
#if you not install zookeeper by brew, do this either 
$ ./zkServer.sh stop
then 
$ ./zkServer.sh start 
```
- Another bug, 
`INFO Unable to read additional data from server sessionid 0x0, likely server has closed socket, closing socket connection and attempting reconnect (org.apache.zookeeper.ClientCnxn)` &#128531; 
```
$ vim /usr/local/Cellar/zookeeper #zoo.cfg to config the file especially the hostname 
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


###### Start Kafka Server start before running Kafka Rest to listen requests. 
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
<strong><small><span style='color:#f67280'>At the same time, you can run your Producer and Consumer to get the real time data when you use Rest.</span></small></strong>

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
###### Start Confluent REST Proxy
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

###### Let's first test the API with action POST
<p style='color:#f67280'><strong><small> 
To make easy, I suggest to use the new version or check your version before using, kafka.json.v2 works easily than kafka.avro.v1 for bug facing. 
</small></strong></p>

##### produce the message
```
curl -X POST -H "Content-Type: application/vnd.kafka.json.v2+json" \
          --data '{"value_schema": "{\"type\": \"record\", \"name\":\"User\", \"fields\"{\"name\": \"name\", \"type\":\"string\"}]}",
          "records":[{"value":{"name": "tester1"}},
                    {"value":{"name": "tester2"}},
                    {"value":{"name": "tester3"}},
                    {"value":{"name": "tester4"}}]
          }' \
          "http://localhost:8082/topics/avrotestV1"

#you will see 
{"offsets":[{"partition":1,"offset":0,"error_code":null,"error":null},{"partition":0,"offset":0,"error_code":null,"error":null},{"partition":29,"offset":0,"error_code":null,"error":null},{"partition":27,"offset":0,"error_code":null,"error":null}],"key_schema_id":null,"value_schema_id":null}

curl -X POST -H "Content-Type: application/vnd.kafka.v2+json" -H "Accept: application/vnd.kafka.v2+json" \
    --data '{"name": "my_consumer_instance", "format": "json", "auto.offset.reset": "earliest"}' \
    http://localhost:8082/consumers/my_json_consumer_group
#you will see 
{"instance_id":"my_consumer_instance","base_uri":"http://localhost:8082/consumers/my_json_consumer_group/instances/my_consumer_instance"}
```

##### consume message 
```
curl -X POST -H "Content-Type: application/vnd.kafka.v2+json" --data '{"topics":["avrotestV1"]}' \
    http://localhost:8082/consumers/my_json_consumer_group/instances/my_consumer_instance/subscription
#nothing return. 
curl -X GET -H "Accept: application/vnd.kafka.json.v2+json" \
    http://localhost:8082/consumers/my_json_consumer_group/instances/my_consumer_instance/records

# you will see 
[{"topic":"avrotestV1","key":null,"value":{"name":"tester2"},"partition":0,"offset":0},{"topic":"avrotestV1","key":null,"value":{"name":"tester3"},"partition":29,"offset":0},{"topic":"avrotestV1","key":null,"value":{"name":"tester4"},"partition":27,"offset":0},{"topic":"avrotestV1","key":null,"value":{"name":"tester1"},"partition":1,"offset":0}]
```

##### The bugs might happen:

1. `{"error_code":40403,"message":"Consumer instance not found."}`
`[2020-02-19 15:07:49,885] INFO 0:0:0:0:0:0:0:1 - - [19/Feb/2020:07:07:49 +0000] "GET /consumers/my_avro_consumer/instances/rest-consumer-2/topics/avrotest HTTP/1.1" 404 61  5 (io.confluent.rest-utils.requests:62)` &#128531;
the issue is caused by only run one instance, at least three as mentioned before. Let's go back to previous step `Start Schema Registry`. I know this issue is Zookeeper one, which as a layman, I have no knowledge related to this. But the situation now, I need to push myself a bit to get understand the basics. Back to previous step, as mentioned it a must that need at least 3 nodes to run on ZooKeeper as base requirement for distributed system, so let's add two more. This step is well known as <strong>Zookeeper ensemble</strong>.<br/>
<strong><small><span style='color:#f67280'>Pay attention to this three parameters, which need to be config correctly</span></small></strong>
- dataDir
- clientPort
- server.n #the server/instance number

2. after debug `[2020-02-19 18:18:09,008] WARN [Producer clientId=producer-3] 1 partitions have leader brokers without a matching listener, including [avrotest-0] (org.apache.kafka.clients.NetworkClient:1044)`
if only one partition but multiple comnsumers, it will make fails here, config your partition parameter in kafka.server.properties, the default one is 1, you can increase it, I think according to what I have read from the best Confluent blog <a href='https://www.confluent.io/blog/how-choose-number-topics-partitions-kafka-cluster/'>How to choose the number of topics/partitions in a Kafka cluster?</a>. 

3. Server error, `{"error_code":500,"message":"Internal Server Error"}`, which is caused by your schema when you produce message, so check it first, or do you missing the coma etc. <br/>
<strong><small><span style='color:#f67280'>
Error code 50001 – Zookeeper error.<br/>
Error code 50002 – Kafka error.<br/>
Error code 50003 – Retriable Kafka error. Although the operation failed, it’s possible that retrying the request will be successful.<br/>
Error code 50101 – Only SSL endpoints were found for the specified broker, but SSL is not supported for the invoked API yet.
</span></small></strong>

4. `INFO Got user-level KeeperException when processing sessionid:0x10040f0af820000 type:create cxid:0xd zxid:0x3f txntype:-1 reqpath:n/a Error Path:/config/brokers Error:KeeperErrorCode = NodeExists for /config/brokers (org.apache.zookeeper.server.PrepRequestProcessor)` 
https://stackoverflow.com/questions/43559328/got-user-level-keeperexception-when-processing
you can config the timeout, which works for me. 
consumer.instance.timeout.ms property which is set to 300000 ms by default. Try increasing it and you will not see the issue.

5. after workaround, it turns out is Error deserializing error. 
`HTTP/1.1 500 Internal Server Error
Date: Wed, 19 Feb 2020 13:19:23 GMT
Content-Type: application/vnd.kafka.avro.v1+json
Content-Length: 88
Server: Jetty(9.4.18.v20190429)
{"error_code":50002,"message":"Kafka error: Error deserializing Avro message for id -1"}
`
I solved it by use the kafka.json.v2 

<hr/>
So now it's all done, we can use it smootly by the API.
once is all tested, you can see from REST about all the topics you created. 

```
$ curl "http://localhost:8082/topics/topicName"

# This is my topics that I created on this couple months
["__confluent.support.metrics","_confluent-metrics","_schemas",
"avrotest","avrotest1","kafka","kafkadev","kafkatest","kafkatesting",
"orders","orders1","orders2","sparktest","test","testavro","testavro1"]

$ curl "http://localhost:8082/topics/avrotest1"
{"name":"avrotest1","configs":{"compression.type":"producer",
"leader.replication.throttled.replicas":"","min.insync.replicas":"1",
"message.downconversion.enable":"true","segment.jitter.ms":"0",
"cleanup.policy":"delete","flush.ms":"9223372036854775807",
"follower.replication.throttled.replicas":"","retention.ms":"604800000",
"segment.bytes":"1073741824","flush.messages":"9223372036854775807",
"message.format.version":"2.3-IV1",
"max.compaction.lag.ms":"9223372036854775807",
"file.delete.delay.ms":"60000","max.message.bytes":"1000012",
"min.compaction.lag.ms":"0","message.timestamp.type":"CreateTime",
"preallocate":"false","index.interval.bytes":"4096",
"min.cleanable.dirty.ratio":"0.5",
"unclean.leader.election.enable":"false","retention.bytes":"-1",
"delete.retention.ms":"86400000","segment.ms":"604800000",
"message.timestamp.difference.max.ms":"9223372036854775807",
"segment.index.bytes":"10485760"},"partitions":[{"partition":0,
"leader":0,"replicas":[{"broker":0,"leader":true,"in_sync":true}]}]
}
```
<hr/>
Above is how to set up and debug the zookeeper with Kafka platform. 
Let's now dive into code and to see how to write this with Python, viola Finally! 
&#128522;

###### Some basic infor for Avro usage between Consumer and Producer
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

<!-- what the payload exactingly mean? 
The payload contains a value schema to specify the format of the data (records with a single field username) and a set of records. -->

```
SCHEMA_REGISTRY_HOST_NAME=schema-registry
SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL=localhost:2181
KAFKA_HEAP_OPTS=-Xms32M -Xmx32M -verbose:gc
```

Schemas:
GET /schemas/ids/{int: id} 
```
GET /schemas/ids/1 HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```
Subject: name under which the schema is registered. It refers to the '<topic>-key' or '<topic>-value', depending on topic being registrred by the key schema or value schema.

```
GET /subjects HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

GET /subjects/(string: subject)/versions
Get a list of versions registered under the specified subject.
```
GET /subjects/test/versions HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

GET /subjects/(string: subject)/versions/(versionId: version)
Get a specific version of the schema registered under this subject
```
GET /subjects/test/versions/1 HTTP/1.1
Host: schemaregistry.example.com
Accept: application/vnd.schemaregistry.v1+json, application/vnd.schemaregistry+json, application/json
```

POST and DELETE 
curl -i -X POST -H "Content-Type: application/vnd.kafka.avro.v1+json" --data
</div>