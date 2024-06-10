1. mkdir console-consumer-read-specific-offsets-partition && cd console-consumer-read-specific-offsets-partition

2. docker compose up -d 

3. kafka-topics --create --topic example-topic --bootstrap-server broker:9092 --replication-factor 1 --partitions 

4. kafka-console-producer --topic example-topic --bootstrap-server broker:9092 \
  --property parse.key=true \
  --property key.separator=":"


5 Read from partitio 


kafka-console-consumer --topic example-topic --bootstrap-server broker:9092 \
 --from-beginning \
 --property print.key=true \
 --property key.separator="-" \
 --partition 0


 kafka-console-consumer --topic example-topic --bootstrap-server broker:9092 \
 --from-beginning \
 --property print.key=true \
 --property key.separator="-" \
 --partition 1


 kafka-console-consumer --topic example-topic --bootstrap-server broker:9092 \
 --property print.key=true \
 --property key.separator="-" \
 --partition 1 \
 --offset 6