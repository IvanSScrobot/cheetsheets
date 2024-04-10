How to manually handle znodes:
https://programmer.group/zookeeper-command-line-tool-zkcli.sh.html

to delete zookeeper info about Kafka brokers, go to /usr/local/kafka/bin/ and run 
```
./zookeeper-shell.sh zk1:2181
```
Then, use "ls" and "rmr" commands. Deleting path /controller might help if "existing cluster controller is just filling its logs with exceptions and errors or the JVM process is hung".

to check if Kafka brokers are connected to Zookeeper, go to /usr/local/kafka/bin/ and run 
```
ZK=zk1:2181 && sudo ./zookeeper-shell.sh "${ZK}" ls /my-kafka/brokers/ids
```

to check if Zookeeper ok, from a 'deploy' node (for example) , run 
```
echo 'ruok' | nc "${ZK}" 2181; echo
```

Confirming the Election Status of a ZooKeeper Service:
```
echo "stat" | nc server.example.org 2181 | grep Mode

bin/zkServer.sh status
```

ZooKeeper commands: 
https://zookeeper.apache.org/doc/r3.4.5/zookeeperAdmin.html#sc_zkCommands

To quickly test Zookeeper:
```
echo stat | nc localhost 2181
```

To connect to Zk servr and work with it (browse etc):
```
/data1/hdp/3.1.4.0-315/zookeeper/bin/zkCli.sh -server 127.0.0.1:2181
```

To get all available Kafka brokers:
```
echo dump | nc localhost 2181 | grep brokers
```




Why is Zookeeper necessary for Apache Kafka?

https://www.conduktor.io/kafka/zookeeper-with-kafka/#Should-you-use-Zookeeper-with-Kafka-clients?-2

Controller election


The controller is one of the most important broking entity in a Kafka ecosystem, and it also has the responsibility to maintain the leader-follower relationship across all the partitions. If a node by some reason is shutting down, it’s the controller’s responsibility to tell all the replicas to act as partition leaders in order to fulfill the duties of the partition leaders on the node that is about to fail. So, whenever a node shuts down, a new controller can be elected and it can also be made sure that at any given time, there is only one controller and all the follower nodes have agreed on that.


Configuration Of Topics

The configuration regarding all the topics including the list of existing topics, the number of partitions for each topic, the location of all the replicas, list of configuration overrides for all topics and which node is the preferred leader, etc.


Access control lists

Access control lists or ACLs for all the topics are also maintained within Zookeeper.


Membership of the cluster

Zookeeper also maintains a list of all the brokers that are functioning at any given moment and are a part of the cluster.


https://low-orbit.net/zookeeper
