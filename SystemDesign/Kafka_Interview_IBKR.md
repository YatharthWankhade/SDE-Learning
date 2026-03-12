# Apache Kafka – In-Depth Interview Notes (IBKR JD)

## Table of Contents
1. [Kafka Architecture](#1-kafka-architecture)
2. [Core Concepts](#2-core-concepts)
3. [Producers](#3-producers)
4. [Consumers & Consumer Groups](#4-consumers--consumer-groups)
5. [Topics, Partitions & Replication](#5-topics-partitions--replication)
6. [Kafka with Java](#6-kafka-with-java)
7. [Scalability & Performance](#7-scalability--performance)
8. [Kafka Connect & Streams (overview)](#8-kafka-connect--streams-overview)
9. [Common Interview Questions](#9-common-interview-questions)

---

## 1. Kafka Architecture

```
Producers ──→ [ Broker 1 | Broker 2 | Broker 3 ] ──→ Consumers
              └──── Zookeeper / KRaft (coordination) ────┘

Broker = Kafka server/node
Cluster = multiple brokers
Topic = named stream of events
Partition = ordered, immutable log (unit of parallelism)
Offset = position of message in partition (monotonically increasing)
```

### Why Kafka for IBKR?
- **High throughput** – millions of market ticks/sec
- **Durability** – messages persisted to disk with replication
- **Replay** – consumers can re-read historical messages (playback)
- **Decoupling** – trade systems, risk systems, reporting all independently consume
- **Scalability** – add brokers/partitions without downtime

---

## 2. Core Concepts

### Message Structure
```
Key   → determines partition assignment (null = round-robin)
Value → the actual payload (JSON, Avro, Protobuf)
Headers → optional key-value metadata
Timestamp → event time or ingestion time
Offset  → position within partition
```

### Retention
```bash
# Retain messages for 7 days (default)
log.retention.hours=168

# Retain by size
log.retention.bytes=10737418240   # 10 GB per partition

# Compact topics (keep latest value per key – great for reference data!)
log.cleanup.policy=compact
```

> **IBKR Use Case:** Reference data (instrument master) → compacted topics so consumers always get latest

---

## 3. Producers

### Producer Config
```java
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "broker1:9092,broker2:9092");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,   StringSerializer.class);
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

// Reliability settings
props.put(ProducerConfig.ACKS_CONFIG, "all");        // wait for all ISR replicas
props.put(ProducerConfig.RETRIES_CONFIG, 3);
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);  // exactly-once

// Performance settings
props.put(ProducerConfig.LINGER_MS_CONFIG, 5);       // batch for 5ms
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);  // 16KB batch
props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
```

### Sending Messages
```java
// Sync send (blocks until acknowledged)
ProducerRecord<String, String> record =
    new ProducerRecord<>("market-data", "AAPL", "{\"price\":175.50,\"ts\":\"2024-03-10\"}");
RecordMetadata meta = producer.send(record).get();
System.out.printf("Sent to partition %d @ offset %d%n", meta.partition(), meta.offset());

// Async send with callback (preferred for throughput)
producer.send(record, (metadata, exception) -> {
    if (exception != null) {
        log.error("Failed to send: {}", exception.getMessage());
    } else {
        log.debug("Sent to {}:{}", metadata.partition(), metadata.offset());
    }
});

producer.flush();   // flush all pending messages
producer.close();   // close connections
```

### ACKs Explained
| `acks` | Durability | Latency | Risk |
|---|---|---|---|
| `0` | None – fire and forget | Lowest | Data loss possible |
| `1` | Leader only | Medium | Loss if leader crashes before replication |
| `all` (-1) | All ISR replicas | Highest | Safe, no data loss |

---

## 4. Consumers & Consumer Groups

```java
Properties props = new Properties();
props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "broker1:9092");
props.put(ConsumerConfig.GROUP_ID_CONFIG, "trade-processor-group");
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,   StringDeserializer.class);
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest"); // start from beginning

// Manual commit for exactly-once processing
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(List.of("market-data", "trade-events"));

try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        for (ConsumerRecord<String, String> record : records) {
            processMessage(record.key(), record.value());
        }
        consumer.commitSync(); // commit after processing batch
    }
} finally {
    consumer.close();
}
```

### Consumer Group Rebalancing
```
Topic: market-data (4 partitions: P0, P1, P2, P3)

Consumer Group A (2 consumers):
  Consumer-1 → P0, P1
  Consumer-2 → P2, P3

Add Consumer-3 → rebalance:
  Consumer-1 → P0, P1
  Consumer-2 → P2
  Consumer-3 → P3

Note: Max parallelism = number of partitions
      If consumers > partitions, some consumers idle
```

### Offset Management
```java
// Seek to specific offset (replay)
consumer.assign(List.of(new TopicPartition("market-data", 0)));
consumer.seek(new TopicPartition("market-data", 0), 1000); // start from offset 1000

// Seek to beginning
consumer.seekToBeginning(consumer.assignment());

// Get current lag
Map<TopicPartition, Long> endOffsets = consumer.endOffsets(consumer.assignment());
// lag = endOffset - committedOffset
```

---

## 5. Topics, Partitions & Replication

```bash
# Create topic
kafka-topics.sh --create \
  --bootstrap-server broker1:9092 \
  --topic market-data \
  --partitions 12 \
  --replication-factor 3

# Describe topic
kafka-topics.sh --describe --bootstrap-server broker1:9092 --topic market-data

# List topics
kafka-topics.sh --list --bootstrap-server broker1:9092

# Console producer/consumer for debugging
kafka-console-producer.sh --bootstrap-server broker1:9092 --topic test
kafka-console-consumer.sh --bootstrap-server broker1:9092 --topic test --from-beginning

# Check consumer group lag
kafka-consumer-groups.sh --bootstrap-server broker1:9092 \
  --describe --group trade-processor-group
```

### Replication
```
ISR = In-Sync Replicas (replicas caught up with leader)
min.insync.replicas=2  → require at least 2 replicas in-sync before ack

Leader partition handles all reads and writes
Follower partitions replicate from leader
If leader fails → new leader elected from ISR
```

---

## 6. Kafka with Java

### Custom Serializer (Avro – production standard)
```java
// Using Confluent Schema Registry with Avro
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, KafkaAvroSerializer.class);
props.put("schema.registry.url", "http://schema-registry:8081");

// Trade schema (Avro)
// {
//   "type": "record", "name": "Trade",
//   "fields": [
//     {"name": "symbol", "type": "string"},
//     {"name": "price",  "type": "double"},
//     {"name": "ts",     "type": "long"}
//   ]
// }
```

### Error Handling & Dead Letter Queue
```java
try {
    processMessage(record);
    consumer.commitSync();
} catch (ProcessingException e) {
    // Send to DLQ (dead letter queue)
    producer.send(new ProducerRecord<>("market-data-dlq", record.key(), record.value()));
    log.error("Failed processing, sent to DLQ: {}", record.offset());
    consumer.commitSync(); // still commit to avoid re-processing
}
```

---

## 7. Scalability & Performance

### Partition Count Strategy
```
Rule of thumb:
- Target throughput / (throughput per partition)
- For IBKR market data: 1M msgs/sec ÷ 100K/partition → 10 partitions minimum
- Add headroom: 2x → 20 partitions
- Consider consumer parallelism

Key-based partitioning → same symbol always goes to same partition (ordering guarantee)
```

### Performance Tuning
```java
// Producer tuning for high throughput
props.put("linger.ms", 20);              // batch longer
props.put("batch.size", 65536);          // 64KB batch
props.put("buffer.memory", 67108864);    // 64MB buffer
props.put("compression.type", "lz4");   // fast compression

// Consumer tuning
props.put("fetch.min.bytes", 50000);     // wait for 50KB before returning
props.put("max.poll.records", 500);      // 500 records per poll
```

---

## 8. Kafka Connect & Streams (overview)

```bash
# Kafka Connect: move data in/out of Kafka without code
# JDBC Source Connector: Oracle → Kafka
# JDBC Sink Connector: Kafka → Oracle

# Example: Oracle CDC to Kafka using Debezium
# Reads Oracle redo logs, publishes changes as Kafka events (great for IBKR ref data sync)
```

```java
// Kafka Streams: stream processing in-process
StreamsBuilder builder = new StreamsBuilder();
KStream<String, String> marketData = builder.stream("market-data");
KStream<String, Double> prices = marketData
    .mapValues(v -> JsonParser.parseDouble(v, "price"))
    .filter((symbol, price) -> price > 100.0);
prices.to("filtered-market-data", Produced.with(Serdes.String(), Serdes.Double()));
```

---

## 9. Common Interview Questions

| Question | Key Points |
|---|---|
| **What is Kafka?** | Distributed, fault-tolerant, high-throughput event streaming platform |
| **How does Kafka ensure ordering?** | Within a partition only; same key → same partition |
| **Exactly-once semantics?** | Idempotent producer + transactional API + consumer manual commit |
| **What happens if a consumer crashes?** | Rebalance; another consumer in group picks up unprocessed partitions |
| **Consumer lag?** | Difference between latest offset and committed offset; monitor with Grafana |
| **How to replay Kafka messages?** | Reset consumer group offset to earlier position: `kafka-consumer-groups.sh --reset-offsets` |
| **Kafka vs RabbitMQ?** | Kafka: log-based, high throughput, replay; RabbitMQ: queue-based, push model, lower latency |
| **What is log compaction?** | Keeps only latest message per key; deletes older duplicates; great for reference data |
| **What is a controller broker?** | Manages partition leader elections and broker membership in cluster |
| **What is min.insync.replicas?** | Min replicas that must acknowledge write before producer gets success response |
