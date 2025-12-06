# Chapter 12: Running Kafka in Production

## Deployment Options

Choose the right deployment model for your needs.

---

## 1. Managed Services (Easiest)

### Confluent Cloud

**Pros:**
- Fully managed (no ops overhead)
- Auto-scaling
- Built-in monitoring
- Security by default

**Cons:**
- Cost ($$$$)
- Vendor lock-in

**Best For:** Teams wanting zero operational overhead

---

### AWS MSK (Managed Streaming for Kafka)

**Pros:**
- AWS integration
- Managed infrastructure
- Reasonable pricing

**Cons:**
- AWS-specific
- Limited version upgrades

**Best For:** AWS-native architectures

---

## 2. Self-Managed on VMs

### Architecture

```
Load Balancer
    ↓
┌─────────────┬─────────────┬─────────────┐
│  Broker 1   │  Broker 2   │  Broker 3   │
│  (Leader)   │  (Follower) │  (Follower) │
└─────────────┴─────────────┴─────────────┘
     ↓              ↓              ↓
┌─────────────┬─────────────┬─────────────┐
│ ZooKeeper 1 │ ZooKeeper 2 │ ZooKeeper 3 │
└─────────────┴─────────────┴─────────────┘
```

**Pros:**
- Full control
- Cost-effective at scale
- No vendor lock-in

**Cons:**
- You manage everything
- Requires expertise

---

## 3. Kubernetes Deployment

### Using Strimzi Operator

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
spec:
  kafka:
    version: 3.5.0
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
    storage:
      type: persistent-claim
      size: 100Gi
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 10Gi
```

**Pros:**
- Cloud-native
- Auto-healing
- Scalable

**Cons:**
- Kubernetes complexity
- Storage performance considerations

---

## Capacity Planning

### Calculator

```python
# Throughput requirements
messages_per_second = 10000
avg_message_size_kb = 1
retention_days = 7
replication_factor = 3

# Calculate disk needed
daily_data_gb = (messages_per_second * avg_message_size_kb * 86400) / 1024 / 1024
total_disk_gb = daily_data_gb * retention_days * replication_factor

print(f"Required disk space: {total_disk_gb:.2f} GB")
```

### Sizing Guidelines

| Workload | Brokers | vCPUs/Broker | RAM/Broker | Disk/Broker |
|----------|---------|--------------|------------|-------------|
| **Small** | 3 | 4 | 8 GB | 500 GB |
| **Medium** | 5 | 8 | 16 GB | 1 TB |
| **Large** | 7+ | 16+ | 32 GB+ | 2+ TB SSD | ---

## Production Configuration

### Broker Config (`server.properties`)

```properties
# Broker basics
broker.id=1
log.dirs=/var/lib/kafka/data

# Networking
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://broker1.example.com:9092

# Replication
default.replication.factor=3
min.insync.replicas=2
unclean.leader.election.enable=false

# Performance
num.network.threads=8
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400

# Retention
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000

# Compression
compression.type=producer
```

---

## Deployment Checklist

!!! success "Pre-Production"
    - [ ] Minimum 3 brokers for HA
    - [ ] `replication.factor >= 3`
    - [ ] `min.insync.replicas = 2`
    - [ ] Monitoring configured (Prometheus + Grafana)
    - [ ] Alerts set up (lag, disk, CPU)
    - [ ] Backup strategy defined
    - [ ] Disaster recovery plan documented

---

<div class="result" markdown>

!!! success "Next"
    Set up **[Monitoring & Troubleshooting](13-monitoring.md)** →

</div>
