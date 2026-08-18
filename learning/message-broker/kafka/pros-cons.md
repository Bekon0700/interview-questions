---
title: Kafka — Pros & Cons
category: message-broker
topic: kafka
tags: [learning, message-broker, kafka, pros-cons]
related: ["[[kafka]]"]
---

> [!success] Pros
> - **Very high throughput** — sequential disk writes and partitioned parallelism let a modest Kafka cluster sustain millions of records/sec. Example: LinkedIn-scale deployments handle trillions of messages/day across a cluster.
> - **Durable, replayable log** — records aren't deleted on read, so a new consumer can backfill from history, or an existing one can rewind to reprocess after a bug. Example: resetting a fraud-detection consumer's offset to reprocess the last 24 hours after fixing a logic error.
> - **Natural fan-out** — many independent consumer groups can read the same topic without affecting each other or duplicating storage. Example: `orders` topic feeds a billing service, a shipping service, and an analytics pipeline simultaneously, each at its own pace.
> - **Horizontal scalability** — throughput and storage scale by adding partitions and brokers, not by upgrading a single machine. Example: growing from 6 to 24 partitions across more brokers to absorb 4x traffic growth.
> - **Strong durability guarantees** — replication plus configurable `acks` means committed writes survive broker failure. Example: `acks=all` with replication factor 3 tolerates the loss of 2 out of 3 brokers without losing acknowledged writes.
> - **Rich ecosystem** — Kafka Connect and Kafka Streams/ksqlDB cover data integration and stream processing without bespoke glue code. Example: a Debezium connector streams DB changes in; a Streams app aggregates them, without either being hand-rolled.

> [!warning] Cons
> - **Operational complexity** — running, tuning, and monitoring a Kafka cluster (partition counts, replication, retention, broker sizing) is significantly harder than standing up a simple queue. Example: a poorly chosen partition count can't be reduced later without recreating the topic, causing production headaches.
> - **Not ideal for simple task queues** — Kafka's consumer-group model doesn't give per-message acknowledgment/retry/dead-letter semantics as naturally as RabbitMQ or SQS. Example: implementing "retry this one failed message 3 times then dead-letter it" requires extra application logic Kafka doesn't provide out of the box.
> - **Higher latency than in-memory brokers for small workloads** — the disk-backed log and batching optimizations that give Kafka throughput add latency compared to a lightweight in-memory broker for low-volume use cases. Example: a small internal app sending a few messages/minute pays more operational cost for less benefit than just using RabbitMQ.
> - **Storage costs at scale** — long retention windows on high-volume topics consume significant disk (and replicated 3x by default). Example: retaining 30 days of a 1TB/day topic with replication factor 3 requires ~90TB of storage.
> - **Rebalancing pauses** — when a consumer joins/leaves a group, Kafka triggers a rebalance that can briefly stop processing for that group (mitigated but not eliminated by newer cooperative rebalancing protocols). Example: a rolling deploy of a consumer service causes brief processing gaps as partitions get reassigned.
> - **Steeper learning curve** — concepts like partitions, offsets, consumer groups, and replication require real investment to use correctly, versus the simpler mental model of "push a message, pop a message." Example: new engineers commonly misunderstand that ordering is only guaranteed per-partition, not topic-wide, and design bugs around that assumption.
