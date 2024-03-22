1. mkdir console-consumer-read-specific-offsets-partition && cd console-consumer-read-specific-offsets-partition

2. docker compose up -d 

3. kafka-topics --create --topic example-topic --bootstrap-server broker:9092 --replication-factor 1 --partitions 

4. 
