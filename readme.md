https://developer.confluent.io/tutorials/kafka-console-consumer-producer-basics/kafka.html

1. docker exec -t broker kafka-topics --create --topic orders --bootstrap-server broker:9092


2. kafka-console-consumer \
  --topic orders \
  --bootstrap-server broker:9092

3. kafka-console-producer \
  --topic orders \
  --bootstrap-server broker:9092

## with key value pair 
4.  kafka-console-producer \
  --topic orders \
  --bootstrap-server broker:9092 \
  --property parse.key=true \
  --property key.separator=":"

5.  kafka-console-consumer \
  --topic orders \
  --bootstrap-server broker:9092 \
  --from-beginning \
  --property print.key=true \
  --property key.separator="-" 



## Avro and Schema Registry.
https://developer.confluent.io/tutorials/kafka-console-consumer-producer-avro/kafka.html

1. docker exec -t broker kafka-topics --create --topic orders-avro --bootstrap-server broker:9092

2. a create a schema file orders-avro-schema.json
2. docker cp orders-avro-schema.json schema-registry:/home/appuser/orders-avro-schema.json

3. kafka-avro-console-consumer --topic orders-avro --bootstrap-server broker:9092 --property schema.registry.url=http://localhost:8081

4. docker exec schema-registry bash

5. kafka-avro-console-consumer \
  --topic orders-avro \
  --bootstrap-server broker:9092 \
  --property schema.registry.url=http://localhost:8081

6. kafka-avro-console-producer \
  --topic orders-avro \
  --bootstrap-server broker:9092 \
  --property schema.registry.url=http://localhost:8081 \
  --property value.schema="$(< /home/appuser/orders-avro-schema.json)"

with key value 

7 . kafka-avro-console-producer --topic orders-avro --bootstrap-server broker:9092  --property schema.registry.url=http://localhost:8081 --property value.schema="$(< /home/appuser/orders-avro-schema.json)" --property key.serializer=org.apache.kafka.common.serialization.StringSerializer --property parse.key=true --property key.separator=":"


8 -- to show all key in consumer 

kafka-avro-console-consumer \
  --topic orders-avro \
  --property schema.registry.url=http://localhost:8081 \
  --bootstrap-server broker:9092 \
  --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
  --property print.key=true \
  --property key.separator="-" \
  --from-beginning

Serializer desirialixer 


{
  "type": "record",
  "name": "primitives",
  "namespace": "io.confluent.avro.random.generator",
  "fields":
    [
      { "name": "key_field", "type": "long" },
      { "name": "value_field", "type": "double" }
    ]
}

## How to optimize your Kafka producer for throughput

When optimizing for performance, you'll typically need to consider tradeoffs between throughput and latency. Because of Kafka’s design, it isn't hard to write large volumes of data into it. But many of the Kafka configuration parameters have default settings that optimize for latency. If your use case calls for higher throughput, this tutorial walks you through how to use `kafka-producer-perf-test` to measure baseline performance and tune your producer for large volumes of data.

 --   confluent kafka topic create topic-perf
  
 -- docker run -v $PWD/configuration/ccloud.properties:/etc/ccloud.properties confluentinc/cp-server:7.3.0 /usr/bin/kafka-producer-perf-test \
    --topic topic-perf \
    --num-records 10000 \
    --record-size 8000 \
    --throughput -1 \
    --producer.config /etc/ccloud.properties


## How to use the Confluent Parallel Consumer
The Confluent Parallel Consumer is an open source Apache 2.0-licensed Java library that enables you to consume from a Kafka topic with a higher degree of parallelism than the number of partitions for the input data (the effective parallelism limit achievable via an Apache Kafka consumer group). This is desirable in many situations, e.g., when partition counts are fixed for a reason beyond your control, or if you need to make a high-latency call out to a database or microservice while consuming and want to increase throughput.

In this tutorial, you'll first build a small "hello world" application that uses the Confluent Parallel Consumer library to read a handful of records from Kafka. Then you'll write and execute performance tests at a larger scale in order to compare the Confluent Parallel Consumer with a baseline built using a vanilla Apache Kafka consumer group.

### build gradle 
 parallel-consumer-core dependency, which is available in Maven Central. This artifact includes the Confluent Parallel Consumer’s core API. There are also separate modules for using the Confluent Parallel Consumer with reactive API frameworks like Vert.x (parallel-consumer-vertx) and Reactor (parallel-consumer-reactor). These modules are out of scope for this introductory tutorial.



buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "com.github.jengelman.gradle.plugins:shadow:6.1.0"
    }
}

plugins {
    id "java"
}

sourceCompatibility = JavaVersion.VERSION_17
targetCompatibility = JavaVersion.VERSION_17
version = "0.0.1"

repositories {
    mavenCentral()
}

apply plugin: "com.github.johnrengelman.shadow"

dependencies {
    implementation "io.confluent.parallelconsumer:parallel-consumer-core:0.5.2.4"
    implementation "org.apache.commons:commons-lang3:3.12.0"
    implementation "org.slf4j:slf4j-simple:2.0.0"
    implementation "me.tongfei:progressbar:0.9.3"
    implementation 'org.awaitility:awaitility:4.2.0'

    testImplementation "junit:junit:4.13.2"
    testImplementation 'org.awaitility:awaitility:4.2.0'
    testImplementation "io.confluent.parallelconsumer:parallel-consumer-core:0.5.2.4:tests" // for LongPollingMockConsumer
}

test {
    testLogging {
        outputs.upToDateWhen { false }
        showStandardStreams = true
        exceptionFormat = "full"
    }
}

jar {
  manifest {
    attributes(
      "Class-Path": configurations.compileClasspath.collect { it.getName() }.join(" "),
    )
  }
}

shadowJar {
    archiveBaseName = "confluent-parallel-consumer-application-standalone"
}


package io.confluent.developer;


import io.confluent.parallelconsumer.ParallelConsumerOptions;
import io.confluent.parallelconsumer.ParallelStreamProcessor;
import org.apache.kafka.clients.consumer.Consumer;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.nio.file.Paths;
import java.util.Collections;
import java.util.Properties;

import static io.confluent.parallelconsumer.ParallelConsumerOptions.CommitMode.PERIODIC_CONSUMER_SYNC;
import static io.confluent.parallelconsumer.ParallelConsumerOptions.ProcessingOrder.KEY;
import static io.confluent.parallelconsumer.ParallelStreamProcessor.createEosStreamProcessor;
import static org.apache.commons.lang3.RandomUtils.nextInt;



/**
 * Simple "hello world" Confluent Parallel Consumer application that simply consumes records from Kafka and writes the
 * message values to a file.
 */
public class ParallelConsumerApplication {

  private static final Logger LOGGER = LoggerFactory.getLogger(ParallelConsumerApplication.class.getName());
  private final ParallelStreamProcessor<String, String> parallelConsumer;
  private final ConsumerRecordHandler<String, String> recordHandler;

  /**
   * Application that runs a given Confluent Parallel Consumer, calling the given handler method per record.
   *
   * @param parallelConsumer the Confluent Parallel Consumer instance
   * @param recordHandler    record handler that implements method to run per record
   */
  public ParallelConsumerApplication(final ParallelStreamProcessor<String, String> parallelConsumer,
                                     final ConsumerRecordHandler<String, String> recordHandler) {
    this.parallelConsumer = parallelConsumer;
    this.recordHandler = recordHandler;
  }

  /**
   * Close the parallel consumer on application shutdown
   */
  public void shutdown() {
    LOGGER.info("shutting down");
    if (parallelConsumer != null) {
      parallelConsumer.close();
    }
  }

  /**
   * Subscribes to the configured input topic and calls (blocking) `poll` method.
   *
   * @param appProperties application and consumer properties
   */
  public void runConsume(final Properties appProperties) {
    String topic = appProperties.getProperty("input.topic.name");

    LOGGER.info("Subscribing Parallel Consumer to consume from {} topic", topic);
    parallelConsumer.subscribe(Collections.singletonList(topic));

    LOGGER.info("Polling for records. This method blocks", topic);
    parallelConsumer.poll(context -> recordHandler.processRecord(context.getSingleConsumerRecord()));
  }

  public static void main(String[] args) throws Exception {
    if (args.length < 1) {
      throw new IllegalArgumentException(
          "This program takes one argument: the path to an environment configuration file.");
    }

    final Properties appProperties = PropertiesUtil.loadProperties(args[0]);

    // random consumer group ID for rerun convenience
    String groupId = "parallel-consumer-app-group-" + nextInt();
    appProperties.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);

    // construct parallel consumer
    final Consumer<String, String> consumer = new KafkaConsumer<>(appProperties);
    final ParallelConsumerOptions options = ParallelConsumerOptions.<String, String>builder()
        .ordering(KEY)
        .maxConcurrency(16)
        .consumer(consumer)
        .commitMode(PERIODIC_CONSUMER_SYNC)
        .build();
    ParallelStreamProcessor<String, String> eosStreamProcessor = createEosStreamProcessor(options);

    // create record handler that writes records to configured file
    final String filePath = appProperties.getProperty("file.path");
    final ConsumerRecordHandler<String, String> recordHandler = new FileWritingRecordHandler(Paths.get(filePath));

    // run the consumer!
    final ParallelConsumerApplication consumerApplication = new ParallelConsumerApplication(eosStreamProcessor, recordHandler);
    Runtime.getRuntime().addShutdownHook(new Thread(consumerApplication::shutdown));
    consumerApplication.runConsume(appProperties);
  }

}










