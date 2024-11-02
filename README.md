# Git Guide for Kafka, Zookeeper, Schema Registry, and Kafka UI Docker Compose Setup

This document provides a guide to set up a local development environment using Docker Compose for Kafka, Zookeeper, Schema Registry, and Kafka UI. 

## Prerequisites
- **Docker** and **Docker Compose** should be installed on your system.
- Basic knowledge of Docker Compose.

## Docker Compose Configuration

Below is an explanation of each service configured in the `docker-compose.yml` file.

### 1. **Zookeeper**

The `zookeeper` service manages Kafka brokers. Kafka relies on Zookeeper for broker coordination, leader election, and configuration management.

```yaml
zookeeper:
  container_name: zookeeper
  image: confluentinc/cp-zookeeper:7.4.4
  environment:
    ZOOKEEPER_CLIENT_PORT: 2181
    ZOOKEEPER_TICK_TIME: 2000
  ports:
    - 22181:2181  # Map Zookeeper's internal port 2181 to 22181 on your host
```

- **Image**: Uses Confluent's Zookeeper image (`cp-zookeeper:7.4.4`).
- **Ports**: Maps `2181` to `22181` on the host, allowing external access if needed.
- **Environment Variables**:
  - `ZOOKEEPER_CLIENT_PORT`: Defines the client port for connecting Kafka brokers.
  - `ZOOKEEPER_TICK_TIME`: Configures the basic time unit for Zookeeper's event management.

### 2. **Kafka**

The `kafka` service is the central component in this setup, running a Kafka broker that uses Zookeeper.

```yaml
kafka:
  container_name: kafka
  image: confluentinc/cp-kafka:7.4.4
  depends_on:
    - zookeeper
  ports:
    - 29092:29092
  environment:
    KAFKA_BROKER_ID: 1
    KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
    KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
    KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
    KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

- **Image**: Uses Confluent's Kafka image (`cp-kafka:7.4.4`).
- **Dependencies**: Kafka depends on Zookeeper to be ready before it starts.
- **Ports**: Maps `29092` (used for external connections).
- **Environment Variables**:
  - `KAFKA_BROKER_ID`: Unique ID for the broker.
  - `KAFKA_ZOOKEEPER_CONNECT`: Points to the Zookeeper service for broker coordination.
  - `KAFKA_ADVERTISED_LISTENERS`: Exposes Kafka on `localhost:29092` for local machine connections.
  - `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP`: Defines the security protocol mapping.
  - `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR`: Sets replication factor for offsets topic.

### 3. **Kafka UI**

Kafka UI provides a web interface to view topics, consumer groups, and other broker information.

```yaml
kafka-ui:
  image: provectuslabs/kafka-ui:latest
  depends_on:
    - kafka
  ports:
    - "8080:8080"
  environment:
    KAFKA_CLUSTERS_0_NAME: local
    KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092
    KAFKA_CLUSTERS_0_ZOOKEEPER: zookeeper:2181
```

- **Image**: Uses Kafka UI (`provectuslabs/kafka-ui`).
- **Dependencies**: Depends on Kafka.
- **Ports**: Maps `8080` for accessing the UI.
- **Environment Variables**:
  - `KAFKA_CLUSTERS_0_NAME`: Sets the name of the Kafka cluster.
  - `KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS`: Defines the connection string to Kafka.
  - `KAFKA_CLUSTERS_0_ZOOKEEPER`: Connects to Zookeeper for additional management.

### 4. **Schema Registry**

Schema Registry allows storing and retrieving schemas for data in Kafka topics, ensuring data compatibility.

```yaml
schema-registry:
  image: confluentinc/cp-schema-registry:latest
  hostname: schema-registry
  depends_on:
    - kafka
  ports:
    - "8081:8081"
  environment:
    SCHEMA_REGISTRY_HOST_NAME: schema-registry
    SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: PLAINTEXT://kafka:9092
    SCHEMA_REGISTRY_LISTENERS: "http://0.0.0.0:8081"
```

- **Image**: Uses Confluent's Schema Registry image.
- **Dependencies**: Schema Registry relies on Kafka.
- **Ports**: Exposes `8081` for external schema management.
- **Environment Variables**:
  - `SCHEMA_REGISTRY_HOST_NAME`: Sets the hostname for the Schema Registry.
  - `SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS`: Configures Kafka connection.
  - `SCHEMA_REGISTRY_LISTENERS`: Specifies the listener configuration.

## Volumes

```yaml
volumes:
  kafka_data1:
    driver: local
```

Defines a volume for storing Kafka data to ensure data persistence between container restarts.

## Running the Setup

1. Save the configuration above to a `docker-compose.yml` file.
2. Run the following command to start all services:

   ```bash
   docker-compose up -d
   ```

3. Access the Kafka UI at [http://localhost:8080](http://localhost:8080).
4. The Schema Registry is available at [http://localhost:8081](http://localhost:8081).

## Stopping the Services

To stop all services, run:

```bash
docker-compose down
```

## Conclusion

This Docker Compose setup provides a convenient way to experiment with Kafka, Zookeeper, Schema Registry, and Kafka UI. It's useful for development and testing environments.