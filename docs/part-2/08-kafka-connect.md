# Chapter 8: Kafka Connect — Real Integrations

## 🎯 Learning Objectives

- Kafka Connect fundamentals
- Source vs Sink connectors
- Real integrations (DB, S3, Elasticsearch)

---

## 🔌 What is Kafka Connect?

Framework for streaming data between Kafka and external systems **without writing code**.

```
Database → Source Connector → Kafka → Sink Connector → Data Lake
```

---

## 📥 Source Connectors

Pull data **into** Kafka from external systems.

### JDBC Source (Database → Kafka)

```json
{
  "name": "mysql-source",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "connection.url": "jdbc:mysql://localhost:3306/mydb",
    "mode": "incrementing",
    "incrementing.column.name": "id",
    "topic.prefix": "mysql-",
    "table.whitelist": "orders,customers"
  }
}
```

---

## 📤 Sink Connectors

Push data **from** Kafka to external systems.

### S3 Sink (Kafka → S3)

```json
{
  "name": "s3-sink",
  "config": {
    "connector.class": "io.confluent.connect.s3.S3SinkConnector",
    "topics": "orders",
    "s3.bucket.name": "my-data-lake",
    "s3.region": "us-east-1",
    "format.class": "io.confluent.connect.s3.format.parquet.ParquetFormat",
    "flush.size": "1000"
  }
}
```

---

## 🚀 Popular Connectors

| Connector | Type | Use Case |
|-----------|------|----------|
| JDBC | Source/Sink | Database sync |
| S3 | Sink | Data lake |
| Elasticsearch | Sink | Search indexing |
| MongoDB | Source/Sink | NoSQL integration |
| Debezium | Source | CDC (Change Data Capture) |

---

## 🛠️ Running Connect

```bash
# Start Kafka Connect
docker run -d \
  --name kafka-connect \
  -p 8083:8083 \
  confluentinc/cp-kafka-connect:7.5.0

# Deploy a connector
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d @connector-config.json
```

---

## 🎯 Real Example: DB → Kafka → Elasticsearch

*[Complete ETL pipeline example]*

---

<div class="result" markdown>

!!! success "Part II Complete!"
    Master **[Architecture Patterns](../part-3/index.md)** next →

</div>
