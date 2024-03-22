1. docker compose up -d
2. docker exec -it broker bash
3. kafka-topics --create --topic parallel-consumer-input-topic --bootstrap-server broker:9092 --replication-factor 1 --partitions 1 
4. gradle wrapper  // generate download  the gradle  wrapper 
5. ./gradlew shadowJar

6. java -cp build/libs/confluent-parallel-consumer-application-standalone-0.0.1-all.jar io.confluent.developer.ParallelConsumerApplication configuration/dev.properties

7. kafka-console-producer --topic parallel-consumer-input-topic --bootstrap-server broker:9092 --property "parse.key=true" --property "key.separator=:"
8. Run Test : ./gradlew test

# Performance test 

1. docker exec -it broker bash

2. kafka-topics --create --topic perftest-parallel-consumer-input-topic --bootstrap-server broker:9092 --replication-factor 1 --partitions 6
3. ./gradlew shadowJar

## Compile and run the KafkaConsumer-based performance test

4. java -cp build/libs/confluent-parallel-consumer-application-standalone-0.0.1-all.jar io.confluent.developer.MultithreadedKafkaConsumerPerfTest configuration/perftest-kafka-consumer.properties

## Confluent based perf test class perf test run it 

5. java -cp build/libs/confluent-parallel-consumer-application-standalone-0.0.1-all.jar io.confluent.developer.ParallelConsumerPerfTest configuration/perftest-parallel-consumer.properties


### In this section of the tutorial, we created a performance test for the Confluent Parallel Consumer, and a KafkaConsumer baseline to which to compare.

This gave us a couple of data points, but only for one specific test context: each test aimed to consume records as quickly as possible in a single JVM while simulating a 20ms workload per-record.

We can turn a few knobs and pull some levers to gather more performance test results in other application contexts. Since we used helper classes and parameterized configuration in this tutorial, you can easily choose other performance test adventures. Some questions you might explore:

How does performance compare if we increase or decrease the simulated workload time?

What if we commit offsets more frequently or even synchronously or transactionally in each test? In the case of the Confluent Parallel Consumer, this entails setting parallel.consumer.seconds.between.commits to a value lower than 60 seconds, and using a parallel.consumer.commit.mode of PERIODIC_CONSUMER_SYNC or PERIODIC_TRANSACTIONAL_PRODUCER. These commit modes better simulate an application designed to more easily pick up where it left off when recovering from an error.

What if we change the properties of the KafkaConsumer instance(s) most relevant to throughput (fetch.min.bytes and max.poll.records)?

What if we use KEY or PARTITION ordering when configuring the Confluent Parallel Consumer (as opposed to UNORDERED)?

How does the throughput comparison change if we create perftest-parallel-consumer-input-topic with more (or fewer) partitions?

What if we use larger, more realistic records and not just integers from 1 to 10,000? What if we also play with different key spaces?
