# KSQLDB/Postgres to PostgreSQL using Kafka Connect

> My objective would be to quickly set up a data pipeline and move data from KSQL/PostgresSQL to PostgreSQL.

## Debezium

We would use the Debezium KSQL connector for reading KSQL Database logs. Debezium would put the events in Kafka.

## Kafka-Connect

We would use the JDBC Connector to read the events from the Kafka Topic and put in PostgreSQL.

## Build the pipeline

We would use docker-compose to set up the following:

- KSQLDB-Server: The source DB.
- KSQLDB-CLI
- PostgreSQL: The destination DB
- Kafka-Connect(Debezium and JDBC Connector): Debezium for reading MySQL Logs and JDBC Connector for pushing the change to PostgreSQL.
- Kafka
- Schema Registry
- Zookeeper

To get a local copy up and running follow these simple example steps.

### Prerequisites

```
git clone https://github.com/akbari4yaseen/ksqldb-postgres-kafka-connect.git
```

## Getting Started

> Build the image

To build the Kafka-connect, we will add the JDBC connector and PostgreSQL JDBC Driver to our Kafka-Connect image.

```bash
cd debezium-jdbc
docker-compose build .
```

> At the root of the project folder run:

```bash
docker-compose up -d
```

> PostgresSQL

```bash
CREATE DATABASE kafka_test;
```

> Debezium Connector

Execute the following curl command to set up the Debezium connector for reading the logs from PostgresSQL

```bash
curl http://localhost:8083/connectors -i -X POST -H 'Content-Type: application/json' -d  '{"name": "postgres-source",
  "config": {"connector.class":"io.debezium.connector.postgresql.PostgresConnector",
    "tasks.max":"1",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "postgres",
    "database.password": "postgres",
    "database.dbname" : "kafka_test",
    "database.server.name": "dbserver1",
    "database.whitelist": "kafka_test",
    "database.history.kafka.bootstrap.servers": "kafka:9092",
    "database.history.kafka.topic": "schema-changes.kafka_test",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "io.confluent.connect.avro.AvroConverter",
    "key.converter.schemas.enable": "false",
    "value.converter.schemas.enable": "true",
    "value.converter.schema.registry.url": "http://schema-registry:8081",
    "transforms": "unwrap",
    "transforms.unwrap.type": "io.debezium.transforms.UnwrapFromEnvelope"
  }
}'
```

> JDBC connector

Execute the following curl command to set up the JDBC connector for writing the events from "kafka_test" KSQLDB to PostgreSQL.

We have kept "auto.create": "true" so that it automatically creates tables in PostgreSQL.

``` bash
curl http://localhost:8083/connectors -i -X POST -H 'Content-Type: application/json' -d '{"name": "lead-sink",
"config": {"connector.class":"io.confluent.connect.jdbc.JdbcSinkConnector",
  "tasks.max":"10",
  "topics": "lead_actors",
  "key.converter": "org.apache.kafka.connect.storage.StringConverter",
  "value.converter": "io.confluent.connect.avro.AvroConverter",
  "value.converter.schema.registry.url": "http://schema-registry:8081",
  "connection.url": "jdbc:postgresql://postgres:5432/kafka_test?user=postgres&password=postgres",
  "key.converter.schemas.enable": "false",
  "value.converter.schemas.enable": "true",
  "auto.create": "true",
  "auto.evolve": "true",
  "insert.mode": "upsert",
  "pk.fields": "title",
  "pk.mode": "record_key"
}
}'
```

You can start editing the connections by modifying `postgres-sink.json | postgres-source.json`.

👤 **Yaseen Akbari**

- GitHub: [@githubhandle](https://github.com/akbari4yaseen)
- Twitter: [@twitterhandle](https://twitter.com/AkbariYaseen)
- LinkedIn: [LinkedIn](https://linkedin.com/in/yaseen-akbari)

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

Feel free to check the [issues page](../../issues/).

## Show your support

Give a ⭐️ if you like this project!

## Play Around

Create "Table" and "Streams" in KSQL, see them getting reflected in PostgreSQL kafKa_test.

