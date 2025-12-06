# Chapter 14: Security Without Complexity

## Essential Kafka Security

Secure your Kafka cluster without over-engineering.

---

## Security Layers

```
┌─────────────────────────────────────┐
│   Authentication (who are you?)    │  ← SSL/SASL
├─────────────────────────────────────┤
│   Authorization (what can you do?) │  ← ACLs
├─────────────────────────────────────┤
│   Encryption (secure data)          │  ← SSL/TLS
└─────────────────────────────────────┘
```

---

## 1. SSL/TLS Encryption

Encrypt data in transit.

### Generate Certificates

```bash
# Create CA
openssl req -new -x509 -keyout ca-key -out ca-cert -days 365

# Create broker keystore
keytool -genkey -keystore kafka.server.keystore.jks \
  -validity 365 -storepass password -keypass password \
  -dname "CN=kafka-broker" -storetype pkcs12

# Sign certificate
keytool -certreq -keystore kafka.server.keystore.jks \
  -file cert-file -storepass password -keypass password

openssl x509 -req -CA ca-cert -CAkey ca-key \
  -in cert-file -out cert-signed -days 365
```

### Broker Config

```properties
# Enable SSL
listeners=SSL://:9093
advertised.listeners=SSL://broker1.example.com:9093

# SSL settings
ssl.keystore.location=/var/private/ssl/kafka.server.keystore.jks
ssl.keystore.password=password
ssl.key.password=password
ssl.truststore.location=/var/private/ssl/kafka.server.truststore.jks
ssl.truststore.password=password
```

### Producer/Consumer Config

```python
producer = KafkaProducer(
    bootstrap_servers='broker:9093',
    security_protocol='SSL',
    ssl_cafile='/path/to/ca-cert',
    ssl_certfile='/path/to/client-cert',
    ssl_keyfile='/path/to/client-key'
)
```

---

## 2. Authentication (SASL)

Verify user identity.

### SASL/PLAIN (Simple)

**Broker Config:**

```properties
listeners=SASL_SSL://:9093
security.inter.broker.protocol=SASL_SSL
sasl.mechanism.inter.broker.protocol=PLAIN
sasl.enabled.mechanisms=PLAIN
```

**JAAS Config (`kafka_server_jaas.conf`):**

```
KafkaServer {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="admin"
  password="admin-secret"
  user_admin="admin-secret"
  user_producer="producer-secret"
  user_consumer="consumer-secret";
};
```

**Client Config:**

```python
producer = KafkaProducer(
    bootstrap_servers='broker:9093',
    security_protocol='SASL_SSL',
    sasl_mechanism='PLAIN',
    sasl_plain_username='producer',
    sasl_plain_password='producer-secret'
)
```

---

### SASL/SCRAM (Recommended)

More secure than PLAIN (hashed passwords).

```bash
# Create user
kafka-configs --bootstrap-server localhost:9092 \
  --alter --add-config 'SCRAM-SHA-256=[password=secret]' \
  --entity-type users --entity-name alice
```

---

## 3. ACLs (Authorization)

Control who can do what.

### Grant Permissions

```bash
# Producer write access
kafka-acls --bootstrap-server localhost:9092 \
  --add --allow-principal User:producer \
  --operation Write --topic orders

# Consumer read access
kafka-acls --bootstrap-server localhost:9092 \
  --add --allow-principal User:consumer \
  --operation Read --topic orders \
  --group consumer-group-1

# Admin access
kafka-acls --bootstrap-server localhost:9092 \
  --add --allow-principal User:admin \
  --operation All --topic '*' --cluster
```

### List ACLs

```bash
kafka-acls --bootstrap-server localhost:9092 \
  --list --topic orders
```

---

## Practical Security Policies

### Microservices Pattern

Each service gets:
- **Own user** (e.g., `order-service`, `inventory-service`)
- **Minimal permissions** (principle of least privilege)

```bash
# Order Service
kafka-acls --add --allow-principal User:order-service \
  --operation Write --topic orders

# Inventory Service
kafka-acls --add --allow-principal User:inventory-service \
  --operation Read --topic orders \
  --group inventory-group
```

---

## Security Checklist

!!! success "Production Security"
    - [ ] SSL/TLS enabled for all connections
    - [ ] SASL authentication required
    - [ ] ACLs enforced (not `allow.everyone`)
    - [ ] Separate users per service
    - [ ] Audit logging enabled
    - [ ] Secrets in vault (not hardcoded)
    - [ ] Network segmentation (VPC/firewall)
    - [ ] ZooKeeper secured (if using)

---

## Quick Start: Secure Kafka with Docker

```yaml
version: '3.8'
services:
  kafka:
    image: confluentinc/cp-kafka:7.5.0
    environment:
      KAFKA_LISTENERS: SASL_SSL://:9093
      KAFKA_ADVERTISED_LISTENERS: SASL_SSL://kafka:9093
      KAFKA_SECURITY_INTER_BROKER_PROTOCOL: SASL_SSL
      KAFKA_SASL_ENABLED_MECHANISMS: PLAIN
      KAFKA_SSL_KEYSTORE_LOCATION: /etc/kafka/secrets/kafka.keystore.jks
      KAFKA_SSL_KEYSTORE_PASSWORD: password
      KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/kafka_jaas.conf"
    volumes:
      - ./secrets:/etc/kafka/secrets
      - ./kafka_jaas.conf:/etc/kafka/kafka_jaas.conf
```

---

<div class="result" markdown>

!!! success "Next"
    Learn **[Scaling & Optimization](15-scaling.md)** →

</div>
