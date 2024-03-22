https://developer.confluent.io/tutorials/creating-first-apache-kafka-producer-application/kafka.html#prerequisites

1. docker compose up
2. docker exec broker bash
3. kafka-topics --create --topic output-topic --bootstrap-server broker:9092 --replication-factor 1 --partitions 1 
4. gradle wrapper
5. /gradlew shadowJar
6. java -jar build/libs/kafka-producer-application-standalone-0.0.1.jar configuration/dev.properties input.txt
7. kafka-console-consumer --topic output-topic \
 --bootstrap-server broker:9092 \
 --from-beginning \
 --property print.key=true \
 --property key.separator=" : "





