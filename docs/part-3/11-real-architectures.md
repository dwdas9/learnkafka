# Chapter 11: Real-World Architectures

## 🏗️ Production System Designs

Study battle-tested architectures from real companies.

---

## 💳 1. Payment Processing System

```mermaid
graph TB
    A[API Gateway] -->|payment.requested| K[Kafka]
    K --> B[Fraud Detection]
    K --> C[Payment Processor]
    K --> D[Notification Service]
    C -->|payment.completed| K
    K --> E[Accounting]
    K --> F[Analytics]
```

**Key Patterns:**
- Event-driven orchestration
- Idempotent processing (duplicate payments)
- Dead letter queues for failed payments

**Topics:**
- `payments.requested`
- `payments.validated`
- `payments.completed`
- `payments.failed`

---

## 🚗 2. Ride-Hailing Event Platform

```mermaid
graph LR
    subgraph Producers
        D[Driver App]
        R[Rider App]
        M[Matching Service]
    end
    
    subgraph Kafka
        T1[ride.requested]
        T2[ride.matched]
        T3[ride.completed]
    end
    
    subgraph Consumers
        A[Analytics]
        B[Billing]
        C[Ratings]
        N[Notifications]
    end
    
    D --> T2
    R --> T1
    M --> T2
    T1 --> M
    T2 --> N
    T3 --> A
    T3 --> B
    T3 --> C
```

**Scale Requirements:**
- 1M+ rides/day
- Sub-second matching latency
- Geographic partitioning

**Topics Strategy:**
- Partition by city/region (geo-locality)
- Separate topics for different stages

---

## 📊 3. Clickstream Analytics Platform

```mermaid
graph TB
    W[Website/Mobile] -->|clicks, views| K1[Kafka: raw-events]
    K1 --> S[Kafka Streams: Enrichment]
    S -->|enriched-events| K2[Kafka]
    K2 --> ES[Elasticsearch]
    K2 --> DW[Data Warehouse]
    K2 --> RT[Real-time Dashboard]
```

**Data Flow:**
1. **Ingest:** Millions of click events/sec
2. **Enrich:** Add user metadata, session data
3. **Route:** Different sinks for different use cases

**Partitioning:**
- By `user_id` (session analysis)
- By `session_id` (event ordering)

---

## 🏭 4. IoT Data Ingestion Pipeline

```mermaid
graph TB
    subgraph Edge Devices
        S1[Sensor 1]
        S2[Sensor 2]
        S3[Sensor N]
    end
    
    subgraph Kafka
        T[telemetry-data<br/>100 partitions]
    end
    
    subgraph Processing
        KS[Kafka Streams<br/>Anomaly Detection]
        SP[Spark<br/>Batch Analytics]
    end
    
    subgraph Storage
        TS[Time-Series DB]
        DL[Data Lake]
    end
    
    S1 & S2 & S3 --> T
    T --> KS
    T --> SP
    KS --> TS
    SP --> DL
```

**Challenges:**
- High volume (10K+ devices × 1 msg/sec)
- Out-of-order delivery
- Device failures

**Solutions:**
- Windowed aggregations
- Compression
- Tiered storage

---

## 🏦 5. Data Lake Ingestion

```mermaid
graph LR
    subgraph Sources
        DB1[(MySQL)]
        DB2[(Postgres)]
        API[External APIs]
    end
    
    subgraph Kafka
        CDC[CDC Topics<br/>Debezium]
        API_T[API Events]
    end
    
    subgraph Sinks
        S3[S3/MinIO<br/>Parquet]
        SF[Snowflake]
        EL[Elasticsearch]
    end
    
    DB1 -->|CDC| CDC
    DB2 -->|CDC| CDC
    API --> API_T
    CDC --> S3
    CDC --> SF
    API_T --> EL
```

**CDC (Change Data Capture) Pattern:**
- Captures every database change as event
- No impact on source databases
- Enables real-time data warehouse

---

## 📈 Architecture Decision Guide

| Requirement | Pattern | Example |
|-------------|---------|---------|
| **High throughput** | Many partitions, compression | Clickstream |
| **Strict ordering** | Single partition or keyed | Payments |
| **Geographic distribution** | Multi-cluster replication | Ride-hailing |
| **Complex processing** | Kafka Streams | Fraud detection |
| **Integration-heavy** | Kafka Connect | Data lake |

---

## 🎓 Key Takeaways

!!! tip "Lessons from Production"
    1. **Start simple, scale as needed**
    2. **Monitor everything** (lag, throughput, errors)
    3. **Design for failure** (retries, DLQs, idempotence)
    4. **Use the right tool** (Kafka for streaming, DB for storage)
    5. **Test at scale** (chaos engineering)

---

<div class="result" markdown>

!!! success "Part III Complete!"
    Learn **[Production Deployment](../part-4/index.md)** next →

</div>
