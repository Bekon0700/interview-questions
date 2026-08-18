---
title: RabbitMQ — Implementation
category: message-broker
topic: rabbitmq
tags: [learning, message-broker, rabbitmq, implementation]
related: ["[[rabbitmq]]", "[[all-features]]"]
---

Hands-on work for RabbitMQ, using Node.js + [amqplib](https://www.npmjs.com/package/amqplib) against a local RabbitMQ instance. One fully worked example first, then a problem set — **problems 2+ are yours to build unaided**; only the expected final output is given.

## Local setup (for reference)

```yaml
# docker-compose.yml
services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"   # AMQP
      - "15672:15672" # management UI (guest/guest)
```

```bash
npm install amqplib
```

---

## Worked example — publish and consume via a direct exchange

**Goal:** publish one message through a `direct` exchange to a queue and consume it, understanding the exchange → binding → queue → consumer flow (as opposed to Kafka's topic → partition → consumer-group flow).

```js
// publish.js
const amqp = require('amqplib');

async function run() {
  const connection = await amqp.connect('amqp://localhost');
  const channel = await connection.createChannel();

  const exchange = 'greetings-exchange';
  const routingKey = 'greeting.hello';

  await channel.assertExchange(exchange, 'direct', { durable: true });

  channel.publish(exchange, routingKey, Buffer.from('hello rabbitmq'));
  console.log('message published');

  await channel.close();
  await connection.close();
}

run().catch(console.error);
```

```js
// consume.js
const amqp = require('amqplib');

async function run() {
  const connection = await amqp.connect('amqp://localhost');
  const channel = await connection.createChannel();

  const exchange = 'greetings-exchange';
  const queue = 'greetings-queue';
  const routingKey = 'greeting.hello';

  await channel.assertExchange(exchange, 'direct', { durable: true });
  await channel.assertQueue(queue, { durable: true });
  await channel.bindQueue(queue, exchange, routingKey);

  channel.consume(queue, (msg) => {
    if (msg) {
      console.log(`routingKey=${msg.fields.routingKey} value=${msg.content.toString()}`);
      channel.ack(msg);
    }
  });
}

run().catch(console.error);
```

**Why this works:** unlike Kafka, the producer never touches the queue directly — it publishes to the `greetings-exchange` with routing key `greeting.hello`. The consumer script declares the queue and **binds** it to that exchange with the same routing key, which is what makes the exchange forward matching messages there. The consumer must explicitly `ack` the message, or RabbitMQ will consider it undelivered and requeue it on disconnect.

Running `node consume.js` (leave it running — it consumes continuously) then, in another terminal, `node publish.js` prints:

```
routingKey=greeting.hello value=hello rabbitmq
```

---

## Problem set

Build each of these yourself in your own environment (extending the setup above). Only the expected final output is given — no solution code.

### Beginner

**1. Fanout broadcast**
Create a `fanout` exchange `broadcast-exchange`. Create two separate queues, each bound to it with no routing key needed (fanout ignores routing keys). Publish one message. Run two separate consumer processes, one per queue.

Expected output:
```
Both consumer processes print the exact same message content — the single publish reached both queues independently.
```

**2. Topic routing patterns**
Create a `topic` exchange `logs-exchange`. Bind one queue with pattern `logs.error.*` and another with pattern `logs.*.critical`. Publish three messages with routing keys `logs.error.db`, `logs.warning.critical`, and `logs.info.minor`.

Expected output:
```
Queue bound to "logs.error.*" receives only the logs.error.db message.
Queue bound to "logs.*.critical" receives only the logs.warning.critical message.
The logs.info.minor message is not delivered to either queue (no matching binding) and is simply dropped by the exchange.
```

### Intermediate

**3. Competing consumers (work queue)**
Create a queue `jobs-queue` (no exchange needed — publish directly to the default exchange with the queue name as routing key). Publish 10 messages numbered `job-1` through `job-10`. Start **two** consumer processes on the same queue simultaneously, each logging which job numbers it receives.

Expected output:
```
Between the two consumer processes, all 10 jobs are processed exactly once in total.
Each process gets a roughly even share (not necessarily exact, but neither process gets 0 and neither gets all 10).
```

**4. Manual ack vs. crash requeue**
Using `jobs-queue`, write a consumer that receives a message, logs it, then deliberately does **not** ack it (simulate a crash by closing the connection immediately after receiving one message without calling `channel.ack`). Restart a normal (acking) consumer afterward.

Expected output:
```
The un-acked message is NOT lost — after the crashed consumer's connection closes, RabbitMQ redelivers that same message to the next consumer that connects, which then processes and acks it successfully.
```

### Advanced

**5. Dead-letter exchange**
Create queue `retry-queue` with `x-dead-letter-exchange` pointing to a `dlx-exchange` bound to a `dead-letter-queue`. Write a consumer that always `nack`s messages with `requeue: false` (simulating permanent processing failure). Publish one message to `retry-queue`.

Expected output:
```
The message never stays in retry-queue — after being nacked with requeue:false, it is routed to dead-letter-queue, where a separate consumer can observe it landed there instead of being silently lost.
```

**6. Priority queue ordering**
Create a queue with `x-max-priority: 10`. Publish 5 messages with priorities `1, 9, 3, 7, 5` (in that publish order), then start a consumer that reads all 5.

Expected output:
```
The consumer receives the messages in priority order, not publish order: priority 9 first, then 7, then 5, then 3, then 1 — even though they were published as 1, 9, 3, 7, 5.
```
