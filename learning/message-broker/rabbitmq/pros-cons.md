---
title: RabbitMQ — Pros & Cons
category: message-broker
topic: rabbitmq
tags: [learning, message-broker, rabbitmq, pros-cons]
related: ["[[rabbitmq]]"]
---

> [!success] Pros
> - **Flexible, rich routing** — exchanges (direct/topic/fanout/headers) let you express complex delivery rules without application-side logic. Example: a `topic` exchange routes `orders.eu.cancelled` and `orders.us.cancelled` to different regional queues purely via binding patterns.
> - **Mature delivery guarantees per message** — per-message ack/nack, redelivery, and dead-lettering give fine-grained control over individual message outcomes. Example: a job that fails 3 times is automatically dead-lettered for manual review instead of looping forever.
> - **Natural fit for task/work queues** — competing consumers round-robin work automatically; no partitioning scheme to plan ahead of time. Example: scaling from 5 to 10 PDF-generation workers immediately increases throughput with zero reconfiguration.
> - **Lower operational footprint for moderate workloads** — simpler mental model and lighter resource needs than a Kafka cluster for small-to-medium throughput. Example: a small SaaS backend runs a single RabbitMQ node comfortably for its email/notification queues.
> - **Protocol flexibility** — supports AMQP, MQTT, and STOMP, useful when integrating IoT devices or heterogeneous clients. Example: IoT sensors publish via lightweight MQTT while backend services consume the same broker via AMQP.
> - **Priority and TTL built in** — urgent messages can jump the queue, and stale messages can expire automatically. Example: password-reset emails (priority 9) bypass a backlog of marketing emails (priority 1) in the same queue.

> [!warning] Cons
> - **No durable replay by default** — once a message is acked and removed, it's gone; there's no built-in way for a new consumer to "catch up" on history the way Kafka's log allows. Example: a new analytics service can't retroactively read the last week of order events — RabbitMQ never kept them.
> - **Lower raw throughput than Kafka** — per-message ack overhead and the queue-based model cap throughput well below what a partitioned Kafka log can sustain. Example: RabbitMQ is a poor fit for ingesting millions of clickstream events/sec; Kafka is the standard choice there.
> - **Harder to fan out to many independent consumer groups** — while fanout exchanges can broadcast, each additional independent "reader" typically needs its own queue bound to the exchange, and none of them get replay of messages published before they existed. Example: adding a 4th analytics pipeline months later can't retroactively see historical order events, unlike a new Kafka consumer group reading from the beginning.
> - **Manual scaling considerations for ordering** — RabbitMQ doesn't have a built-in partitioning concept, so preserving strict per-entity ordering across multiple competing consumers requires deliberate design (e.g. consistent-hash exchanges), whereas Kafka gets this from partition keys by default. Example: without extra setup, two consumers on the same queue can process two messages for the same user out of order.
> - **Clustering/HA setup has its own complexity** — quorum queues and cluster management require real operational understanding, similar in spirit to (if generally lighter than) Kafka's replication tuning. Example: undersized quorum queue replica counts can still leave a queue unavailable if too many nodes are lost at once.
> - **Broker becomes a routing decision point** — because routing logic lives in exchange/binding configuration rather than code, complex routing topologies can become hard to audit or reason about over time. Example: a sprawling set of topic bindings accumulated over years can make it unclear which queues actually receive a given message without inspecting the broker directly.
