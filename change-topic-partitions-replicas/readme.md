mkdir change-topic-partitions-replicas && cd change-topic-partitions-replicas
mkdir src test
docker exec -it broker kafka-topics --bootstrap-server broker:29092 --topic topic1 --create --replication-factor 1 --partitions 1
docker exec -t broker kafka-topics --bootstrap-server broker:29092 --topic topic1 --describe

docker exec -it ksqldb-cli ksql http://ksqldb-server:8088

CREATE STREAM S1 (COLUMN0 VARCHAR KEY, COLUMN1 VARCHAR) WITH (KAFKA_TOPIC = 'topic1', VALUE_FORMAT = 'JSON');

CREATE STREAM S2 WITH (KAFKA_TOPIC = 'topic2', VALUE_FORMAT = 'JSON', PARTITIONS = 2, REPLICAS = 2) AS SELECT * FROM S1;

docker exec -t broker kafka-topics --bootstrap-server broker:29092 --topic topic2 --describe

CREATE STREAM S1 (COLUMN0 VARCHAR KEY, COLUMN1 VARCHAR) WITH (KAFKA_TOPIC = 'topic1', VALUE_FORMAT = 'JSON');

CREATE STREAM S2 WITH (KAFKA_TOPIC = 'topic2', VALUE_FORMAT = 'JSON', PARTITIONS = 2, REPLICAS = 2) AS SELECT * FROM S1;



docker exec -i broker kafka-console-producer \
  --bootstrap-server broker:29092 \
  --topic topic1 \
  --property parse.key=true \
  --property key.separator=,

