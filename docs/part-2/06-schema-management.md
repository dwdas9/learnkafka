# Chapter 6: Schema Management for Real Teams

## Learning Objectives

- Schema formats (Avro, Protobuf, JSON Schema)
- Schema Registry usage
- Schema evolution strategies

---

## Schema Formats Comparison

| Format | Pros | Cons | Best For |
|--------|------|------|----------|
| **Avro** | Compact, built-in evolution | Binary | Data pipelines |
| **Protobuf** | Fast, language support | Learning curve | Microservices |
| **JSON Schema** | Human-readable | Larger size | APIs, debugging | ---

## Schema Registry

Central repository for managing schemas with versioning.

```python
from confluent_kafka import avro
from confluent_kafka.avro import AvroProducer

value_schema = avro.loads('''
{
  "type": "record",
  "name": "Order",
  "fields": [
    {"name": "order_id", "type": "string"},
    {"name": "amount", "type": "float"}
  ]
}
''')

producer = AvroProducer({
    'bootstrap.servers': 'localhost:9092',
    'schema.registry.url': 'http://localhost:8081'
}, default_value_schema=value_schema)
```

---

## Schema Evolution

### Compatible Changes 

- Adding optional fields
- Adding fields with defaults
- Removing fields with defaults

### Incompatible Changes 

- Removing required fields
- Changing field types
- Renaming fields (without aliases)

---

## Mini Project #3: Versioned Events

*[Hands-on project with schema evolution]*

---

<div class="result" markdown>

!!! success "Next"
    Explore **[Stream Processing](07-stream-processing.md)** →

</div>
