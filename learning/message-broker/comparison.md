---
title: Message Broker — Comparison
category: message-broker
topic: comparison
tags: [learning, message-broker, comparison]
related: ["[[message-broker]]", "[[kafka/kafka]]", "[[rabbitmq/rabbitmq]]"]
---

Feature-by-feature comparison of the Topics in this Category.

| Dimension | Kafka | RabbitMQ |
|---|---|---|
| **Core model** | Durable, partitioned append-only log | Exchange-routed queues |
| **Message lifecycle** | Retained per policy (time/size/compaction); not deleted on read | Deleted once acknowledged by a consumer |
| **Replay** | Native — reset consumer offset, reread history | Not supported by default — once acked, it's gone |
| **Ordering guarantee** | Per-partition (use a key to keep related messages ordered) | Per-queue, generally FIFO, but not guaranteed across competing consumers without extra design |
| **Fan-out to independent readers** | Native — any number of consumer groups can independently read the full topic, including from before they existed | Possible via fanout exchange, but new readers get only messages published after their queue exists |
| **Work distribution model** | Partition assigned to one consumer per group | Round-robin "competing consumers" on a shared queue |
| **Routing flexibility** | Simple: topic + optional partition key | Rich: direct/topic/fanout/headers exchanges with binding patterns |
| **Throughput** | Very high (millions of records/sec via partitioned, sequential disk I/O) | Moderate — sufficient for most task-queue workloads, well below Kafka's ceiling |
| **Delivery guarantees** | At-least-once by default; exactly-once achievable via idempotent/transactional producers | At-least-once via manual ack/nack; per-message redelivery control |
| **Priority messages** | Not supported natively | Native priority queues |
| **Dead-lettering** | Not built-in — handled at the application/consumer level | Native dead-letter exchanges |
| **TTL on individual messages** | Retention is topic-wide, not per-message | Native per-message and per-queue TTL |
| **Coordination layer** | KRaft (built-in Raft controller, replaces ZooKeeper in modern versions) | Built-in clustering; quorum queues use Raft for replication |
| **Typical use case** | Event streaming, activity logs, CDC pipelines, analytics, systems needing replay | Task queues, RPC-style request/reply, complex routing, background job processing |
| **Operational complexity** | Higher — partition planning, broker sizing, retention tuning | Lower for moderate scale — simpler mental model, smaller footprint |

## When to choose which

- **Choose Kafka** when you need to retain and replay history, fan out the same stream to many independent downstream systems, or sustain very high throughput (e.g. clickstream ingestion, event sourcing, CDC from a database).
- **Choose RabbitMQ** when you need reliable, flexible routing of discrete tasks with per-message control (priority, TTL, dead-lettering) and don't need consumers to replay history (e.g. background job processing, email/notification dispatch, RPC-style request/reply between services).
- **A rough test:** if a brand-new consumer added six months from now should be able to "catch up" on everything that happened before it existed, that's a strong signal for Kafka. If every message just needs to be reliably handled once and then forgotten, RabbitMQ is usually the simpler, sufficient choice.
