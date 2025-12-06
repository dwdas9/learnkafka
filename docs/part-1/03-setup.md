# Chapter 3: Setup in 10 Minutes

## Quick Start with Docker

Get Kafka running locally in minutes using Docker Compose.

---

## Prerequisites

- Docker Desktop installed
- 8GB RAM available
- Terminal/Command line access

---

## Step 1: Docker Compose Setup

Create `docker-compose.yml`:

```yaml
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
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

**Start Kafka:**

```bash
docker-compose up -d
```

---

## Step 2: Verify Installation

```bash
# Check running containers
docker ps

# Should see zookeeper and kafka running
```

---

## Step 3: Create Your First Topic

```bash
# Create a topic named "test"
docker exec -it <kafka-container-id> kafka-topics --create \
  --topic test \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1

# List topics
docker exec -it <kafka-container-id> kafka-topics --list \
  --bootstrap-server localhost:9092
```

---

## Step 4: Send Messages (Producer)

```bash
# Start producer console
docker exec -it <kafka-container-id> kafka-console-producer \
  --topic test \
  --bootstrap-server localhost:9092

# Type messages (press Enter after each):
> Hello Kafka!
> This is my first message
> Event-driven architecture is cool
```

Press `Ctrl+C` to exit.

---

## Step 5: Consume Messages

Open a new terminal:

```bash
# Start consumer from beginning
docker exec -it <kafka-container-id> kafka-console-consumer \
  --topic test \
  --from-beginning \
  --bootstrap-server localhost:9092
```

You should see your messages!

---

## Mini Project #1: Simple Event Pipeline

**Goal:** Build a basic producer → Kafka → consumer flow

**Tasks:**

1. Create a topic called `events`
2. Produce 10 messages with different data
3. Consume them with a consumer group
4. Verify all messages received

**Time:** 15-30 minutes

??? example "Solution Commands"
    ```bash
    # Create topic
    kafka-topics --create --topic events \
      --bootstrap-server localhost:9092 \
      --partitions 3 --replication-factor 1
    
    # Produce messages
    for i in {1..10}; do
      echo "Event $i: User action at $(date)" | \
      kafka-console-producer --topic events \
        --bootstrap-server localhost:9092
    done
    
    # Consume
    kafka-console-consumer --topic events \
      --from-beginning \
      --bootstrap-server localhost:9092 \
      --group my-group
    ```

---

## Troubleshooting

!!! warning "Common Issues"
    **Port already in use:**
    ```bash
    # Check what's using port 9092
    lsof -i :9092  # Mac/Linux
    netstat -ano | findstr :9092  # Windows
    ```
    
    **Kafka won't start:**
    ```bash
    # Check logs
    docker logs <kafka-container-id>
    ```
    
    **Can't connect:**
    - Ensure `KAFKA_ADVERTISED_LISTENERS` matches your setup
    - Check firewall settings

---

<div class="result" markdown>

!!! success "Next Step"
    Kafka is running! Now let's **[write code](04-programming.md)** →

</div>
