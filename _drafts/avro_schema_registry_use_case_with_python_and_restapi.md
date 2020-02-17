for this case, I will make a separate exploratory to do the avro and with its own REST api, which I don't know before, so I would like to write and give some test with it, so that I can see how to connect with Kafka messaging system and also with the later Kafka Connect, as an industrial case. 

```
SCHEMA_REGISTRY_HOST_NAME=schema-registry
SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL=zookeeper-1:2181
KAFKA_HEAP_OPTS=-Xms32M -Xmx32M -verbose:gc
```
Confluent’s schema-registry, which provides a RESTful interface for storing and retrieving Apache Avro schemas.

Goal: use schema registery to store all the schema, use this to send and receive messages in Python using Avro. 

Avro Consumer: 
The Avro consumer is responsible for pulling a message from Kafka and decoding the content. A message is polled from the Kafka and unpacked to get the schema_id for decoding. The Avro consumer also has a cache, which manages schema_id to schema mappings. The Avro consumer looks into the cache for the stored schema. If schema is not present, Avro consumer makes an API call to the schema-registry server to get the schema for the corresponding schema_id present in the message pack. Then it decodes the message against the Avro schema to get the original message structure.

Avro Producer:
The schema-registry server returns a unique schema_id when a new schema is registered. The Avro producer client takes a message and a schema as input. The client first checks the cache for schema_id for the corresponding schema. If the schema is already registered with schema-registry server, then that schema_id is picked from cache. The message is then encoded with the Avro schema and the schema_id is attached to it and published to Kafka topic. If the schema_id is not found in cache, then the schema is registered with schema-registry server and the schema_id is stored back in cache for future use.

Message Structure: 
In order to interoperate with other Kafka ecosystem consumers and producers that use the Schema registry, the standard message format payload is followed. Every message published to Kafka topic through this client will have three parts

Magic byte: Used to store the version of the schema for the payload being sent
Schema_id: Unique id associated with the Avro schema with which the message is been encoded
Message: Actual bytes of the encoded data



The REST Proxy is an HTTP-based proxy for your Kafka cluster. 

#### read the doc, which is the ability I should to build 
? 
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


