# Chapter 15: Scaling & Optimization

## Scale Kafka for Production Workloads

Tune only what matters for performance.

---

## Performance Tuning Priorities

1. **Partitions** (biggest impact)
2. **Batching** (producer/consumer)
3. **Compression**
4. **Hardware** (disk, network)

---

## 1. Partition Strategy

### Scaling Throughput

```
1 partition  →  100 MB/s
3 partitions →  300 MB/s
10 partitions → 1 GB/s
```

**Rule:** More partitions = more parallelism

**But:** Diminishing returns after ~50 partitions/topic

---

### When to Add Partitions

```bash
# Check current load
kafka-run-class kafka.tools.GetOffsetShell \
  --broker-list localhost:9092 \
  --topic orders

# Add partitions if needed
kafka-topics --alter --topic orders \
  --partitions 10 \
  --bootstrap-server localhost:9092
```

---

## 2. Producer Optimization

### Batching (Biggest Win)

```python
producer = KafkaProducer(
    linger_ms=10,           # Wait 10ms to batch messages
    batch_size=16384,       # 16KB batches
    buffer_memory=33554432  # 32MB buffer
)
```

**Effect:** 10-100x throughput increase

---

### Compression

```python
producer = KafkaProducer(
    compression_type='snappy'  # or 'gzip', 'lz4', 'zstd'
)
```

| Compression | Speed | Ratio | CPU |
|-------------|-------|-------|-----|
| **snappy** | | | Low |
| **lz4** | | | Low |
| **gzip** | | | High |
| **zstd** | | | Medium | **Recommendation:** `snappy` or `lz4` for most cases

---

### Idempotence & Acks

```python
# For critical data
producer = KafkaProducer(
    enable_idempotence=True,  # Exactly-once
    acks='all',               # Wait for all replicas
    retries=3
)

# For high-throughput
producer = KafkaProducer(
    acks=1,  # Only leader acknowledges
    retries=0
)
```

---

## 3. Consumer Optimization

### Batching

```python
consumer = KafkaConsumer(
    max_poll_records=500,        # Fetch 500 messages at once
    fetch_min_bytes=1024,        # Wait for 1KB
    fetch_max_wait_ms=500        # Or wait 500ms
)

# Process in batches
batch = []
for message in consumer:
    batch.append(message)
    if len(batch) >= 100:
        process_batch(batch)
        consumer.commit()
        batch = []
```

---

### Parallel Processing

```python
from concurrent.futures import ThreadPoolExecutor

executor = ThreadPoolExecutor(max_workers=10)

def process_message(msg):
    # Heavy processing
    time.sleep(0.1)
    return result

for message in consumer:
    executor.submit(process_message, message)
```

---

## 4. Broker Tuning

### Disk Performance

```properties
# Use XFS or ext4 filesystem
# Mount with noatime
# Use RAID 10 for production

# Broker config
num.io.threads=16              # More threads for disk I/O
log.flush.interval.messages=10000
log.flush.interval.ms=1000
```

---

### Network Tuning

```properties
# Increase network buffer
socket.send.buffer.bytes=1048576    # 1MB
socket.receive.buffer.bytes=1048576  # 1MB
socket.request.max.bytes=104857600   # 100MB max request

# More network threads
num.network.threads=8
```

---

## 5. Avoiding Rebalance Pain

### Problem: Rebalancing Slows Everything

**Causes:**
- Consumer processing too slow
- Heartbeat timeout
- New consumers joining

### Solution: Tune Timeouts

```python
consumer = KafkaConsumer(
    max_poll_interval_ms=600000,  # 10 minutes
    session_timeout_ms=45000,     # 45 seconds
    heartbeat_interval_ms=15000,  # 15 seconds (1/3 of session timeout)
    max_poll_records=500          # Don't fetch too many
)
```

---

### Static Group Membership

Prevents rebalancing on consumer restart.

```python
consumer = KafkaConsumer(
    group_id='my-group',
    group_instance_id='consumer-1'  # Static ID
)
```

---

## 6. Tiered Storage (Kafka 3.6+)

Offload old data to cheaper storage (S3/Azure Blob).

```properties
# Enable tiered storage
remote.log.storage.system.enable=true
remote.log.manager.task.interval.ms=30000

# Retention: Keep 1 day local, 7 days remote
log.retention.ms=86400000
log.local.retention.ms=86400000
```

**Benefits:**
- 90% disk cost reduction
- Infinite retention
- Fast broker startup

---

## Performance Benchmarking

### Test Producer Throughput

```bash
kafka-producer-perf-test \
  --topic test \
  --num-records 1000000 \
  --record-size 1000 \
  --throughput -1 \
  --producer-props bootstrap.servers=localhost:9092 \
    acks=1 \
    compression.type=snappy
```

### Test Consumer Throughput

```bash
kafka-consumer-perf-test \
  --topic test \
  --messages 1000000 \
  --bootstrap-server localhost:9092
```

---

## Scaling Checklist

!!! success "Before Scaling"
    - [ ] Identify bottleneck (producer, consumer, broker, disk)
    - [ ] Enable compression
    - [ ] Tune batching parameters
    - [ ] Add partitions if needed
    - [ ] Scale consumers horizontally
    - [ ] Monitor metrics after changes

---

## Scale Milestones

| Throughput | Brokers | Partitions/Topic | Notes |
|------------|---------|------------------|-------|
| **< 10 MB/s** | 3 | 3-6 | Small deployment |
| **10-100 MB/s** | 5-7 | 10-20 | Medium scale |
| **100 MB/s - 1 GB/s** | 10+ | 30-50 | Large scale |
| **> 1 GB/s** | 20+ | 100+ | Enterprise | ---

<div class="result" markdown>

!!! success "Part IV Complete!"
    Build **[Real Projects](../part-5/index.md)** next →

</div>
