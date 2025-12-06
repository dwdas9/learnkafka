# Chapter 13: Monitoring & Troubleshooting

## Metrics That Matter

Focus on the metrics that actually indicate problems.

---

## Critical Metrics

### 1. Consumer Lag

**Most important metric** — how far behind are consumers?

```bash
# Check lag
kafka-consumer-groups --bootstrap-server localhost:9092 \
  --describe --group my-group

# Output:
# TOPIC    PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
# orders   0          1000           1500            500   Behind!
# orders   1          2000           2000            0     Caught up
```

!!! danger "When to Alert"
    - Lag > 10,000 messages
    - Lag increasing consistently
    - Lag > 5 minutes (time-based)

---

### 2. Broker Metrics

| Metric | Warning Threshold | Critical |
|--------|-------------------|----------|
| **CPU Usage** | > 70% | > 90% |
| **Disk Usage** | > 70% | > 85% |
| **Network I/O** | Near saturation | Saturated |
| **Under-Replicated Partitions** | > 0 | > 10 | ---

### 3. Producer Metrics

```python
# Monitor producer metrics
metrics = producer.metrics()

# Key metrics:
# - record-send-rate
# - record-error-rate
# - request-latency-avg
# - buffer-available-bytes
```

---

## Monitoring Stack

### Prometheus + Grafana

```yaml
# docker-compose.yml
services:
  kafka-exporter:
    image: danielqsj/kafka-exporter
    ports:
      - "9308:9308"
    command:
      - '--kafka.server=kafka:9092'
  
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
  
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
```

**Popular Dashboards:**
- Kafka Overview (ID: 7589)
- Kafka Exporter (ID: 7589)

---

## Debugging Consumer Lag

### Lag Investigation Steps

1. **Check if consumers are running**
   ```bash
   kafka-consumer-groups --bootstrap-server localhost:9092 \
     --describe --group my-group --members
   ```

2. **Check consumer logs for errors**
   ```bash
   docker logs my-consumer | grep ERROR
   ```

3. **Measure consumer processing time**
   ```python
   import time
   
   for message in consumer:
       start = time.time()
       process(message)
       duration = time.time() - start
       print(f"Processed in {duration:.3f}s")
   ```

4. **Scale consumers if needed**
   ```bash
   # Add more consumer instances
   docker-compose up --scale consumer=5
   ```

---

## Common Issues & Solutions

### Issue 1: Slow Consumer

**Symptoms:** Lag keeps increasing

**Solutions:**

=== "Scale Horizontally"
    Add more consumer instances (up to # of partitions)

=== "Optimize Processing"
    ```python
    # Batch processing
    batch = []
    for message in consumer:
        batch.append(message)
        if len(batch) >= 100:
            process_batch(batch)
            consumer.commit()
            batch = []
    ```

=== "Increase Partitions"
    ```bash
    kafka-topics --alter --topic orders \
      --partitions 10 \
      --bootstrap-server localhost:9092
    ```

---

### Issue 2: Out of Disk Space

**Symptoms:** Broker crashes, errors writing to log

**Solutions:**

=== "Reduce Retention"
    ```bash
    kafka-configs --alter \
      --entity-type topics \
      --entity-name orders \
      --add-config retention.ms=86400000 \  # 1 day
      --bootstrap-server localhost:9092
    ```

=== "Enable Compression"
    ```python
    producer = KafkaProducer(
        compression_type='gzip'  # or 'snappy', 'lz4'
    )
    ```

=== "Add More Disks"
    Mount additional volumes to `log.dirs`

---

### Issue 3: Rebalancing Storm

**Symptoms:** Constant rebalancing, slow processing

**Root Causes:**
- `max.poll.interval.ms` too low
- Consumer processing too slow
- Network issues

**Solution:**
```python
consumer = KafkaConsumer(
    'orders',
    max_poll_interval_ms=600000,  # 10 minutes
    session_timeout_ms=45000,     # 45 seconds
    heartbeat_interval_ms=15000   # 15 seconds
)
```

---

## At-Least-Once vs Exactly-Once

### At-Least-Once (Default)

```python
for message in consumer:
    process(message)
    consumer.commit()  # After processing
```

**Guarantees:** No message loss  
**Risk:** Duplicates on failure

---

### Exactly-Once (Transactions)

```python
producer = KafkaProducer(
    transactional_id='my-transactional-id',
    enable_idempotence=True
)

producer.init_transactions()
producer.begin_transaction()

try:
    producer.send('topic', message)
    producer.commit_transaction()
except:
    producer.abort_transaction()
```

**Guarantees:** No duplicates  
**Trade-off:** Higher latency, complexity

---

## Monitoring Checklist

!!! success "Essential Monitoring"
    - [ ] Consumer lag alerts
    - [ ] Broker disk/CPU/memory alerts
    - [ ] Under-replicated partitions alert
    - [ ] Producer error rate alert
    - [ ] Grafana dashboards set up
    - [ ] Log aggregation (ELK, Splunk)
    - [ ] Runbooks for common issues

---

<div class="result" markdown>

!!! success "Next"
    Implement **[Security](14-security.md)** →

</div>
