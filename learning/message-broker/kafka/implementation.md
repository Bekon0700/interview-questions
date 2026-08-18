---
title: Kafka — Implementation
category: message-broker
topic: kafka
tags: [learning, message-broker, kafka, implementation]
related: ["[[kafka]]", "[[all-features]]"]
---

Hands-on work for Kafka, using Node.js + [KafkaJS](https://kafka.js.org/) against a local single-broker Kafka (KRaft mode, no ZooKeeper needed). One fully worked example first, then a problem set — **problems 2+ are yours to build unaided**; only the expected final output is given.

## Local setup (for reference)

```yaml
# docker-compose.yml
services:
  kafka:
    image: apache/kafka:latest
    ports:
      - "9092:9092"
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@localhost:9093
```

```bash
npm install kafkajs
```

---

## Worked example — produce and consume a single message

**Goal:** publish one record to a topic and read it back with a basic consumer, understanding the client/producer/consumer lifecycle.

```js
// client.js
const { Kafka } = require('kafkajs');

const kafka = new Kafka({
  clientId: 'learning-app',
  brokers: ['localhost:9092'],
});

module.exports = kafka;
```

```js
// produce.js
const kafka = require('./client');

async function run() {
  const producer = kafka.producer();
  await producer.connect();

  await producer.send({
    topic: 'greetings',
    messages: [
      { key: 'user-1', value: 'hello kafka' },
    ],
  });

  console.log('message sent');
  await producer.disconnect();
}

run().catch(console.error);
```

```js
// consume.js
const kafka = require('./client');

async function run() {
  const consumer = kafka.consumer({ groupId: 'greetings-reader' });
  await consumer.connect();
  await consumer.subscribe({ topic: 'greetings', fromBeginning: true });

  await consumer.run({
    eachMessage: async ({ partition, message }) => {
      console.log(
        `partition=${partition} key=${message.key?.toString()} value=${message.value.toString()}`
      );
    },
  });
}

run().catch(console.error);
```

**Why this works:** `producer.send` appends the record to the `greetings` topic (auto-created with 1 partition by default on most local setups). The consumer, running in group `greetings-reader`, subscribes `fromBeginning: true` so it reads from the start of the log rather than only new records — since nothing has been read by this group before, it gets everything, i.e. the one message just sent.

Running `node produce.js` then `node consume.js` prints:

```
partition=0 key=user-1 value=hello kafka
```

---

## Problem set

Build each of these yourself in your own environment (extending the setup above). Only the expected final output is given — no solution code.

### Beginner

**1. Multiple messages, no key**
Produce 5 messages (no key) to a new topic `numbers` with values `"1"` through `"5"`, then consume and print them all from the beginning.

Expected output (order may vary since there's no key, but all 5 must appear exactly once):
```
value=1
value=2
value=3
value=4
value=5
```

**2. Keyed partitioning**
Create topic `orders` with 3 partitions. Produce 6 messages keyed by `userId` values `A`, `B`, `A`, `C`, `B`, `A` (values can be anything). Consume and print `key` + `partition` for each. Confirm all messages with the same key land on the same partition.

Expected output (exact partition numbers will vary by hash, but this invariant must hold):
```
Every message with key=A prints the same partition number as every other key=A message.
Same for key=B and key=C.
```

### Intermediate

**3. Consumer group scaling**
Create topic `tasks` with 4 partitions, produce 20 messages spread across keys so all partitions get traffic. Start **two** consumer processes in the same group (`groupId: 'task-workers'`) at the same time. Log which partitions each process gets assigned.

Expected output:
```
Between the two processes, all 4 partitions are covered exactly once (e.g. process A gets partitions [0,1], process B gets partitions [2,3] — exact split may vary, but no partition is assigned to both, and none is left unassigned).
```

**4. Offset replay**
Using the `orders` topic from problem 2, run a consumer in group `replay-demo` that reads all 6 messages once, then stop it. Without producing any new messages, reset that consumer group's offsets to the beginning (via `consumer.seek()` per partition on startup, or the CLI: `kafka-consumer-groups.sh --group replay-demo --reset-offsets --to-earliest --execute --topic orders`) and run it again.

Expected output:
```
First run: prints all 6 messages once.
Second run (after reset): prints the same 6 messages again, identical to the first run.
```

### Advanced

**5. Idempotent producer + duplicate detection**
Create a producer with `idempotent: true` and `acks: -1` (all). Write a small wrapper that simulates a retry by calling `producer.send` twice with the exact same message batch object (simulating a network-timeout-then-retry scenario). Consume the topic and count how many records actually landed.

Expected output:
```
Even though send() was effectively invoked twice for "the same" logical write, the topic contains only 1 record from that batch — not 2.
```
(Note: true idempotence requires the retry to be a genuine broker-level retry of the same producer epoch/sequence, not just calling `.send()` twice from application code — part of this problem is discovering that distinction and documenting what you had to do to actually trigger dedupe-worthy behavior, e.g. forcing a retry via `retry` config and a transient broker disconnect, rather than two independent sends.)

**6. Windowed aggregation**
Produce `page-view` events (each with a `page` key and a timestamp value) at a steady rate for ~30 seconds across 3 different page keys. Build a consumer that maintains an in-memory count per page within 5-second tumbling windows, and prints a summary line at the end of each window, then resets counts for the next window.

Expected output (values will depend on your production rate, but the shape must match):
```
[window 0-5s] home=4 pricing=2 about=1
[window 5-10s] home=3 pricing=3 about=2
... (one summary line every 5 seconds for the ~30s run, three page keys per line, counts resetting each window)
```
