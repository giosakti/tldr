# Kafka

Source Systems
Producer
Kafka
Consumer
Target Systems

## Kafka Extended API

Source Systems
Kafka Connect Source
Kafka Cluster <-> Kafka Streams
Kafka Connect Sink
Target Systems

## Topic and Partitions

Topics:
  - a particular stream of data
  - Similar to a table in a database (without all the constraints)
  - You can have as many topics as you want
  - A topic is identified by its name

Partitions:
  - Topics are split in partitions
  - Each partition is ordered
  - Each message within a partition gets an incremental id, called offset

Offset:
  - Only have a meaning for a specific partition
  - Order is guaranteed only within a partition
  - Data is kept only for a limited time (default is 2w)
  - Once the data is written to a partition, it can't be changed (immutability)
  - Data is assigned randomly to a partition unless a key is provided
  - You can have as many partitions per topic as you want

Brokers:
  - A kafka cluster is composed of multiple brokers (servers)
  - Each broker is identified with its ID
  - Each broker contains certain topic partitions (!)
  - After connecting to any broker, you will be connected to the entire cluster
  - A good number to get started is 3 brokers, but some big clusters have 100

Topic Replication Factor:
  - Topics should have a replication factor > 1

Partition Leader:
  - At any time, only 1 broker can be a leader for a given partition
  - Only that leader can receive and serve data for a partition
  - The other brokers will synchronize the data

Producers
  - Write data to topics
  - They only have to specify the topic name and one broker to connect to and Kafka will manage the rest
  - Producers can receive acknowledgement of data writes
    - Acks=0, won't wait for ack
    - Acks=1, producer will wait for leader ack
    - Acks=all, producer will wait for leader + replicas
  - Producers can choose to send a key with the message.
    - If the key is sent then the producer has the guarantee that all messages for that key will always go to the same partition
    - This enables to guarantee ordering for a specific key

Consumers
  - Read data from a topic
  - They only have to specify the topic name and one broker to connect to and Kafka will manage the rest
  - Data is read in order for each partitions but is read in parallel across partitions

Consumer Groups
  - Consumers read data in consumer groups
  - Each consumer within a group reads from exclusive partitions
  - You cannot have more consumers than partitions (in a customer group)

Consumer Offsets
  - Kafka stores the offsets at which a consumer group has been reading
  - The offsets commit live in a Kafka topic named "____consumer__offsets"
  - When a consumer has processed data received from Kafka, it should be committing the offsets

Zookeeper
  - Zookeper manage brokers
  - Zookeper helps in performing leader election for partitions
  - Zookeper sends notifications to Kafka in case of changes
  - Kafka can't work without Zookeeper
  - Zookeper usually operates in and odd quota (3,5,7)
  - Zookeper has a leader and rest is followers

Kafka Guarantees
  - Messages are appended to a topic-partition in the order they are sent
  - Consumers read messages in the order stored in a topic-partition
  - With a replication factor of N, producers and consumers can tolerate up to N-1 brokers being down
  - Replication factor of 3 is a good idea
  - As long as the number of partitions remains constant for a topic (no new partitions), the same key will always go to the same partition

Delivery Semantics for Consumers
  - Consumers choose when to commit offsets
    - At most once: offsets are committed as soon as the message is received. If the processing goes wrong, the message will be lost.
    - At least once: offsets are committed after the message is processed. If the processing goes wrong, the message will be read again. This can result in duplicate processing of messages. Make sure your processing is idempotent.
    - Exactly once: very difficult to achieve / needs strong engineering.

## Running Kafka

    # Run Kafka
    docker run --rm -it -p 2181:2181 -p 3030:3030 -p 8081:8081 -p 8082:8082 -p 8083:8083 -p 9092:9092 -e ADV_HOST=127.0.0.1 landoop/fast-data-dev

    # Open in browser: http://127.0.0.1:3030

    # Start Kafka CMD
    docker run --rm -it --net=host landoop/fast-data-dev bash

    # See the broker log
    docker exec -it <ID> tail -f /var/log/broker.log

## Kafka Basic Commands

    # List all topics
    kafka-topics --zookeeper 127.0.0.1:2181 --list

    # Create new topic
    # Replication factor must <= num of brokers
    kafka-topics --zookeeper 127.0.0.1:2181 --create --topic first_topic --partition 3 --replication-factor 3

    # Describe topic
    kafka-topics --zookeeper 127.0.0.1:2181 --topic second-topic --describe

    # Delete topic
    #
    # Delete topic is not very common in kafka, especially in production
    # Just create new topic instead
    kafka-topics --zookeeper 127.0.0.1:2181 --topic second-topic --delete

    # This will start interactive mode where you can produce message(s)
    # separated by newline
    #
    # This command will also create a topic on the first line if it's not created
    # Be careful, it's best to create a topic manually first rather than publishing
    # message directly.
    kafka-console-producer --broker-list 127.0.0.1:9092 --topic first-topic

    # This will start interactive mode for consuming messages
    # It will start from the very end of the queue
    kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first-topic <--from-beginning> <--partition partition_num> <--offset offset_num>

    # Specify consumer group
    # will commit read offset automatically
    kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first-topic --consumer-property group.id=mygroup1

    # Specify configuration use parameter --config <key>=<value>
    --config cleanup.policy=compact

### Partition & Repplication Factor

Roughly, each partition can get a throughput of 10MB/s
More partitions implies:
  - Better parallelism, better throughput
  - BUT more files opened on your system
  - BUT if a broker fails (unclean shutdown), lots of concurrent leader elections
  - BUT added latency to replicate (in the order of milliseconds)

Guidelines for partition:
  - Keep the number of partition per brokers between 2000 and 4000
  - Keep the number of partitions in the cluster to less than 20.000
  - Partitions per topic = (1 to 2) x (# of brokers), max 10 partitions

Replication factor should be at least 2, maximum of 3
The higher the replication factor:
  - Better resilience of your system (N-1 brokers can fail)
  - BUT longer replication (higher latency is acks=all)
  - BUT more disk space on your system (50% more if RF is 3 instead of 2)

Guidelines for replication factor:
  - Set it to 3 (you must have at least 3 brokers)
  - If replication performance is an issue, get a better broker instead of less RF

It is best to get the parameters (partitions count, replication factor) right the first time!
  - If the partitions count increase during a topic lifecycle, you will break your keys ordering guarantees
  - Partition can only be added, not removed
  - Over-partition is better than under-partition
  - If the replication factor increases during a topic lifecycle, you put more pressure on your cluster, which can lead to unexpected performance decrease

### Partitions and Segments

Topics are made of partitions
Partitions are made of segments (files)

    log.segment.bytes: the max size of a single segment in bytes
    log.segment.ms: the time Kafka will wait before committing the segment if not full

Segments come with two indexes (files)
- An offset to position index
- A timestamp to offset index
- Find data in O(1)

A smaller ```log.segment.bytes``` (size, default: 1GB) means:
  - more segments per partitions
  - log compaction happens more often
  - BUT kafka has to keep more files opened (Error:Too many open files)
Ask yourself: How fast will I have new segments based on throughput?

A smaller ```log.segment.ms``` (time, default 1 week) means:
  - You set a max frequency for log compaction (more frequent triggers)
  - Maybe you want daily compaction instead of weekly?
Ask yourself: How often do I need log compaction to happen?

### Log Cleanup Policies

Many kafka clusters make data expire, according to a policy called log cleanup

```log.cleanup.policy=delete```(kafka default)
  - ```log.retention.hours```: Delete based on age of data (default is a week)
  - ```log.retention.bytes```Delete based on max size of log (default is -1 == infinite)
  - Two common pair of options:
    - ```log.retention.hours=168``` and ```log.retention.bytes=-1```
    - ```log.retention.hours=17520``` and ```log.retention.bytes=524288000```

```log.cleanup.policy=compact``` (kafka default for topic ____consumer__offsets)
  - Delete based on keys of your messages
  - Will delete old duplicate keys after the active segment is committed
  - Infinite time and space retention
  - Log compaction ensures that your log contains at least the last known value for a specific key within a partition
  - Very useful if we just require a SNAPSHOT instead of full history (such as for a data table in a database)

Log cleanup guarantees:
  - Any consumer that is reading from the head of a log will still see all the messages sent to the topic
  - Ordering of messages it kept
  - The offset of a message is immutable
  - Deleted records can still be seen by consumers for a period of ```delete.retention.ms```

Log cleanup myth busting:
  - It doesn't prevent you from pushing duplicate data to kafka
  - It doesn't prevent you from reading duplicate data from kafka
  - Log compaction can fail from time to time

Deleting data from Kafka allows you to:
- Control the size of the data on the disk, delete obsolete data
- Overall: Limit maintenance work on the Kafka cluster

How often does log cleanup happen?
- Log cleanup happens on your partition segments!
- Smaller / more segments means that log cleanup will happen more often!
- Log cleanup shouldn't happen too often => takes CPU and RAM resources
- The cleaner checks for work every 15 seconds (```log.cleaner.backoff.ms```)

### Topic Compression

Topics can be compressed using ```compression.type```
  - options ```gzip, snappy, lz4, uncompressed, producer```
  - If you need compression, ideally you keep default as ```producer```
    - The producer will perform the compression on its side
    - The broker will take the data as is => saves CPU resources on the broker
  - If compression is enabled, make sure you're sending batches of data
  - Data will be uncompressed by the consumer
  - Compression only makes sense if you're sending non-binary data

### Other Advanced Configurations

- ```max.messages.bytes``` (default is 1MB): if your messages get bigger than 1MB, increase this parameter on the topic and your consumers buffer
- ```min.isync.replicas``` (default is 1): if using acks=all, specify how many brokers need to acknowledge the write. see https://logallthethings.com/2016/07/11/min-insync-replicas-what-does-it-actually-do/
- ```unclean.leader.election``` (default is true): if set to true, it will allow replicas who are not in sync to become leader as a last resort if all ISRs are offline. This can lead to data loss. If set to false the topic will go offline until the ISRs come back up.
