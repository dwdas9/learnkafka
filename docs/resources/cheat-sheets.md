# Kafka Cheat Sheets

## Quick Reference

Essential commands and configurations for daily Kafka operations.

---

## Topic Management

```bash
# Create topic
kafka-topics --create \
  --topic my-topic \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1

# List topics
kafka-topics --list --bootstrap-server localhost:9092

# Describe topic
kafka-topics --describe --topic my-topic --bootstrap-server localhost:9092

# Delete topic
kafka-topics --delete --topic my-topic --bootstrap-server localhost:9092

# Alter topic (add partitions)
kafka-topics --alter --topic my-topic --partitions 6 --bootstrap-server localhost:9092
```

---

## Producer Commands

```bash
# Console producer
kafka-console-producer --topic my-topic --bootstrap-server localhost:9092

# With key
kafka-console-producer --topic my-topic \
  --property "parse.key=true" \
  --property "key.separator=:" \
  --bootstrap-server localhost:9092

# Performance test
kafka-producer-perf-test \
  --topic test \
  --num-records 1000000 \
  --record-size 1000 \
  --throughput -1 \
  --producer-props bootstrap.servers=localhost:9092
```

---

## Consumer Commands

```bash
# Console consumer (from beginning)
kafka-console-consumer --topic my-topic \
  --from-beginning \
  --bootstrap-server localhost:9092

# With group
kafka-console-consumer --topic my-topic \
  --group my-group \
  --bootstrap-server localhost:9092

# With key
kafka-console-consumer --topic my-topic \
  --property print.key=true \
  --property key.separator=":" \
  --bootstrap-server localhost:9092
```

---

## Consumer Group Management

```bash
# List groups
kafka-consumer-groups --list --bootstrap-server localhost:9092

# Describe group (check lag)
kafka-consumer-groups --describe --group my-group --bootstrap-server localhost:9092

# Reset offsets to beginning
kafka-consumer-groups --reset-offsets --to-earliest \
  --group my-group --topic my-topic \
  --execute --bootstrap-server localhost:9092

# Reset to specific offset
kafka-consumer-groups --reset-offsets --to-offset 100 \
  --group my-group --topic my-topic:0 \
  --execute --bootstrap-server localhost:9092
```

---

## Configuration

### Producer Config (Python)

```python
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    acks='all',                    # 0, 1, or 'all'
    retries=3,
    max_in_flight_requests_per_connection=1,
    compression_type='snappy',     # snappy, gzip, lz4, zstd
    batch_size=16384,              # 16KB
    linger_ms=10,                  # Wait 10ms to batch
    buffer_memory=33554432,        # 32MB
    enable_idempotence=True
)
```

### Consumer Config (Python)

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'my-topic',
    bootstrap_servers='localhost:9092',
    group_id='my-group',
    auto_offset_reset='earliest',  # 'earliest' or 'latest'
    enable_auto_commit=True,
    auto_commit_interval_ms=5000,
    max_poll_records=500,
    max_poll_interval_ms=300000,   # 5 minutes
    session_timeout_ms=10000,
    heartbeat_interval_ms=3000
)
```

---

## Security

### SSL Config

```python
producer = KafkaProducer(
    bootstrap_servers='broker:9093',
    security_protocol='SSL',
    ssl_cafile='/path/to/ca-cert',
    ssl_certfile='/path/to/client-cert',
    ssl_keyfile='/path/to/client-key'
)
```

### SASL Config

```python
producer = KafkaProducer(
    bootstrap_servers='broker:9093',
    security_protocol='SASL_SSL',
    sasl_mechanism='PLAIN',
    sasl_plain_username='user',
    sasl_plain_password='password'
)
```

---

## Monitoring

```bash
# Check broker logs
docker logs kafka-broker

# JMX metrics
kafka-run-class kafka.tools.JmxTool \
  --object-name kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec

# Get offsets
kafka-run-class kafka.tools.GetOffsetShell \
  --broker-list localhost:9092 \
  --topic my-topic
```

---

## Common Patterns

### At-Least-Once Processing

```python
for message in consumer:
    try:
        process(message)
        consumer.commit()
    except Exception as e:
        log_error(e)
        # Message will be reprocessed
```

### Exactly-Once (Transactions)

```python
producer.init_transactions()
producer.begin_transaction()
try:
    producer.send('topic', message)
    producer.commit_transaction()
except:
    producer.abort_transaction()
```

---

## Docker Compose

```yaml
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

---

## Download

[Download PDF Cheat Sheet →](#)

---

<div class="result" markdown>

!!! tip "More Resources"
    Check out **[Tools & Links](tools.md)** and **[FAQ](faq.md)**

</div>
