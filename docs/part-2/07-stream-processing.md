# Chapter 7: Stream Processing that Matters

## 🎯 Learning Objectives

- Kafka Streams fundamentals
- KStream vs KTable
- Windowing operations
- Joins and aggregations

---

## 🌊 Kafka Streams Basics

Kafka Streams = Library for building stream processing apps on Kafka.

```java
StreamsBuilder builder = new StreamsBuilder();
KStream<String, Order> orders = builder.stream("orders");

// Filter high-value orders
KStream<String, Order> highValue = orders.filter(
    (key, order) -> order.getAmount() > 1000
);

highValue.to("high-value-orders");
```

---

## 📊 KStream vs KTable

| KStream | KTable |
|---------|--------|
| Event log | Changelog (state) |
| Insert-only | Upserts (latest value) |
| All events matter | Current state matters |

---

## ⏰ Windowing

Group events by time windows for aggregations.

```java
KStream<String, Order> orders = builder.stream("orders");

// Tumbling window: non-overlapping 1-hour windows
KTable<Windowed<String>, Long> ordersPerHour = orders
    .groupByKey()
    .windowedBy(TimeWindows.of(Duration.ofHours(1)))
    .count();
```

### Window Types

- **Tumbling:** Fixed, non-overlapping (9-10am, 10-11am)
- **Hopping:** Fixed, overlapping (9-10am, 9:30-10:30am)
- **Sliding:** Event-driven windows
- **Session:** Activity-based gaps

---

## 🎯 Mini Project #4: Fraud Detection Pipeline

**Goal:** Detect suspicious patterns in transactions

*[Complete fraud detection example with windowing and alerts]*

---

<div class="result" markdown>

!!! success "Next"
    Learn **[Kafka Connect](08-kafka-connect.md)** →

</div>
