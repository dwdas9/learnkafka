# Chapter 10: Anti-Patterns to Avoid

## Common Kafka Mistakes

Learn from others' mistakes — avoid these common pitfalls.

---

## 1. Too Many Partitions

**Problem:** Creating topics with 100+ partitions "just in case"

**Why Bad:**
- Increases broker memory/CPU
- Slower leader elections
- Rebalancing takes forever

!!! danger "Don't Do This"
    ```bash
    # Creating 1000 partitions for a small topic
    kafka-topics --create --topic logs \
      --partitions 1000  # Overkill!
    ```

!!! success "Do This Instead"
    ```bash
    # Start small, scale as needed
    kafka-topics --create --topic logs \
      --partitions 3  # Reasonable start
    ```

**Rule of Thumb:** Start with 3-6 partitions, scale based on throughput needs

---

## 2. One Topic Per Entity Instance

**Problem:** Creating `user-123-events`, `user-456-events`, etc.

**Why Bad:**
- Topic explosion (metadata overhead)
- Hard to query/aggregate
- Management nightmare

!!! success "Do This Instead"
    Use **one topic with partition keys**
    ```python
    # All users in 'user-events', keyed by user_id
    producer.send('user-events', key='user-123', value=event)
    ```

---

## 3. Schema Chaos

**Problem:** No schema governance, everyone sends random JSON

**Results:**
- Field name inconsistencies (`userId` vs `user_id`)
- Type mismatches (string vs int)
- Breaking changes without notice

!!! success "Solution"
    - Use Schema Registry
    - Enforce compatibility rules
    - Version your schemas

---

## 4. Shared State in Consumers

**Problem:** Multiple consumer instances sharing mutable state

```python
# BAD: Shared global state
total_revenue = 0  # Shared across instances!

for message in consumer:
    total_revenue += message.value['amount']
```

**Why Bad:** Consumer group rebalancing causes data loss/corruption

!!! success "Do This Instead"
    - Use Kafka Streams state stores
    - Use external databases
    - Make consumers stateless

---

## 5. Ignoring Consumer Lag

**Problem:** Not monitoring how far behind consumers are

**Consequences:**
- Delayed processing
- Disk fills up (retention policy)
- Business impact

!!! success "Solution"
    ```bash
    # Monitor lag
    kafka-consumer-groups --bootstrap-server localhost:9092 \
      --describe --group my-group
    ```

---

## 6. Using Kafka as a Database

**Problem:** Treating Kafka as primary storage

**Why Bad:**
- Not designed for random access queries
- No indexes, joins, or ACID transactions
- Retention policies may delete data

!!! success "Use Kafka For"
    - Event streaming
    - Message passing
    - Temporary buffering

!!! success "Use Databases For"
    - Querying by arbitrary fields
    - Long-term storage
    - ACID transactions

---

## 7. Large Messages

**Problem:** Sending multi-MB messages

**Consequences:**
- Network congestion
- Memory pressure
- Slow consumers

!!! success "Alternatives"
    1. **Claim Check Pattern:** Store payload in S3, send reference
    2. **Chunking:** Split large payloads
    3. **Compression:** Enable producer compression

---

## 8. No Error Handling

**Problem:** Assuming messages always process successfully

```python
# BAD: No error handling
for message in consumer:
    process(message.value)  # What if this fails?
```

!!! success "Robust Pattern"
    ```python
    for message in consumer:
        try:
            process(message.value)
            consumer.commit()
        except RecoverableError as e:
            # Retry logic
            retry_with_backoff(message)
        except FatalError as e:
            # Send to DLQ
            send_to_dlq(message, error=e)
            consumer.commit()  # Don't reprocess
    ```

---

## Anti-Pattern Checklist

Before going to production, verify:

- [ ] Partition count is reasonable (< 50 per topic)
- [ ] Using partition keys properly (not creating topic per entity)
- [ ] Schema governance in place
- [ ] Consumers are stateless or use Kafka Streams
- [ ] Monitoring consumer lag
- [ ] Kafka is not the primary database
- [ ] Message sizes < 1MB
- [ ] Error handling and DLQ implemented

---

<div class="result" markdown>

!!! success "Next"
    Study **[Real-World Architectures](11-real-architectures.md)** →

</div>
