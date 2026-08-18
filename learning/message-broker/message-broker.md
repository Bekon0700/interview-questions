---
title: Message Broker — Category
topic: message-broker
tags: [learning, category, message-broker]
related: []
---

> [!abstract] What this Category is
> A **message broker** is a system that lets independent services communicate by passing messages through an intermediary, instead of calling each other directly. Producers publish messages without knowing who (if anyone) is listening; consumers read messages without knowing who produced them. This decoupling is what lets services scale, fail, and deploy independently.

## History — why this category was necessary

Before message brokers, services that needed to talk to each other did it through direct calls (HTTP/RPC) or by sharing a database. Both approaches create **tight coupling**: if the receiving service is down, slow, or overloaded, the caller is affected too. As systems grew into many independent services (especially with the shift to microservices in the 2000s–2010s), teams needed a way to let producers and consumers operate on their own schedules.

Early messaging systems (IBM MQ, JMS-based brokers, later RabbitMQ) solved this for **task queues** — one message, one consumer, processed once. Kafka (created at LinkedIn, 2011) came from a different angle: it modeled messaging as a **durable, replayable log** rather than a queue that empties as it's consumed, which unlocked event streaming, multiple independent consumers reading the same data, and reprocessing history — not just point-to-point delivery.

## What problem this category solves

- **Decoupling** — producers and consumers don't need to know about each other or be online at the same time.
- **Buffering / backpressure** — a slow consumer doesn't block a fast producer; the broker absorbs the difference.
- **Fan-out** — one message can reach many independent consumers.
- **Reliability** — messages can be persisted so a crashed consumer doesn't lose work.
- **Asynchrony** — long-running work (emails, video processing, analytics) can happen off the request path.

## Alternative names / adjacent terms

- **Message queue** — often used interchangeably, though strictly a queue implies point-to-point, consume-once delivery (e.g. traditional RabbitMQ/SQS usage).
- **Event streaming platform** — the term Kafka itself prefers, emphasizing durable, replayable logs of events over transient queued tasks.
- **Pub/sub system** — emphasizes the publish/subscribe delivery model shared by most brokers in this category.

## Topics in this Category

- [[kafka/kafka|Kafka]] — distributed, durable event-streaming log; append-only, replayable, built for high-throughput fan-out.
- [[rabbitmq/rabbitmq|RabbitMQ]] — classic queue broker with flexible exchange-based routing; built for reliable task distribution, not replay.

See [[comparison]] for a feature-by-feature breakdown, and [[recall]] for comparison-style review prompts.
