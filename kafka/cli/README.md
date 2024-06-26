## kafka-topics.sh
### How to create a Kafka topic, how to list, how to describe a topic, how to change topic settings
```
kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --create --partitions 3 --replication-factor 1
kafka-topics.sh --bootstrap-server localhost:9092 --list
kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic first_topic
kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic first_topic --partitions 5
```
https://www.conduktor.io/kafka/kafka-topics-cli-tutorial/

## kafka-console-producer.sh
### How to produce messages with key in the Kafka Console Producer CLI?
```
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first_topic --property parse.key=true --property key.separator=:
```

## kafka-console-consumer.sh
### How to show both the key and value

```
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first_topic --formatter kafka.tools.DefaultMessageFormatter --property print.timestamp=true --property print.key=true --property print.value=true --from-beginning
```

## kafka-consumer-groups.sh
### Describe
``` 
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group consumer

```

### List all consumer groups
```
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list --state
```

### Reset offset
First, stop consumers. Then, run:
```
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-first-application --reset-offsets --to-earliest --execute --topic first_topic
```
or
```
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-first-application --reset-offsets --shift-by -2 --execute --topic first_topic
```
We can use one of --to-datetime, --by-period, --to-earliest, --to-latest, --shift-by, --from-file, --to-current

### Describe all consumer groups and state

```
./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --all-groups  --state

```

## kafka-config
### Change number of in-sync replicas on a topic level
```
./kafka-configs.sh --bootstrap-server localhost:9092 --alter --entity-type topics --entity-name configured-topic --add-config min.insync.replicas=2
```

### Change number of in-sync replicas for brokers
```
$ kafka-configs.sh --bootstrap-server localhost:9092 --alter --entity-type brokers --entity-default --add-config min.insync.replicas=2 
```

### Set Log retention
```
kafka-configs.sh --bootstrap-server localhost:9092 --alter --entity-type topics --entity-name configured-topic --add-config retention.ms=-1,retention.bytes=524288000
```

## kafka-reassign-partitions
### Expand the cluster
https://kafka.apache.org/documentation/#basic_ops_increase_replication_factor 

```
bin/kafka-reassign-partitions.sh --bootstrap-server localhost:9092 --topics-to-move-json-file topics-to-move.json --broker-list "5,6" --generate
```

The new assignment should be saved in a json file (e.g. expand-cluster-reassignment.json) to be input to the tool with the --execute option as follows:
```
bin/kafka-reassign-partitions.sh --bootstrap-server localhost:9092 --reassignment-json-file expand-cluster-reassignment.json --execute
```

To verify:
```
bin/kafka-reassign-partitions.sh --bootstrap-server localhost:9092 --reassignment-json-file expand-cluster-reassignment.json --verify
```
