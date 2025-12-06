# Frequently Asked Questions

## Common Kafka Questions

---

## Getting Started

### Q: Do I need ZooKeeper?

**A:** Depends on Kafka version:

- **Kafka < 3.3:** Yes, ZooKeeper required
- **Kafka 3.3+:** Can use KRaft mode (ZooKeeper-free)
- **Kafka 4.0+:** ZooKeeper deprecated

**Recommendation:** Use ZooKeeper for now (more stable), migrate to KRaft later.

---

### Q: How many partitions should I create?

**A:** Start small, scale as needed:

- **Small workload:** 3-6 partitions
- **Medium:** 10-20 partitions
- **Large:** 30-50+ partitions

**Rule:** Don't exceed 50 partitions/topic unless necessary.

**Why:** Each partition adds broker overhead.

---

### Q: What's a good message size?

**A:** 
- **Ideal:** < 1 MB
- **Max:** 1 MB (default broker limit)
- **Larger:** Use claim-check pattern (store in S3, send reference)

---

## Architecture

### Q: Kafka vs RabbitMQ?

| Feature | Kafka | RabbitMQ |
|---------|-------|----------|
| **Use Case** | Event streaming, high throughput | Task queues, complex routing |
| **Replay** | Yes | No |
| **Throughput** | Very high (100K+ msg/s) | Moderate (10K msg/s) |
| **Ordering** | Per partition | Per queue |
| **Best For** | Event logs, analytics | Work distribution | **TLDR:** Use Kafka for event streaming, RabbitMQ for task queues.

---

### Q: Can I use Kafka as a database?

**A:**  **No.** Kafka is not a database.

**Use Kafka for:**
- Streaming data
- Event logs
- Temporary buffering

**Use Database for:**
- ACID transactions
- Complex queries (JOIN, WHERE)
- Long-term storage with indexes

**Pattern:** Store events in Kafka, persist state in database.

---

## Operations

### Q: How do I monitor consumer lag?

```bash
kafka-consumer-groups --bootstrap-server localhost:9092 \
  --describe --group my-group
```

**Alert thresholds:**
-  Warning: Lag > 10,000 messages
-  Critical: Lag > 100,000 or increasing consistently

---

### Q: My consumer is slow. How to scale?

**Options:**

1. **Add more consumer instances** (up to # of partitions)
2. **Increase partitions** (requires rebalancing)
3. **Optimize processing** (batch, parallel)
4. **Tune `max.poll.records`** (fetch more messages)

---

### Q: What's the retention limit?

**A:** Configurable per topic:

```bash
# Default: 7 days
log.retention.hours=168

# Unlimited (use with tiered storage)
log.retention.hours=-1
```

**Disk space = throughput × retention period × replication factor**

---

## Performance

### Q: How to improve producer throughput?

**Top optimizations:**

1. **Batching:**
   ```python
   linger_ms=10  # Wait 10ms to batch
   batch_size=16384  # 16KB batches
   ```

2. **Compression:**
   ```python
   compression_type='snappy'
   ```

3. **Async sending:**
   ```python
   producer.send('topic', msg)  # Don't wait for .get()
   ```

4. **Reduce acks:**
   ```python
   acks=1  # Only leader (faster than 'all')
   ```

---

### Q: At-least-once vs exactly-once?

| Guarantee | Behavior | Use When |
|-----------|----------|----------|
| **At-least-once** | May process duplicates | Most cases (default) |
| **Exactly-once** | No duplicates | Financial transactions | **At-least-once (default):**
```python
for msg in consumer:
    process(msg)
    consumer.commit()  # May reprocess on failure
```

**Exactly-once (transactions):**
```python
producer.init_transactions()
producer.begin_transaction()
producer.send('topic', msg)
producer.commit_transaction()
```

**Trade-off:** Exactly-once has higher latency.

---

## Troubleshooting

### Q: "Not enough replicas" error

**Cause:** `min.insync.replicas` > available brokers

**Solution:**
```bash
# Reduce min ISR
kafka-configs --alter --topic my-topic \
  --add-config min.insync.replicas=1 \
  --bootstrap-server localhost:9092
```

**Or:** Add more brokers.

---

### Q: Consumer rebalancing constantly

**Causes:**
- Processing too slow
- `max.poll.interval.ms` too low
- Network issues

**Solution:**
```python
consumer = KafkaConsumer(
    max_poll_interval_ms=600000,  # 10 minutes
    session_timeout_ms=45000,
    heartbeat_interval_ms=15000
)
```

---

### Q: Out of disk space

**Solutions:**

1. **Reduce retention:**
   ```bash
   kafka-configs --alter --topic my-topic \
     --add-config retention.ms=86400000  # 1 day
   ```

2. **Enable compression**
3. **Add more disks**
4. **Use tiered storage** (Kafka 3.6+)

---

## Security

### Q: How to enable SSL?

```python
producer = KafkaProducer(
    bootstrap_servers='broker:9093',
    security_protocol='SSL',
    ssl_cafile='/path/to/ca-cert',
    ssl_certfile='/path/to/client-cert',
    ssl_keyfile='/path/to/client-key'
)
```

[Full SSL setup →](../part-4/14-security.md)

---

### Q: What's the difference between SSL and SASL?

- **SSL (TLS):** Encryption (secure data in transit)
- **SASL:** Authentication (verify identity)

**Best practice:** Use both (`SASL_SSL`)

---

## Advanced

### Q: When to use Kafka Streams vs Flink?

| Feature | Kafka Streams | Flink |
|---------|---------------|-------|
| **Setup** | Library (no cluster) | Requires cluster |
| **Use Case** | Kafka-native pipelines | Complex CEP |
| **Learning Curve** | Easy | Steep | **Use Kafka Streams** for simple transformations/aggregations.

**Use Flink** for complex multi-source processing.

---

### Q: Can I change partition count later?

**Yes, but:**
- Can only **increase** (not decrease)
- Breaks keyed ordering (messages redistribute)

```bash
kafka-topics --alter --topic my-topic --partitions 10
```

 **Warning:** Existing keyed messages won't move partitions.

---

## Still Have Questions?

<div class="result" markdown>

!!! tip "Get Help"
    - [Kafka Users Mailing List](https://lists.apache.org/list.html?users@kafka.apache.org)
    - [Stack Overflow](https://stackoverflow.com/questions/tagged/apache-kafka)
    - [Confluent Community Slack](https://launchpass.com/confluentcommunity)

</div>
