---
title: RabbitMQ — All Features
category: message-broker
topic: rabbitmq
tags: [learning, message-broker, rabbitmq, features]
related: ["[[rabbitmq]]"]
---

Comprehensive feature list for RabbitMQ. Each feature includes why it was necessary and a concrete example.

## Exchanges (direct, topic, fanout, headers)

Producers never publish directly to a queue — they publish to an **exchange**, which routes the message to queues based on its type and bindings.

> [!tip] Why this was necessary
> Hard-coding "producer knows exactly which queue to write to" couples producers to consumer topology. Exchanges decouple that: a producer just publishes with a routing key, and routing logic (which queues get it) lives in bindings that can change without touching producer code.

> [!example]
> A `fanout` exchange named `order-events` is bound to three queues: `billing-queue`, `shipping-queue`, `analytics-queue`. One `publish` call delivers the same order-placed message to all three, with zero routing-key logic needed.

## Queues and bindings

A **queue** stores messages; a **binding** is a rule connecting an exchange to a queue (optionally with a routing-key pattern).

> [!tip] Why this was necessary
> Separating "where messages live" (queue) from "how they get there" (binding) lets you rewire delivery rules — add a new consumer queue, change routing patterns — without redeploying producers.

> [!example]
> A `topic` exchange bound with pattern `orders.eu.*` routes only European order events to `eu-fulfillment-queue`, while `orders.*.cancelled` routes any region's cancellations to a separate `cancellations-queue`.

## Acknowledgments (ack/nack) and redelivery

Consumers explicitly **acknowledge** a message after successful processing; unacknowledged messages (e.g. consumer crash) are automatically **requeued**.

> [!tip] Why this was necessary
> Without explicit acks, a consumer crashing mid-processing would silently lose the message it was working on. Manual acknowledgment guarantees at-least-once delivery — a message is only removed once RabbitMQ has proof it was handled.

> [!example]
> A consumer pulls an "image resize" job, crashes halfway through, and never sends an ack. RabbitMQ detects the dropped connection and redelivers the same message to another available consumer.

## Competing consumers (work queues)

Multiple consumers can subscribe to the same queue; RabbitMQ distributes messages between them (round-robin by default, or based on prefetch/fairness settings).

> [!tip] Why this was necessary
> This is RabbitMQ's core mechanism for horizontal scaling of task processing — add more consumer instances to a queue and throughput increases, without any partitioning scheme to plan ahead of time (unlike Kafka).

> [!example]
> A `pdf-generation` queue receiving 1000 jobs/min has 5 worker instances subscribed; each worker gets roughly 200 jobs/min, and adding a 6th worker immediately reduces each worker's share.

## Publisher confirms

Producers can request a **confirm** from the broker that a published message was successfully received and persisted, similar in spirit to Kafka's `acks`.

> [!tip] Why this was necessary
> Fire-and-forget publishing risks silent message loss if the broker fails to receive or persist a message. Confirms let a producer know definitively whether a publish succeeded, so it can retry only genuinely failed publishes.

> [!example]
> A payment service publishes a `payment-initiated` event and waits for a publisher confirm before returning success to the caller, guaranteeing the event isn't lost even if the broker was momentarily unreachable.

## Dead-letter exchanges (DLX)

Messages that are rejected, expire (TTL), or exceed a queue's length limit can be automatically routed to a separate **dead-letter exchange** instead of being dropped or endlessly retried.

> [!tip] Why this was necessary
> Some messages are genuinely unprocessable (malformed payload, permanently failing downstream call). Without a DLX, they'd either loop forever between requeue and failure, or silently vanish — neither is acceptable for debugging or auditing.

> [!example]
> A queue configured with `x-dead-letter-exchange: dlx` and a max of 3 delivery attempts routes any message that fails a 4th time to `dlx`, where a separate monitoring consumer logs it for manual investigation.

## Message TTL and queue length limits

Individual messages or entire queues can have a **time-to-live**, and queues can cap their maximum length, with configurable behavior (drop oldest, reject newest, or dead-letter) when limits are hit.

> [!tip] Why this was necessary
> Some messages are only useful for a limited time (a "typing indicator" event is worthless after 10 seconds) or queues need bounds to prevent unbounded memory/disk growth if consumers fall behind.

> [!example]
> A `notifications` queue sets `x-message-ttl: 60000` (60s) — a push notification that hasn't been delivered within a minute is dropped rather than sent late and confusing the user.

## Priority queues

A queue can be configured to support **message priority**, so higher-priority messages are delivered before lower-priority ones even if they arrived later.

> [!tip] Why this was necessary
> Not all work is equal — an urgent password-reset email shouldn't wait behind a backlog of bulk marketing emails in the same queue.

> [!example]
> An `emails` queue with `x-max-priority: 10` lets the password-reset producer publish with priority 9, jumping ahead of priority-1 marketing emails still waiting in the queue.

## Clustering and mirrored/quorum queues

RabbitMQ nodes can form a **cluster**, and queues can be replicated across nodes (via **quorum queues**, the modern Raft-based replication mechanism replacing classic mirrored queues) for high availability.

> [!tip] Why this was necessary
> A single-node RabbitMQ is a single point of failure — losing that node loses every queue on it. Quorum queues replicate queue state across a cluster so the queue survives the loss of a minority of nodes.

> [!example]
> A 3-node cluster hosts a quorum queue with replicas on all 3 nodes; when one node crashes, the queue keeps serving publishes/consumes via the remaining 2 replicas with no manual intervention.
