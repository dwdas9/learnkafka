# Chapter 5: Designing Events & Topics for Real Systems

## Learning Objectives

- Event schema design principles
- Topic naming conventions
- Partitioning strategies
- Retention policies

---

## Event Schema Design

### Anatomy of a Good Event

```json
{
  "event_id": "uuid-here",
  "event_type": "order.created",
  "event_version": "v1",
  "timestamp": "2025-12-06T10:30:00Z",
  "source": "order-service",
  "data": {
    "order_id": "ORD-123",
    "user_id": "USER-456",
    "amount": 99.99,
    "items": [...]
  },
  "metadata": {
    "correlation_id": "trace-id",
    "causation_id": "parent-event-id"
  }
}
```

!!! tip "Best Practices"
    - Include event metadata (ID, type, version, timestamp)
    - Use ISO 8601 for timestamps
    - Include source/origin information
    - Add correlation/causation IDs for tracing

---

## Topic Naming Conventions

```
<domain>.<entity>.<event-type>

Examples:
- orders.order.created
- orders.order.updated
- payments.transaction.completed
- inventory.product.stock-changed
```

---

## Choosing Partition Keys

| Use Case | Key Strategy | Example |
|----------|--------------|---------|
| Per-user ordering | `user_id` | All events for user-123 → same partition |
| Per-order ordering | `order_id` | Order lifecycle events stay together |
| Load balancing | `null` | Round-robin distribution |
| Geography | `region` | Events by region | ---

## Real Example: E-Commerce Order Topic

*[Content to be expanded with detailed design example]*

---

<div class="result" markdown>

!!! success "Next"
    Learn **[Schema Management](06-schema-management.md)** →

</div>
