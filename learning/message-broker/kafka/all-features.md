---
title: Kafka — All Features
category: message-broker
topic: kafka
tags: [learning, message-broker, kafka, features]
related: ["[[kafka]]"]
---

Comprehensive feature list for Kafka. Each feature includes why it was necessary and a concrete example.

## Topics and partitions

A **topic** is a named stream of records; it's split into **partitions**, each an ordered, append-only log.

> [!tip] Why this was necessary
> A single log on a single machine caps throughput at that machine's disk/network. Splitting a topic into partitions lets Kafka spread writes and reads across many brokers, so throughput scales roughly linearly with partition count.

> [!example]
> A topic `orders` with 6 partitions can be spread across 3 brokers (2 partitions each). Producers publishing 100k orders/sec are distributed across all 6 partitions instead of bottlenecking on one log.

## Partition keys and ordering

Producers can attach a **key** to a record; Kafka hashes the key to consistently route all records with that key to the same partition, preserving order *within* that key.

> [!tip] Why this was necessary
> Kafka only guarantees ordering within a single partition, not across a whole topic. Keying lets you choose which records must stay ordered relative to each other (e.g. all events for one user) while still parallelizing across partitions for everything else.

> [!example]
> Keying `orders` events by `userId` ensures all of one user's order events arrive in order, even though different users' events may be processed out of order relative to each other across partitions.

## Consumer groups

Multiple consumer instances can share a **group id**; Kafka assigns each partition to exactly one consumer within the group, and rebalances assignments when consumers join or leave.

> [!tip] Why this was necessary
> This is how Kafka achieves both fan-out (different groups each get their own full copy of the stream) and horizontal scaling (consumers within one group split the work, so adding more consumers speeds up processing up to the partition count).

> [!example]
> A topic with 6 partitions and a consumer group of 3 instances: each consumer is assigned 2 partitions. Meanwhile a *second*, independent consumer group (e.g. an analytics pipeline) reads the exact same topic from the beginning, unaffected by the first group.

## Offsets and replay

Each consumer (group) tracks an **offset** per partition — its read position — stored in Kafka itself (an internal `__consumer_offsets` topic). Consumers can reset their offset to reprocess history.

> [!tip] Why this was necessary
> Because Kafka doesn't delete records on read, "replay" is just a pointer reset rather than a special operation — this is what lets a new consumer backfill from history, or an existing one recover from a bug by reprocessing the last day's events.

> [!example]
> A bug in a fraud-detection consumer processed a day's events incorrectly. Ops resets that consumer group's offset to 24 hours ago and lets it reprocess — no data was lost because it was never deleted.

## Replication and fault tolerance

Each partition has a configurable number of **replicas** across brokers; one is the **leader** (handles all reads/writes), others are **followers** that copy the leader's log.

> [!tip] Why this was necessary
> Disks and machines fail. Without replication, losing a broker means permanently losing every partition it hosted. Replication lets Kafka promote a follower to leader and keep serving traffic transparently.

> [!example]
> A topic with replication factor 3: if the broker hosting the leader for partition 2 crashes, one of the two in-sync followers is automatically elected the new leader within seconds.

## Retention policies

Topics can retain records for a configured **time** (e.g. 7 days) or **size**, or use **log compaction** to keep only the latest record per key indefinitely.

> [!tip] Why this was necessary
> Different use cases need different retention: an event log for replay/debugging wants time-based retention; a "current state" topic (like a changelog of account balances) wants compaction so it always holds the latest value per key without growing forever.

> [!example]
> A `user-profile-updates` topic uses log compaction keyed by `userId` — even after millions of updates, the topic only retains the most recent profile record per user, acting like a compacted snapshot.

## Producer delivery guarantees (acks, idempotence, exactly-once)

Producers can configure `acks` (how many replicas must confirm a write) and enable **idempotent** producers or **transactions** for exactly-once semantics across multiple partitions/topics.

> [!tip] Why this was necessary
> Networks retry, and naive retries after a timeout can duplicate a write. Idempotent producers (using a producer ID + sequence number) let Kafka detect and drop duplicate retries, and transactions extend that guarantee across multiple partitions in a single atomic write.

> [!example]
> A payment producer sets `acks=all` and `enable.idempotence=true`. If a network blip causes the producer to retry a write that actually succeeded, Kafka recognizes the duplicate sequence number and doesn't double-append it.

## Kafka Connect

A framework for **connectors** that move data between Kafka and external systems (databases, S3, Elasticsearch) without writing custom producer/consumer code.

> [!tip] Why this was necessary
> Most real systems need to get data *into* Kafka from existing databases (CDC) and *out* of Kafka into stores made for querying. Writing bespoke integration code for every source/sink doesn't scale across an organization — Connect standardizes it into configurable, reusable connectors.

> [!example]
> A Debezium source connector streams every row change from a PostgreSQL `orders` table into a Kafka topic in real time, without the application code ever calling Kafka directly.

## Kafka Streams / ksqlDB

A client library (Kafka Streams) and SQL-like layer (ksqlDB) for building stream-processing applications directly on top of Kafka topics — filtering, joining, aggregating streams into new topics.

> [!tip] Why this was necessary
> Many use cases need more than "move records from A to B" — they need running aggregations, joins between streams, or windowed computations (e.g. rolling 5-minute counts). Without this, every team would reimplement stateful stream processing on their own.

> [!example]
> A Kafka Streams app consumes a `page-views` topic, aggregates a 1-minute tumbling window count per page, and writes the result to a `page-view-counts` topic for a real-time dashboard.

## KRaft (Kafka Raft) — ZooKeeper removal

Since Kafka 3.x (default from 4.0), cluster metadata and controller election are handled by Kafka's own built-in Raft-based consensus (**KRaft**) instead of an external ZooKeeper ensemble.

> [!tip] Why this was necessary
> Running ZooKeeper alongside Kafka meant operating two distributed systems with separate failure modes, scaling limits, and operational knowledge. KRaft folds that consensus responsibility into Kafka itself, simplifying deployment and improving controller failover time.

> [!example]
> A new Kafka cluster deployed in 2024+ runs with `process.roles=broker,controller` and no separate ZooKeeper cluster to install, patch, or monitor.
