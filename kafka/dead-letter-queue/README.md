# Route messages to a dead letter queue - "Reliable Reprocessing"

What is DLQ? Simple topic in the Kafka cluster which acts as the destination for messages that were not able to make it to their desired destination due to some error.

Kafka Connect can be configured to send messages that it cannot process (such as a de-serialization error as seen in “fail fast” above) to a dead letter queue. Valid messages are processed as normal, and the pipeline keeps on running. Invalid messages can then be inspected from the dead letter queue, and ignored or fixed and reprocessed as required.

https://www.confluent.io/blog/kafka-connect-deep-dive-error-handling-dead-letter-queues/

https://quarkus.io/blog/kafka-failure-strategy/

[The dead-letter queue](https://en.wikipedia.org/wiki/Dead_letter_queue) is a well-known pattern to handle message processing failure. Instead of failing fast or ignoring and continuing the processing, it stores the failing messages into a specific destination: a dead letter. An administrator, human or software, can review the failing messages and decide what can be done (retry, skip, etc.). Note that you can only apply this strategy if the ordering is not essential to the application. So you can, later, review the failing messages:
```
Kafka Connect
errors.tolerance = all
errors.deadletterqueue.topic.name = <topic name>
 
Kafka Stream
default.deserialization.exception.handler
 
#Depending on your installation, Kafka Connect either writes this to stdout, or to a log file. Either way, you get a bunch of
#verbose output for each failed message. To enable this, set:
errors.log.enable = true
 
#You can also opt to include metadata about the message itself in the output. This metadata includes some of the same
#items you can see added to the message headers above, including the source message’s topic and offset
errors.log.include.messages = true
```

How to read failed messages from DLQ?

Taking the dead letter queue of the first as the source topic and attempting to deserialize the records as JSON. All we do here is change the value.converter and key.converter, the source topic name and the name for the dead letter queue (to avoid recursion if this connector has to route any messages to a dead letter queue).

```

curl -X POST http://localhost:8083/connectors -H "Content-Type: application/json" -d '{
        "name": "file_sink_06__02-json",
        "config": {
                "connector.class": "org.apache.kafka.connect.file.FileStreamSinkConnector",
                "topics":"dlq_file_sink_06__01",
                "file":"/data/file_sink_06.txt",
                "value.converter":"org.apache.kafka.connect.json.JsonConverter",
                "value.converter.schemas.enable": false,
                "key.converter":"org.apache.kafka.connect.json.JsonConverter",
                "key.converter.schemas.enable": false,
                "errors.tolerance":"all",
                "errors.deadletterqueue.topic.name":"dlq_file_sink_06__02",
                "errors.deadletterqueue.topic.replication.factor":1,
                "errors.deadletterqueue.context.headers.enable":true,
                "errors.retry.delay.max.ms": 60000,
                "errors.retry.timeout": 300000
                }
        }'
```