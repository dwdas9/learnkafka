# Chapter 4: Programming with Kafka

## 🎯 Write Your First Producer & Consumer

Learn to interact with Kafka using code (Java and Python examples provided).

---

## 🐍 Python Setup

```bash
pip install kafka-python
```

---

## 📤 Building a Producer

### Basic Producer (Python)

```python
from kafka import KafkaProducer
import json
import time

# Create producer
producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

# Send messages
for i in range(10):
    event = {
        'order_id': f'ORD-{i}',
        'amount': 100 + i * 10,
        'timestamp': time.time()
    }
    
    # Send to 'orders' topic
    future = producer.send('orders', value=event)
    
    # Wait for send to complete
    record_metadata = future.get(timeout=10)
    
    print(f"Sent to partition {record_metadata.partition} "
          f"at offset {record_metadata.offset}")

producer.flush()
producer.close()
```

### With Key (for Partitioning)

```python
producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    key_serializer=lambda k: k.encode('utf-8'),
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

# Messages with same key go to same partition
producer.send('orders', key='user-123', value={'order': 'data'})
```

---

## 📥 Building a Consumer

### Basic Consumer (Python)

```python
from kafka import KafkaConsumer
import json

consumer = KafkaConsumer(
    'orders',
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest',  # Start from beginning
    group_id='order-processors',
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

print("Waiting for messages...")

for message in consumer:
    order = message.value
    print(f"Received: Order {order['order_id']}, "
          f"Amount: ${order['amount']}")
    
    # Process order here
    # ...

consumer.close()
```

### With Manual Commit

```python
consumer = KafkaConsumer(
    'orders',
    bootstrap_servers='localhost:9092',
    group_id='order-processors',
    enable_auto_commit=False,  # Manual commit
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

for message in consumer:
    try:
        # Process message
        process_order(message.value)
        
        # Commit offset manually
        consumer.commit()
    except Exception as e:
        print(f"Error: {e}")
        # Don't commit on error (will reprocess)
```

---

## ⚙️ Producer Configuration

### Acknowledgments (acks)

```python
producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    acks='all',  # 0, 1, or 'all'
    retries=3,
    max_in_flight_requests_per_connection=1  # Ordering guarantee
)
```

| acks | Behavior | Use Case |
|------|----------|----------|
| `0` | Fire and forget | Metrics, logs (OK to lose) |
| `1` | Leader acknowledges | Balanced (default) |
| `all` | All replicas acknowledge | Critical data |

---

## 🔄 Idempotent Producer

Ensures exactly-once semantics (no duplicates even with retries).

```python
producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    enable_idempotence=True,  # Prevents duplicates
    acks='all',
    retries=3
)
```

---

## 🎯 Mini Project #2: Order Events System

**Goal:** Build a publisher that sends order events, and an analytics consumer that processes them.

**Architecture:**

```
Order Service (Producer)
        ↓
    orders topic
        ↓
Analytics Service (Consumer)
```

### Requirements

1. **Producer:** Generate 100 random orders
2. **Consumer:** Calculate total revenue
3. **Consumer:** Find average order value
4. **Consumer:** Count orders per user

**Starter Code:**

=== "Producer (order_producer.py)"
    ```python
    from kafka import KafkaProducer
    import json
    import random
    import time
    
    producer = KafkaProducer(
        bootstrap_servers='localhost:9092',
        value_serializer=lambda v: json.dumps(v).encode('utf-8')
    )
    
    users = ['user-1', 'user-2', 'user-3', 'user-4', 'user-5']
    
    for i in range(100):
        order = {
            'order_id': f'ORD-{i}',
            'user_id': random.choice(users),
            'amount': round(random.uniform(10, 500), 2),
            'timestamp': time.time()
        }
        
        producer.send('orders', value=order)
        print(f"Sent: {order}")
        time.sleep(0.1)
    
    producer.flush()
    producer.close()
    ```

=== "Consumer (analytics_consumer.py)"
    ```python
    from kafka import KafkaConsumer
    import json
    
    consumer = KafkaConsumer(
        'orders',
        bootstrap_servers='localhost:9092',
        auto_offset_reset='earliest',
        group_id='analytics',
        value_deserializer=lambda m: json.loads(m.decode('utf-8'))
    )
    
    total_revenue = 0
    order_count = 0
    user_orders = {}
    
    for message in consumer:
        order = message.value
        
        # Calculate metrics
        total_revenue += order['amount']
        order_count += 1
        
        user_id = order['user_id']
        user_orders[user_id] = user_orders.get(user_id, 0) + 1
        
        # Print running totals
        avg_value = total_revenue / order_count
        print(f"Orders: {order_count}, "
              f"Revenue: ${total_revenue:.2f}, "
              f"Avg: ${avg_value:.2f}")
        
        if order_count >= 100:
            break
    
    print("\n=== Final Analytics ===")
    print(f"Total Revenue: ${total_revenue:.2f}")
    print(f"Average Order Value: ${total_revenue/order_count:.2f}")
    print(f"Orders per User: {user_orders}")
    
    consumer.close()
    ```

**Run It:**

```bash
# Terminal 1: Start consumer
python analytics_consumer.py

# Terminal 2: Start producer
python order_producer.py
```

---

## ☕ Java Examples

=== "Producer (Java)"
    ```java
    import org.apache.kafka.clients.producer.*;
    import java.util.Properties;
    
    public class OrderProducer {
        public static void main(String[] args) {
            Properties props = new Properties();
            props.put("bootstrap.servers", "localhost:9092");
            props.put("key.serializer", 
                "org.apache.kafka.common.serialization.StringSerializer");
            props.put("value.serializer", 
                "org.apache.kafka.common.serialization.StringSerializer");
            props.put("acks", "all");
            
            Producer<String, String> producer = new KafkaProducer<>(props);
            
            for (int i = 0; i < 100; i++) {
                String order = String.format(
                    "{\"order_id\": \"ORD-%d\", \"amount\": %.2f}",
                    i, 100 + i * 10.0
                );
                
                ProducerRecord<String, String> record = 
                    new ProducerRecord<>("orders", order);
                
                producer.send(record, (metadata, exception) -> {
                    if (exception != null) {
                        exception.printStackTrace();
                    } else {
                        System.out.printf("Sent to partition %d at offset %d%n",
                            metadata.partition(), metadata.offset());
                    }
                });
            }
            
            producer.close();
        }
    }
    ```

=== "Consumer (Java)"
    ```java
    import org.apache.kafka.clients.consumer.*;
    import java.time.Duration;
    import java.util.*;
    
    public class OrderConsumer {
        public static void main(String[] args) {
            Properties props = new Properties();
            props.put("bootstrap.servers", "localhost:9092");
            props.put("group.id", "order-processors");
            props.put("key.deserializer", 
                "org.apache.kafka.common.serialization.StringDeserializer");
            props.put("value.deserializer", 
                "org.apache.kafka.common.serialization.StringDeserializer");
            props.put("auto.offset.reset", "earliest");
            
            Consumer<String, String> consumer = new KafkaConsumer<>(props);
            consumer.subscribe(Collections.singletonList("orders"));
            
            while (true) {
                ConsumerRecords<String, String> records = 
                    consumer.poll(Duration.ofMillis(100));
                
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf("Received: %s%n", record.value());
                }
            }
        }
    }
    ```

---

## 🎓 Key Takeaways

!!! tip "Remember"
    - Use **acks='all'** for critical data
    - Enable **idempotence** to prevent duplicates
    - **Manual commit** for at-least-once processing
    - Use **keys** for partition control
    - Handle **exceptions** gracefully

---

<div class="result" markdown>

!!! success "Part I Complete!"
    You've mastered the basics. Ready for **[Part II: Build Applications](../part-2/index.md)** →

</div>
