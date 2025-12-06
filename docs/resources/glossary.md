# Glossary

## 📖 Kafka Terminology

Quick reference for Kafka concepts and terms.

---

## Core Concepts

**Apache Kafka**
: Distributed event streaming platform for high-throughput, fault-tolerant data pipelines.

**Event**
: A record of something that happened (e.g., order placed, user clicked). Also called a message or record.

**Topic**
: A category or feed name to which events are published. Like a database table or folder.

**Partition**
: A subdivision of a topic that enables parallelism. Each partition is an ordered, immutable sequence of events.

**Offset**
: A unique sequential ID assigned to each event within a partition. Used to track position.

**Broker**
: A Kafka server that stores data and serves client requests.

**Cluster**
: A group of Kafka brokers working together.

---

## Producers & Consumers

**Producer**
: An application that publishes (writes) events to Kafka topics.

**Consumer**
: An application that subscribes to (reads) events from Kafka topics.

**Consumer Group**
: A group of consumers sharing the same `group.id` that coordinate to consume a topic's partitions.

**Rebalancing**
: Process where partitions are redistributed among consumers in a group (e.g., when a consumer joins/leaves).

---

## Replication & Durability

**Replication Factor**
: Number of copies of each partition. E.g., replication factor of 3 = 1 leader + 2 followers.

**Leader**
: The broker replica that handles all reads and writes for a partition.

**Follower (Replica)**
: A broker replica that replicates the leader and can become leader if needed.

**ISR (In-Sync Replicas)**
: Set of replicas that are fully caught up with the leader.

**Acks (Acknowledgments)**
: Producer setting controlling write durability:
- `acks=0`: Fire and forget
- `acks=1`: Leader acknowledges
- `acks=all`: All ISRs acknowledge

---

## Processing Guarantees

**At-Least-Once**
: Each message is processed at least once (may have duplicates on failure).

**At-Most-Once**
: Each message is processed at most once (may lose messages on failure).

**Exactly-Once**
: Each message is processed exactly once (no duplicates, no loss). Requires transactions.

**Idempotence**
: Property where processing the same message multiple times has the same effect as once.

---

## Advanced

**Kafka Streams**
: Java library for building stream processing applications on Kafka.

**KStream**
: Kafka Streams abstraction for an event stream (all events matter).

**KTable**
: Kafka Streams abstraction for a changelog table (latest value per key).

**KSQL/ksqlDB**
: SQL-like interface for stream processing on Kafka.

**Kafka Connect**
: Framework for streaming data between Kafka and external systems (databases, S3, etc.).

**Source Connector**
: Kafka Connect plugin that imports data from external system → Kafka.

**Sink Connector**
: Kafka Connect plugin that exports data from Kafka → external system.

---

## Schema & Serialization

**Schema Registry**
: Central repository for managing event schemas with versioning.

**Avro**
: Binary serialization format with schema evolution support.

**Protobuf**
: Google's binary serialization format.

**Serializer**
: Converts objects to bytes for sending to Kafka.

**Deserializer**
: Converts bytes from Kafka back to objects.

---

## Operations

**ZooKeeper**
: Coordination service used by Kafka (< 3.3) for metadata management. Being replaced by KRaft.

**KRaft (Kafka Raft)**
: New consensus protocol replacing ZooKeeper (Kafka 3.3+).

**Consumer Lag**
: Difference between latest offset in partition and consumer's current offset.

**Retention**
: How long Kafka keeps messages before deletion (time or size-based).

**Compaction**
: Log cleanup policy that keeps only the latest value per key.

**Tiered Storage**
: Feature to offload older data to cheaper storage (S3, Azure Blob).

---

## Patterns

**Event Sourcing**
: Storing application state as a sequence of events.

**CQRS (Command Query Responsibility Segregation)**
: Separate models for reading and writing data.

**CDC (Change Data Capture)**
: Tracking database changes as events (e.g., using Debezium).

**Saga**
: Pattern for distributed transactions across microservices.

**Dead Letter Queue (DLQ)**
: Topic for storing messages that failed processing.

**Claim Check**
: Pattern where large payloads are stored externally, and only reference is sent.

---

## Performance

**Batching**
: Grouping multiple messages together for efficiency.

**Compression**
: Reducing message size (snappy, gzip, lz4, zstd).

**Throughput**
: Number of messages processed per unit of time.

**Latency**
: Time delay from producer send to consumer receive.

**Back Pressure**
: Mechanism to slow down producers when consumers can't keep up.

---

## Security

**SSL/TLS**
: Encryption protocol for securing data in transit.

**SASL (Simple Authentication and Security Layer)**
: Framework for authentication (PLAIN, SCRAM, GSSAPI).

**ACL (Access Control List)**
: Permissions defining who can read/write topics.

**mTLS (Mutual TLS)**
: Both client and server authenticate each other.

---

<div class="result" markdown>

!!! tip "Learn More"
    Explore **[Cheat Sheets](cheat-sheets.md)** and **[Tools](tools.md)**

</div>
