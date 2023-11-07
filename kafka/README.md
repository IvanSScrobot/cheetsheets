## Useful commands
```

```

## How to replay Kafka offset
Guide: 

https://gist.github.com/marwei/cd40657c481f94ebe273ecc16601674b#file-how_to_reset_kafka_consumer_group_offset-md

### Highlight:
```
Kafka 0.11.0.0 (Confluent 3.3.0) added support to manipulate offsets for a consumer group via cli kafka-consumer-groups command.

List the topics to which the group is subscribed bash kafka-consumer-groups --bootstrap-server <kafkahost:port> --group <group_id> --describe Note the values under "CURRENT-OFFSET" and "LOG-END-OFFSET". "CURRENT-OFFSET" is the offset where this consumer group is currently at in each of the partitions.

Reset the consumer offset for a topic (preview) bash kafka-consumer-groups --bootstrap-server <kafkahost:port> --group <group_id> --topic <topic_name> --reset-offsets --to-earliest This will print the expected result of the reset, but not actually run it.

Reset the consumer offset for a topic (execute) bash kafka-consumer-groups --bootstrap-server <kafkahost:port> --group <group_id> --topic <topic_name> --reset-offsets --to-earliest --execute This will execute the reset and reset the consumer group offset for the specified topic back to 0.

Repeat 1 to check if the reset is successful

Note
The consumer group must have no running instance when performing the reset. Otherwise the reset will be rejected.
There are many other resetting options, run kafka-consumer-groups for details

--shift-by
--to-current
--to-latest
--to-offset
--to-datetime
--by-duration
The command also provides an option to reset offsets for all topics the consumer group subscribes to: --all-topics
```