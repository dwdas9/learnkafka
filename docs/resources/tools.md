# Tools & Links

## 🛠️ Essential Kafka Tools

---

## 🖥️ GUI Tools

### Kafka UI (Recommended)

Free, open-source web UI for managing Kafka clusters.

**Features:**
- Browse topics and messages
- Consumer group monitoring
- Schema registry integration

```bash
docker run -p 8080:8080 \
  -e KAFKA_CLUSTERS_0_NAME=local \
  -e KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=localhost:9092 \
  provectuslabs/kafka-ui
```

[Website →](https://github.com/provectus/kafka-ui)

---

### Confluent Control Center

Enterprise-grade monitoring and management.

**Features:**
- Full cluster visibility
- Stream monitoring
- Data flow visualization

[Website →](https://www.confluent.io/confluent-control-center/)

---

### Offset Explorer (Kafka Tool)

Desktop GUI for Kafka management.

[Download →](https://www.kafkatool.com/)

---

## 📊 Monitoring

### Prometheus + Grafana

```yaml
# docker-compose.yml addition
kafka-exporter:
  image: danielqsj/kafka-exporter
  command: --kafka.server=kafka:9092

prometheus:
  image: prom/prometheus
  volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml

grafana:
  image: grafana/grafana
  ports:
    - "3000:3000"
```

**Pre-built Dashboards:**
- Kafka Overview (ID: 7589)
- Kafka Exporter (ID: 7589)

---

### Burrow

LinkedIn's consumer lag monitoring.

[GitHub →](https://github.com/linkedin/Burrow)

---

## 🔌 Kafka Connect

### Connector Hub

Browse 200+ connectors.

[Confluent Hub →](https://www.confluent.io/hub/)

**Popular Connectors:**
- JDBC Source/Sink
- S3 Sink
- Elasticsearch Sink
- MongoDB Source/Sink
- Debezium (CDC)

---

## 📚 Documentation

| Resource | Link |
|----------|------|
| **Apache Kafka Docs** | [kafka.apache.org](https://kafka.apache.org/documentation/) |
| **Confluent Docs** | [docs.confluent.io](https://docs.confluent.io/) |
| **Kafka Improvement Proposals (KIPs)** | [KIP List](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Improvement+Proposals) |

---

## 🎓 Learning Resources

### Online Courses

- [Kafka Fundamentals (Confluent)](https://developer.confluent.io/)
- [Kafka Streams (Udemy)](https://www.udemy.com/topic/kafka/)

### Books

- **"Kafka: The Definitive Guide"** by Neha Narkhede
- **"Kafka Streams in Action"** by William P. Bejeck Jr.
- **"Designing Event-Driven Systems"** by Ben Stopford

### Videos

- [Kafka Summit Talks](https://www.kafka-summit.org/)
- [Confluent YouTube](https://www.youtube.com/c/Confluent)

---

## 💻 Client Libraries

| Language | Library | Link |
|----------|---------|------|
| **Java** | Apache Kafka | [Maven](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients) |
| **Python** | kafka-python | [PyPI](https://pypi.org/project/kafka-python/) |
| **Go** | Sarama | [GitHub](https://github.com/Shopify/sarama) |
| **Node.js** | KafkaJS | [npm](https://www.npmjs.com/package/kafkajs) |
| **C#** | Confluent.Kafka | [NuGet](https://www.nuget.org/packages/Confluent.Kafka) |
| **Rust** | rust-rdkafka | [crates.io](https://crates.io/crates/rdkafka) |

---

## ☁️ Managed Services

| Provider | Service | Link |
|----------|---------|------|
| **Confluent** | Confluent Cloud | [confluent.cloud](https://www.confluent.io/confluent-cloud/) |
| **AWS** | Amazon MSK | [aws.amazon.com/msk](https://aws.amazon.com/msk/) |
| **Azure** | Azure Event Hubs | [azure.microsoft.com/event-hubs](https://azure.microsoft.com/en-us/services/event-hubs/) |
| **Google Cloud** | N/A | Use self-managed on GKE |
| **Aiven** | Aiven for Kafka | [aiven.io/kafka](https://aiven.io/kafka) |

---

## 🐳 Docker Images

```bash
# Official Kafka
docker pull apache/kafka:latest

# Confluent Platform
docker pull confluentinc/cp-kafka:7.5.0

# Bitnami (easier for development)
docker pull bitnami/kafka:latest
```

---

## 🌐 Community

- [Kafka Users Mailing List](https://lists.apache.org/list.html?users@kafka.apache.org)
- [Stack Overflow [apache-kafka]](https://stackoverflow.com/questions/tagged/apache-kafka)
- [Confluent Community Slack](https://launchpass.com/confluentcommunity)
- [r/apachekafka](https://www.reddit.com/r/apachekafka/)

---

<div class="result" markdown>

!!! tip "Need Help?"
    Check the **[FAQ](faq.md)** or join the community!

</div>
