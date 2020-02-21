---
layout: post
title:  "Avro and Kafka REST for ETL- Part 2"
date:   2020-02-21 01:20:53 +0800
categories: python kafka 
tags: python, data integration
image: mount_fr.jpeg
applause: true
short_description: A guide for how to use kafka for real time data processing with apache spark stream. 
--- 

<div markdown="1" id="text">
Applying REST Proxy and Avro Scema Registry is good for front-facing, be it APP or any API rendering data or whatever. So we are using Apache Avro as a schema writer, to encode and decode messages between Kafka Producer and Consumer(s). Avro is different from JSON, it has schema, which is binary, well but it's flexible and easy to use. At least it's the schema that Confluent highly recommend. Key value and key schema which can be used to register the schema: 
- topic-key: unique integer. 
- topic-value, the result of streamed message, is the payload of message.<br/> 
For Confluent supports many good blogs to introduce Avro and related Avro official <a href='https://github.com/apache/avro/blob/master/lang/py3/avro/io.py'>doc</a>. So now let's move to implement with Python code to produce and consume messages that be sent to Kafka server. Since I have tested code running with Kafak, multiply Zookeepr nodes and Confluent Schema registry and Kafka REST on the previous exploratory article. Now let's dive into get hands dirty time. You can see the Python code <a href='https://github.com/Chloejay/streampipe/blob/master/kafka/kafkatest/producer.py'>here</a> I write for reference. 

Beaware of that Avro schema should be both as the encode the bytes on Producer side and decode on Consumer side. 

<!--more--> 

The general avro schema looks like below, which including `namespace`, `name`, `type` <a href='https://avro.apache.org/docs/current/spec.html'> avro supports 6 complex types, check more from avro doc</a> and `field`. 
```{
   "namespace": "general_name_for_message_content",
   "name": "detailed_description_of_the_message",
   "type": "record",
   "fields" : [
     {
       "name" : "col1",
       "type" : "int" #check more field types from avro doc 
     }]
}
```

###### During set up and POST schema to the REST 

some bugs I came across, those two are easy, which need to go back to recheck your schema which should written in bytes formats. 
`{"error_code":500,"message":"Internal Server Error com.fasterxml.jackson.databind.JsonMappingException: Illegal unquoted character ((CTRL-CHAR, code 10)): has to be escaped using backslash to be included in string value\n`

`{"error_code":500,"message":"Internal Server Error com.fasterxml.jackson.databind.JsonMappingException: Unrecognized character escape (CTRL-CHAR, code 10)\n`

```
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
 --data '{"schema":"{\"namespace\":\"avro\",\"name\": \"avrotest\",\
 \"type\": \"record\",\"fields\":[{\"name\": \"testing\",\"type\": \
 \"string\"},{\"name\":\"favorite_number\",\"type\": \"int\"},\
 {\"name\":\"favorite_color\",\"type\":\"string\"}]}"}'\
 http://localhost:8081/subjects/avrotesting-value/versions

=>{"id":23}
```

`AttributeError: 'str' object has no attribute 'type'` when run the encoder code, check and code and when `import avro.schema`, instead of using avro.schema.parse, use avro.schema.<strong>Parse()</strong> instead for Python 3+. 

<strong>Spoiler alert: always source code is the first source should go for</strong>, check the source code https://github.com/apache/avro/blob/master/lang/py3/avro/io.py, the one is very useful when you debug. 

 `File "/Users/chloeji/anaconda3/lib/python3.7/site-packages/avro/io.py", line 817, in write
    raise AvroTypeException(self.writer_schema, datum)
avro.io.AvroTypeException: The datum {'name': 'avrotest'} is not an example of the schema {
  "type": "record",
  "name": "avrotest",
  "fields": [
    {
      "type": "string",
      "name": "testing"
    },
    {
      "type": "int",
      "name": "favorite_number"
    },
    {
      "type": "string",
      "name": "favorite_color"
    }
  ]
}`

Be ware of the input data type and structure, which REST needs strictly be written, otherwise it fails. 

###### when done, check the producer message. 
```
kafka-avro-console-consumer --bootstrap-server localhost:9092 \
                              --property schema.registry.url=http://localhost:8081 \
                              --topic avrofix \
                              --from-beginning --max-messages 1 
```

###### Let's go through the code. I will explain how to use Avro schema on another article, the use case without Kafka REST. 
```Python 
from kafka import KafkaProducer, SimpleProducer, KafkaClient 
import avro 
from avro import schema 
import avro.io
from avro.io import DatumWriter
import requests 
import struct
import logging
logging.basicConfig(level = logging.INFO)

MAGIC_BYTE = 0
SCHEMA_REGISTRY_URL = 'http://localhost:8081'

def find_latest_schema(topic: str)-> str:
    subject= topic +'-value' 
    versions_res= requests.get(
        url= '{}/subjects/{}/versions'.format(SCHEMA_REGISTRY_URL, subject),
    headers={'Content-Type':'application/vnd.schemaregistry.v1+json',},)
    
    latest_version= versions_res.json()[-1]
    schema_res= requests.get(
        url= '{}/subjects/{}/versions/{}'.format(SCHEMA_REGISTRY_URL, subject, latest_version),
        headers={
                'Content-Type':'application/vnd.schemaregistry.v1+json',},)
    schema_res_json= schema_res.json() 
   
    return schema_res_json['id'], avro.schema.Parse(schema_res_json['schema'])
```
Use requests to get API http://localhost:8081, when for the production, replace with any cloud server instead. Get schema version id and schema value, which is unique for each schema. 

```Python 
class ContextStringIO(io.BytesIO):
    """
    Wrapper to allow use of StringIO via 'with' constructs.
    """
    def __enter__(self):
        return self

    def __exit__(self, *args):
        self.close()
        return False

def encode_producer(topic: str, record: dict) -> str: 
    '''
    Given a parsed avro schema, encode a record for the given topic. The record is expected to be a dictionary.
    :param str topic: Topic name
    :param dict record: An object to serialize
    '''
    try: 
        schema_id, schema= find_latest_schema(topic)
        if not schema:
            raise ('Schema does not exist')
    except Exception as e:
        logging.warning(e) 

    writer= avro.io.DatumWriter(schema)

    with ContextStringIO() as outf:
        #write the magic byte and schema ID in network byte order (big endian)
        outf.write(struct.pack('bI', MAGIC_BYTE, schema_id)) 
        #write ecord to the rest of buffer 
        encoder= avro.io.BinaryEncoder(outf)
        writer.write(record, encoder) 
        return outf.getvalue() 
```
Encode messages, here use a helper function ContextStringIO from official Confluent Kafka repo. 

```Python
def encode_writer(topic: str, _msg: str)-> str:
    KAFKA = KafkaClient('localhost:9092')
    producer = SimpleProducer(KAFKA)
    try: 
        producer.send_messages(topic, _msg)
        logging.info('Send message to topic {}'.format(topic)) 
    except Exception as e:
        logging.warning('Failed to send message to topic {}'.format(topic))
```
Use Kafka-Python client and create one Producer object, send encoded message to topics. You can check the real time records by `kafka-console-consumer --bootstrap-server localhost:9092 --topic avrofix --from-beginning`. 
</div> 