---
title: Kafka
category: message-broker
topic: kafka
tags: [learning, message-broker, kafka, event-streaming]
related: ["[[../message-broker]]"]
---

> [!abstract] Overview
> **Apache Kafka** is a distributed, durable **event-streaming platform**. Instead of modeling messaging as queues that empty once consumed, Kafka models it as an **append-only log**: producers write records to the end of a log, and consumers read from wherever they like in that log, independently, at their own pace — without deleting anything. That single idea (a replayable, persistent log) is what separates Kafka from traditional message queues like RabbitMQ or SQS.

## Backstory

Kafka was built at LinkedIn around 2010–2011 to solve a specific pain: LinkedIn had many systems (search, recommendations, monitoring, data warehousing) that all needed the same streams of activity data (page views, clicks, user actions), but existing point-to-point messaging and batch ETL pipelines couldn't handle the volume or the fan-out cleanly. Jay Kreps, Neha Narkhede, and Jun Rao designed Kafka as a unified, high-throughput log that any number of downstream systems could read independently. It was open-sourced in 2011 and became an Apache project in 2012.

The name is a nod to Franz Kafka — Jay Kreps said he picked it because the system was "a system optimized for writing" and he liked Kafka's writing, with no deeper technical meaning.

## What it is

- A **distributed commit log**: data is organized into **topics**, each topic split into **partitions**, and each partition is an ordered, append-only sequence of records.
- Records are **retained for a configured period** (or size), not deleted on read — so multiple consumers, or the same consumer replaying history, can all read the same data independently.
- **Producers** append records to partitions; **consumers** (organized into **consumer groups**) read records by tracking an **offset** (their position in the log) per partition.
- Kafka runs as a **cluster of brokers**; each partition is replicated across brokers for durability and fault tolerance.

## Why it exists

Traditional queues (like classic RabbitMQ usage) are built around **"consume and remove"** — once a message is processed, it's gone. That's fine for task distribution (send one email once), but breaks down when:
- Multiple independent systems need to react to the **same** event (fan-out to many consumers without duplicating the queue).
- Consumers need to **replay** history — reprocess the last 24 hours of events because a bug was fixed, or backfill a new system that just came online.
- You need very **high throughput**, sequential disk writes, and horizontal scale beyond what a single broker can handle.

Kafka's log-based model solves all three: retention means replay is just "reset your offset," and partitioning means throughput scales by adding more partitions/brokers.

## How it works (high level)

1. A **topic** is created, split into N **partitions**.
2. A **producer** sends a record; it's routed to a partition (via a key, or round-robin if no key).
3. The record is appended to the end of that partition's log and replicated to follower brokers.
4. A **consumer group** subscribes to the topic; Kafka assigns each partition to exactly one consumer *within* that group, so the group as a whole processes every partition, but no two consumers in the same group process the same partition at once.
5. Each consumer tracks its **offset** — how far it has read — so it can resume after a crash, or intentionally rewind to reprocess.
6. **ZooKeeper** historically coordinated cluster metadata/broker leadership; modern Kafka (KRaft mode, Kafka 3.x+) replaces this with a built-in Raft-based controller, removing the ZooKeeper dependency.
