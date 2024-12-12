# Basic CP docker example with KRaft + SR + Connect + C3

Just a basic example for easy prototyping of CP install for test purposes.

- [Basic CP docker example with KRaft + SR + Connect + C3](#basic-cp-docker-example-with-kraft--sr--connect--c3)
  - [Disclaimer](#disclaimer)
  - [Setup](#setup)
    - [Start Docker Compose](#start-docker-compose)
    - [Connect](#connect)
    - [Create topics](#create-topics)
    - [Create Connectors](#create-connectors)
    - [Check Control Center](#check-control-center)
  - [Execute the example](#execute-the-example)
  - [Cleanup](#cleanup)

## Disclaimer

The code and/or instructions here available are **NOT** intended for production usage. 
It's only meant to serve as an example or reference and does not replace the need to follow actual and official documentation of referenced products.

## Setup

### Start Docker Compose

```bash
docker compose up -d
```

### Connect

If you already have the plugin folders you can jump to next step.

You can check the connector plugins available by executing:

```bash
curl localhost:8083/connector-plugins | jq
```

As you see we only have source connectors:

```text
[
  {
    "class": "org.apache.kafka.connect.mirror.MirrorCheckpointConnector",
    "type": "source",
    "version": "7.6.0-ce"
  },
  {
    "class": "org.apache.kafka.connect.mirror.MirrorHeartbeatConnector",
    "type": "source",
    "version": "7.6.0-ce"
  },
  {
    "class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
    "type": "source",
    "version": "7.6.0-ce"
  }
]
```

Let's install confluentinc/kafka-connect-datagen connector plugin for sink.

For that we will open a shell into our connect container:

```bash
docker compose exec -it connect bash
```

Once inside the container we can install a new connector from confluent-hub:

```bash
confluent-hub install confluentinc/kafka-connect-datagen:latest
```

(Choose option 2 and after say yes to everything when prompted.)

Now if we list our plugins again we should see new one corresponding to the Datagen connector.

### Create topics 

Let's create first our topics with two partitions each:

```shell
kafka-topics --bootstrap-server localhost:19092 --topic customers --create --partitions 2 --replication-factor 1
kafka-topics --bootstrap-server localhost:19092 --topic orders --create --partitions 2 --replication-factor 1
```

### Create Connectors

Let's create our source connectors using datagen:

```bash
curl -i -X PUT -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/my-datagen-source2/config -d '{
    "name" : "my-datagen-source2",
    "connector.class": "io.confluent.kafka.connect.datagen.DatagenConnector",
    "kafka.topic" : "customers",
    "output.data.format" : "AVRO",
    "quickstart" : "SHOE_CUSTOMERS",
    "tasks.max" : "1"
}'
curl -i -X PUT -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/my-datagen-source3/config -d '{
    "name" : "my-datagen-source3",
    "connector.class": "io.confluent.kafka.connect.datagen.DatagenConnector",
    "kafka.topic" : "orders",
    "output.data.format" : "AVRO",
    "quickstart" : "SHOE_ORDERS",
    "tasks.max" : "1"
}'
```

### Check Control Center

Open http://localhost:9021 and check cluster is healthy including Kafka Connect.

## Execute the example

You can go to `kstreams` and execute `io.confluent.developer.App` with your IDE.
A `final` topic with the join should appear.

## Cleanup

```bash
docker compose down -v
```