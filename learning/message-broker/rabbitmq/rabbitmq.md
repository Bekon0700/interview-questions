---
title: RabbitMQ
category: message-broker
topic: rabbitmq
tags: [learning, message-broker, rabbitmq]
related: ["[[../message-broker]]", "[[../kafka/kafka]]"]
---

> [!abstract] Overview
> **RabbitMQ** is a traditional **message broker** built around the classic queue model: producers publish messages to an **exchange**, the exchange routes them to one or more **queues** based on rules, and consumers pull messages off a queue — once a message is consumed and acknowledged, it's **removed**. Where Kafka is a durable, replayable log built for high-throughput streaming and fan-out, RabbitMQ is built for flexible, reliable **task distribution and routing**.

## Backstory

RabbitMQ was released in 2007 by Rabbit Technologies (later acquired by SpringSource/VMware/Pivotal), implementing the **AMQP** (Advanced Message Queuing Protocol) standard — an open protocol designed so messaging systems from different vendors could interoperate. It predates Kafka by several years and comes from the classic enterprise messaging lineage (similar in spirit to IBM MQ, JMS brokers), focused on **reliable delivery of discrete units of work** rather than event streaming.

Its core abstraction — exchange + queue + routing — gives it much richer, more flexible routing logic out of the box than Kafka's simpler "topic + partition" model, at the cost of the durable-log properties (replay, multiple independent consumer groups reading full history) that make Kafka suited to streaming.

## What it is

- **Producers** publish messages to an **exchange**, never directly to a queue.
- An **exchange** routes messages to **queues** based on its type: `direct` (exact routing-key match), `topic` (pattern match, e.g. `orders.*`), `fanout` (broadcast to all bound queues), `headers` (match on message headers).
- A **queue** holds messages until a consumer pulls them; once a consumer **acknowledges** a message, RabbitMQ deletes it from the queue.
- Multiple consumers can subscribe to the *same* queue, and RabbitMQ **round-robins** deliveries between them (competing consumers) — this is how RabbitMQ distributes work, as opposed to Kafka's partition-per-consumer assignment.

## Why it exists

Applications often need to hand off discrete units of work — "send this email," "resize this image," "charge this card" — to be processed reliably, exactly once (or with clear at-least-once semantics), with retry and dead-lettering if something fails, and with flexible routing (this message goes to queue A if it matches pattern X, or gets broadcast to every subscriber). RabbitMQ's exchange/queue model and mature delivery guarantees (acknowledgments, publisher confirms, dead-letter exchanges, priority queues, TTLs) directly serve exactly this kind of task-queue use case, which Kafka handles more awkwardly.

## How it works (high level)

1. A producer connects and publishes a message to an **exchange**, with a **routing key**.
2. The exchange consults its **bindings** (rules linking it to queues) and routes the message to zero, one, or more matching queues.
3. A queue holds the message in memory/on disk (depending on durability settings) until delivered.
4. A consumer subscribes to the queue; RabbitMQ pushes (or the consumer pulls) messages to it.
5. The consumer processes the message and sends an **ack**; only then does RabbitMQ remove it from the queue. If the consumer disconnects without acking, the message is requeued for redelivery.
6. Failed/unprocessable messages can be routed to a **dead-letter exchange** for inspection instead of being silently dropped or retried forever.
