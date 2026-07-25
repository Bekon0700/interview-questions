---
title: System Design — Answers
topic: system-design
tags: [interview, fullstack, system-design]
related: ["[[07-cv-deep-dive]]", "[[05-databases]]", "[[04-nestjs]]", "[[09-security]]"]
---

# System Design — Answers

> [!abstract] How to use this note
> Each item has the **question**, a plain-English **explanation**, an **example** (architecture sketch or scenario), and a **Pros / Cons** box. Numbers match [[questions/06-system-design|the questions file]] exactly. Advanced answers (Q19–28) map to your CV projects — use them to tell specific, credible stories in interviews.

> [!warning] CV-linked answers (Q19–28)
> Advanced answers are **model templates** based on your Rokomari Affiliate, WebChat, Ads, and e-commerce work. Replace specifics with what you actually built — interviewers will probe. See [[07-cv-deep-dive]] for STAR stories.

---

## Beginner / Fundamentals

### 1. System design interview steps

> [!question] Q1
> What are the steps you follow in a system design interview? What questions do you ask first?

A system design interview is not a quiz about memorised architectures — it is a structured conversation where you show how you think under ambiguity. Follow a repeatable flow so you never jump straight to boxes and arrows.

**Step 1 — Clarify requirements.** Ask about functional features (what the system must do) and non-functional requirements (scale, latency, availability, consistency, durability). Example clarifiers: "How many daily active users?" "Is strong consistency required for payments, or is eventual OK for analytics?" "What is the read/write ratio?"

**Step 2 — Estimate scale.** Back-of-envelope math: QPS (queries per second), storage over time, bandwidth. This drives whether you need caching, sharding, or queues. Even rough numbers ("~1k writes/sec at peak") show engineering judgment.

**Step 3 — Define APIs and data model.** Sketch the main endpoints or events and the core entities (users, orders, messages). This grounds the design in concrete contracts before infrastructure.

**Step 4 — High-level architecture.** Draw clients → load balancer → services → database/cache/queue. Keep it simple first; add complexity only where scale demands it.

**Step 5 — Deep dive.** Pick the hardest or most interesting component (e.g. real-time delivery, idempotent payments) and explain it in detail.

**Step 6 — Bottlenecks and trade-offs.** Identify single points of failure, hot keys, consistency gaps. Propose mitigations (replication, caching, partitioning) and explain what you are trading away.

> [!example] Opening questions in an interview
> "Before I draw anything — who are the users and what is the core action?" → "How many reads vs writes?" → "What latency is acceptable?" → "Do we need 99.9% or 99.99% availability?" → "Any compliance or security constraints?" These questions signal seniority more than naming Redis immediately.

> [!success] Pros / Cons
> **Pros of a structured approach:** you avoid premature optimisation, demonstrate communication, and leave time for depth. Interviewers can follow your reasoning. **Cons:** spending too long on requirements can eat deep-dive time — practice balancing (~5 min clarify, ~5 min estimate, ~15 min design, ~10 min deep dive).

> [!tip] Interview habit
> Narrate your thinking aloud. Say "I'm choosing X because of Y, and the trade-off is Z." That is what they are grading.

---

### 2. Horizontal vs vertical scaling

> [!question] Q2
> What is horizontal vs vertical scaling? Trade-offs of each?

**Vertical scaling (scale up)** means running on a bigger machine — more CPU, RAM, or faster disk. You keep one (or few) servers and make them more powerful. It is simple: no code changes, no distributed-system headaches. But every machine has a ceiling, upgrades often require downtime, and a single large box is still a single point of failure.

**Horizontal scaling (scale out)** means adding more machines and spreading work across them, usually behind a [[#3 Load balancer|load balancer]]. You can grow near-linearly by adding nodes, survive individual machine failures, and match cloud economics (many small instances). The cost is complexity: services must be **stateless** (or externalise state to [[05-databases|Redis/DB]]), you need distributed coordination, and data must be partitioned ([[#9 Database replication and sharding|sharded or replicated]]).

Modern web systems almost always scale horizontally for the application tier and combine horizontal + vertical for databases (replicas + bigger primaries, then sharding).

> [!example] E-commerce checkout at peak
> Vertical: one 64-core DB server handles all orders until Black Friday overwhelms it — you cannot add capacity without a risky migration. Horizontal: ten stateless [[04-nestjs|NestJS]] API nodes behind a load balancer; [[16-mongodb|MongoDB]] primary + read replicas; Redis cache for hot product pages. Flash sale traffic spreads across nodes instead of crushing one box.

> [!success] Pros / Cons
> **Vertical — Pros:** simple ops, strong consistency on one node, no network partitions between app parts. **Cons:** hard ceiling, expensive top-end hardware, downtime during upgrades, SPOF. **Horizontal — Pros:** elastic growth, fault tolerance, cost-efficient at scale. **Cons:** distributed state, eventual consistency, harder debugging, needs stateless design and good observability.

---

### 3. Load balancer

> [!question] Q3
> What is a load balancer and what algorithms does it use (round robin, least connections, IP hash)?

A **load balancer** sits in front of multiple server instances and distributes incoming traffic so no single machine bears the full load. It also performs **health checks** — if a backend stops responding, traffic is routed away, improving availability.

**Common algorithms:**
- **Round robin** — rotate requests evenly across servers. Simple and fair when all requests cost roughly the same.
- **Least connections** — send the next request to the server with the fewest active connections. Better when request duration varies (long-polling, WebSocket-heavy workloads).
- **IP hash / consistent hash** — map a client IP (or session key) to a specific server so the same client always hits the same backend. Useful for **sticky sessions** when local state has not yet been externalised.
- **Weighted** — send more traffic to more powerful nodes.

Load balancers operate at **L4 (TCP)** — fast, connection-level routing — or **L7 (HTTP)** — can route by URL path, host header, or headers (e.g. `/api` → API cluster, `/static` → CDN).

> [!example] Three NestJS API instances
> ```
> Clients → [Load Balancer] → API-1, API-2, API-3
>                ↓ health checks every 10s
>            API-2 unhealthy → removed from pool
> ```
> Round robin for REST APIs; least connections for WebSocket upgrade traffic; IP hash only if you truly need stickiness (prefer externalising session to Redis instead).

> [!success] Pros / Cons
> **Pros:** higher throughput, no single app-server SPOF, graceful rolling deploys (drain one node at a time), SSL termination at the edge. **Cons:** added hop latency (usually negligible), sticky sessions can cause uneven load, the load balancer itself can become a SPOF (mitigate with redundant LB pairs or managed services like AWS ALB).

---

### 4. Stateless vs stateful services

> [!question] Q4
> What is the difference between stateless and stateful services? Why do we prefer stateless for scaling?

A **stateful** service keeps client-specific data in local memory between requests — session objects, in-memory shopping carts, open WebSocket connection state tied to one process. If that process dies or traffic shifts to another node, the client loses context unless you add complex session migration.

A **stateless** service treats every request independently. Any instance can handle any request because all persistent state lives in external stores ([[16-mongodb|MongoDB]], Redis, S3). The server may hold ephemeral data (a DB connection from a pool) but nothing that identifies a specific user's session across requests.

We prefer stateless app servers because horizontal scaling becomes trivial: add nodes, register them with the load balancer, done. Deployments are rolling and safe. Stateful designs force sticky sessions, complicate failover, and make autoscaling harder.

> [!example] Session in Redis vs in-memory
> **Stateful (bad for scale):** login stores `req.session.userId` in Node's memory on server A; next request hits server B → user appears logged out. **Stateless pattern:** store session in Redis keyed by session ID; every Node instance reads/writes the same Redis. Any node serves any request.

> [!success] Pros / Cons
> **Stateless — Pros:** elastic scaling, simple deploys, fault tolerance. **Cons:** every request may hit Redis/DB (mitigate with caching and JWT for auth). **Stateful — Pros:** lower latency for connection-heavy work (WebSockets), simpler local algorithms. **Cons:** hard to scale horizontally, painful failover. Real-time systems often use stateful *connections* on each node but externalise routing state (Redis pub/sub) — see [[#24 Scaling WebSockets to millions]].

---

### 5. CDN (Content Delivery Network)

> [!question] Q5
> What is a CDN and when does it help?

A **CDN** is a geographically distributed network of edge servers that cache and serve content close to users. Instead of every request travelling to your origin server in one region, static assets are served from a nearby PoP (point of presence), cutting latency and origin load.

CDNs help when:
- Serving **static assets** — images, JS, CSS, fonts, videos.
- Content is **read-heavy and cacheable** — product images, marketing pages, API responses with long `Cache-Control` headers.
- You have **global users** — latency to a single origin would be high for distant regions.
- You need **DDoS absorption** and TLS termination at the edge.

They help less for personalised, uncacheable API responses (user-specific cart, auth) unless you cache semi-static fragments (ISR pages, public catalog slices).

> [!example] Next.js e-commerce product page
> Origin (Bangladesh) serves HTML via ISR; `/_next/static/*`, product images, and fonts are CDN-cached at edge nodes in Singapore, Mumbai, and Frankfurt. A user in London loads images from a European edge — ~30 ms instead of ~250 ms round trip to origin. See [[07-cv-deep-dive]] for your page-load improvements.

> [!success] Pros / Cons
> **Pros:** lower latency, reduced origin bandwidth/cost, better resilience under traffic spikes, automatic HTTP/2 and compression at edge. **Cons:** cache invalidation complexity (stale product prices if TTL too long), cost at very high scale, not a substitute for application-level caching ([[#8 Caching layers]]) for dynamic data.

---

### 6. Synchronous vs asynchronous communication

> [!question] Q6
> What is the difference between synchronous (request/response) and asynchronous (message queue) communication?

**Synchronous (sync)** communication means the caller sends a request and **blocks until a response arrives** — classic REST/HTTP, gRPC call/response. The caller and callee are coupled in time: if the downstream service is slow or down, the caller feels it immediately. Simplicity and immediate feedback are the wins.

**Asynchronous (async)** communication means the caller **publishes a message** (to a queue or topic) and continues without waiting for processing to finish. A separate consumer picks up the message later. The producer and consumer are **decoupled in time** — spikes are buffered, retries are natural, and slow work does not block the user-facing path.

Choose sync when the user needs an immediate answer ("Did payment succeed?"). Choose async when work can happen in the background (send email, calculate commission, aggregate analytics).

> [!example] Order placement flow
> **Sync path:** validate cart → charge payment → return order confirmation (user waits ~500 ms). **Async path:** after order saved, publish `OrderCreated` to RabbitMQ; workers send email, update affiliate commission, push to analytics. User gets fast confirmation; downstream systems catch up within seconds.

> [!success] Pros / Cons
> **Sync — Pros:** simple mental model, strong consistency in one request, easy debugging. **Cons:** cascading failures, latency stacks across service chains, poor spike absorption. **Async — Pros:** decoupling, resilience, smooths load, enables retries. **Cons:** eventual consistency, harder tracing, duplicate-message handling required ([[#25 Exactly-once vs at-least-once]]).

---

### 7. Message queue

> [!question] Q7
> What is a message queue and what problems does it solve? (your CV: RabbitMQ)

A **message queue** is middleware (RabbitMQ, Amazon SQS, Kafka) that buffers messages between **producers** (services that publish events) and **consumers** (workers that process them). Producers write to the queue; consumers pull at their own pace. Messages can be persisted to disk so they survive broker restarts.

**Problems it solves:**
1. **Decoupling** — producer does not need to know which service consumes the event or whether it is online.
2. **Load leveling** — absorb traffic spikes (600 orders in ten minutes) without overwhelming downstream DBs or APIs.
3. **Async processing** — move slow work off the request path (commission calc, notifications).
4. **Reliability** — durable queues + acknowledgements + retries + dead-letter queues for poison messages.
5. **Backpressure** — consumers process at sustainable rate; queue depth becomes a visible metric.

In your CV systems, RabbitMQ offloaded order/commission processing and chat bot replies from the synchronous API path. See [[04-nestjs]] and [[07-cv-deep-dive]].

> [!example] Affiliate order → commission
> ```
> Order API → persist order in MongoDB → publish to `orders.created` queue → return 201
>                                              ↓
>                                    Commission worker (NestJS)
>                                              ↓
>                                    Idempotent upsert by orderId
> ```
> API responds in ~100 ms; commission appears in reports within seconds.

> [!success] Pros / Cons
> **Pros:** resilience, scalability, clear separation of concerns, natural retry semantics. **Cons:** operational overhead (monitoring queue depth, DLQ replay), eventual consistency, duplicate delivery requires idempotent consumers ([[#25 Exactly-once vs at-least-once]]), ordering guarantees vary by broker.

> [!info] RabbitMQ vs Kafka
> **RabbitMQ** — great for task queues, routing, moderate throughput, per-message acks. **Kafka** — log-based, very high throughput, replay by offset, better for event streaming and analytics pipelines ([[#26 Click/impression tracking pipeline]]).

---

### 8. Caching layers

> [!question] Q8
> What is caching and where can you add caches in a system (client, CDN, app, DB)?

**Caching** stores copies of frequently accessed data in faster, closer storage so repeated reads avoid expensive recomputation or DB round trips. Every layer has different scope, TTL, and invalidation rules.

**Cache layers (far → near user):**
1. **Client / browser** — HTTP `Cache-Control`, ETags; zero network for repeat visits.
2. **CDN (edge)** — static assets and cacheable HTML/API at PoPs ([[#5 CDN]]).
3. **Application cache (Redis / in-memory)** — hot keys, computed aggregates, session data, rate-limit counters. Your WebChat and affiliate reporting used Redis here.
4. **Database cache** — MongoDB WiredTiger cache, PostgreSQL shared buffers; query result cache inside the ORM.

**Strategy:** place cache where reads repeat and staleness is tolerable. Define **TTL** (time-to-live) and **invalidation** (delete on write vs lazy expiry). See [[05-databases#17 Caching strategies]] for cache-aside, write-through, and stampede prevention.

> [!example] Affiliate dashboard report
> Request for "today's commissions" → check Redis key `report:affiliate:123:2026-07-26` → hit: return in 5 ms. Miss: query MongoDB with compound index ([[16-mongodb]]), store result with 5-minute TTL, return. On new commission event, worker deletes or updates the key.

> [!success] Pros / Cons
> **Pros:** dramatic latency and DB load reduction, better user experience under read-heavy traffic. **Cons:** stale data risk, invalidation bugs, memory cost, cache stampede on hot key expiry ([[05-databases#19 Cache stampede]]). Multi-layer caches need a clear consistency story.

---

### 9. Database replication and sharding

> [!question] Q9
> What is database replication and sharding at a high level?

**Replication** copies data to multiple database nodes. A **primary** (leader) handles writes; **replicas** (followers) receive a stream of changes and serve read queries or stand by for failover. Benefits: higher read throughput, availability if primary fails (promote a replica), geographic read locality. Trade-off: **replication lag** — reads from replicas may be slightly stale.

**Sharding** (partitioning) splits data **horizontally** across multiple independent databases (shards), each holding a subset keyed by a **shard key** (e.g. `userId`, `tenantId`). No single node holds the full dataset, so write and storage capacity scales out. Trade-off: cross-shard queries are expensive or impossible; bad shard keys create hotspots.

Production systems often combine both: several shards, each with its own replica set. See [[05-databases#13 Replication]] and [[05-databases#14 Sharding]].

> [!example] MongoDB replica set + future sharding
> ```
>                    ┌─ Secondary (reads, backup)
> Primary (writes) ──┼─ Secondary (reads)
>                    └─ Arbiter (election only)
> ```
> At 10 TB and write pressure beyond one primary, shard by `affiliateId` into four shards, each a replica set. Affiliate reports always query one shard when filtered by affiliate.

> [!success] Pros / Cons
> **Replication — Pros:** read scale, HA, backups. **Cons:** lag, failover complexity, split-brain risk without proper quorum. **Sharding — Pros:** write/storage scale beyond one machine. **Cons:** rebalancing pain, application must shard-key-aware, joins across shards hard. Prefer replication first; shard when metrics demand it.

---

### 10. REST vs WebSocket vs long polling

> [!question] Q10
> What are the differences between REST, WebSocket, and long polling?

**REST** over HTTP is **stateless request/response**: client sends GET/POST, server responds, connection closes. Ideal for CRUD APIs, caching, and standard tooling. Not suited for server-initiated push without repeated requests.

**WebSocket** opens a **persistent, full-duplex** TCP connection upgraded from HTTP. Both sides can send messages anytime with low overhead per message. Ideal for real-time chat, live notifications, collaborative editing. Requires connection management and scaling infrastructure ([[#24 Scaling WebSockets to millions]]).

**Long polling** simulates push over HTTP: client sends a request; server **holds it open** until data arrives (or timeout), then client immediately opens another. Works through restrictive firewalls where WebSockets are blocked. Higher latency and overhead than WebSocket due to repeated HTTP handshakes.

> [!example] WebChat message delivery
> **REST:** client polls `GET /messages?since=t` every 2 s — simple but wasteful and up to 2 s delay. **WebSocket:** server pushes new messages instantly on the open socket. **Long polling:** fallback for corporate networks blocking WS upgrade. Your WebChat uses WebSockets for low-latency two-way messaging with Redis pub/sub for multi-server fan-out.

> [!success] Pros / Cons
> **REST — Pros:** universal, cacheable, stateless, easy to scale. **Cons:** no native server push, chatty for real-time. **WebSocket — Pros:** low latency, bidirectional, efficient for many messages. **Cons:** stateful connections, harder to scale, no HTTP caching. **Long polling — Pros:** works everywhere HTTP works. **Cons:** higher latency, more server resources than WS, awkward under load.

---

## Intermediate

### 11. URL shortener

> [!question] Q11
> Design a URL shortener (encoding, storage, redirects, scale). (classic warm-up)

**Requirements:** given a long URL, return a short code; visiting the short URL redirects to the original. Handle billions of URLs, very high read:write ratio (~100:1).

**Encoding:** Map each URL to a short key. Options: (a) **auto-increment ID → base62** (`aB3xK9`) — compact, no collisions; (b) **hash of URL** — risk of collisions (handle with retry or suffix); (c) **custom aliases** — user-chosen, must check uniqueness.

**Storage:** Key-value store or DB table: `shortCode → longUrl, createdAt, userId, expiresAt`. [[16-mongodb]] or DynamoDB works; Redis for hot keys.

**Redirect flow:** `GET /{code}` → lookup (cache first) → HTTP **301** (permanent, cacheable) or **302** (temporary, allows analytics hit on every visit).

**Scale:** Heavy caching (Redis + CDN for popular links). Write-once-read-many. Async click analytics via queue. DB sharded by hash of code if needed.

> [!example] Architecture sketch
> ```
> Client → CDN (cache 301 for hot links) → LB → Redirect Service
>                                              ↓
>                                         Redis (hot codes)
>                                              ↓ miss
>                                         MongoDB / KV store
> Click tracking: redirect enqueues event → analytics worker
> ```

> [!success] Pros / Cons
> **Base62 ID — Pros:** no collisions, predictable length. **Cons:** exposes count, needs central ID generator. **Hash — Pros:** deterministic per URL. **Cons:** collision handling. **301 vs 302:** 301 reduces origin load (browser caches); 302 allows per-click analytics. **Overall trade-off:** optimise for read latency; writes are rare.

---

### 12. Rate limiter

> [!question] Q12
> Design a rate limiter (token bucket vs sliding window, distributed with Redis). (your CV: request queueing)

**Goal:** limit each user/IP/API key to N requests per time window; return **429 Too Many Requests** with `Retry-After` when exceeded.

**Token bucket:** Each key has a bucket with capacity `B` refilled at rate `R` tokens/sec. Each request consumes one token; if empty, reject. Allows **bursts** up to bucket size while enforcing average rate.

**Sliding window:** Count requests in the last `W` seconds (exact window using sorted set timestamps in Redis, or approximate with fixed windows). Smoother than naive fixed window (no double-count at boundary).

**Distributed:** Multiple app instances must share state — store counters in **Redis** with atomic `INCR` + `EXPIRE` or Lua scripts for check-and-decrement. Without shared state, each node has its own limit and effective allowance = N × nodes.

This mirrors your affiliate **Promise-based request queueing** — controlling concurrency/throughput — but at the API gateway layer. See [[07-cv-deep-dive]] and [[04-nestjs#27 Rate limiting]].

> [!example] Redis token bucket (conceptual)
> ```
> Key: ratelimit:user:42
> Fields: tokens=7, last_refill=1690000000
> On request: refill tokens by elapsed×R; if tokens≥1, decrement and allow; else 429
> ```
> Lua script ensures atomic read-modify-write across concurrent requests.

> [!success] Pros / Cons
> **Token bucket — Pros:** allows bursts, simple mental model. **Cons:** burst can still spike downstream. **Sliding window — Pros:** accurate rate over rolling period. **Cons:** more Redis memory/ops. **Redis-backed — Pros:** consistent across nodes. **Cons:** Redis SPOF (use cluster/sentinel), added latency per request (~1 ms). **App-level vs edge:** CDN/API gateway rate limits catch abuse earlier.

---

### 13. JWT authentication system

> [!question] Q13
> Design an authentication system with JWT access/refresh tokens. How do you handle refresh, rotation, and revocation? (your CV)

**Login flow:** User authenticates (password + [[09-security|bcrypt hash]]). Server issues:
- **Access token** — short-lived JWT (15 min), signed (HS256/RS256), contains `sub`, `roles`, `exp`. Sent as `Authorization: Bearer`. Stateless verification — no DB hit per request.
- **Refresh token** — long-lived opaque token or JWT (7–30 days), stored in **httpOnly Secure cookie** (mitigates XSS theft).

**Refresh flow:** Client sends refresh token to `/auth/refresh`. Server validates, issues **new access + new refresh token** (**rotation** — old refresh invalidated).

**Revocation:** Store a hash of the current valid refresh token (or session family ID) server-side (Redis/DB). On logout, password change, or theft detection, delete/invalidate the session. **Reuse detection:** if a rotated refresh token is presented again, revoke the entire session family (possible token theft).

**Security notes:** Never store access tokens in localStorage if avoidable; short access TTL limits exposure; RS256 lets microservices verify without shared secret. See [[09-security]] and your affiliate auth in [[07-cv-deep-dive]].

> [!example] Token lifecycle
> ```
> Login → access (15m) + refresh (7d, httpOnly cookie)
> API calls → Bearer access until 401
> Refresh → new pair; old refresh hash replaced in Redis
> Logout → delete refresh hash; client clears cookie
> Stolen refresh reused → revoke all sessions for user
> ```

> [!success] Pros / Cons
> **Pros:** fast auth (no session DB lookup), horizontal scale, clear separation of short/long credentials. **Cons:** JWT revocation before expiry is hard without blocklists; refresh rotation adds server state; key rotation requires care. **vs server sessions:** JWT scales better; sessions revoke instantly but need sticky/central store.

---

### 14. Notification system

> [!question] Q14
> Design a notification system (email/push, fan-out, retries, dedup).

**Flow:** API receives `SendNotification(userId, channel, template, payload)` → validate preferences (user opted out of SMS?) → persist notification record → enqueue to channel-specific queues (email, push, SMS).

**Workers** consume messages, render templates, call providers (SendGrid, FCM, Twilio). On failure: **exponential backoff** retry (1s, 2s, 4s…); after N failures → **dead-letter queue (DLQ)** for manual inspection.

**Fan-out:** One event ("new follower") may notify many users — publish to a fan-out service that creates per-user jobs (or use topic subscriptions). For millions of followers, batch or use a streaming approach.

**Dedup:** Idempotency key per logical notification (`order:123:shipped:email`) stored in DB/Redis with TTL — duplicate API calls do not send twice.

**Observability:** Track status per notification (queued → sent → delivered → failed). Rate-limit per user to prevent spam.

> [!example] Order shipped notification
> ```
> OrderService → NotificationAPI (idempotencyKey=order:456:shipped)
>      → MongoDB (audit log) → RabbitMQ `notifications.email`
>      → Email worker → SendGrid → webhook updates status to "delivered"
> Push parallel on `notifications.push` queue
> ```

> [!success] Pros / Cons
> **Pros:** decoupled from business logic, absorbs spikes, reliable delivery with retries. **Cons:** eventual delivery, provider rate limits, template/version management, compliance (unsubscribe, GDPR). **Queue vs direct send:** queue always wins at scale; direct calls block and fail loudly under load.

---

### 15. Pagination for large datasets

> [!question] Q15
> How would you design pagination for a large dataset API (offset vs cursor)?

**Offset pagination:** `GET /orders?offset=100000&limit=20` → SQL `OFFSET 100000 LIMIT 20` or MongoDB `.skip(100000).limit(20)`.

Simple for "page 5 of 10" UIs. **Problems at scale:** database must scan and discard 100k rows before returning 20 — O(offset) cost. **Unstable** if rows are inserted/deleted while paging (duplicates or skips).

**Cursor (keyset) pagination:** Sort by indexed field (`createdAt`, `_id`). Client sends `?cursor=eyJ...&limit=20` (opaque token encoding last seen value). Query: `WHERE createdAt < :cursor ORDER BY createdAt DESC LIMIT 20`.

Constant-time regardless of depth. Stable under concurrent inserts (new rows appear on page 1, not in the middle). Requires sort field to be unique or compound `(createdAt, _id)`.

> [!example] Affiliate commission list
> **Bad:** `skip(50000).limit(50)` on 2M documents — seconds of latency ([[16-mongodb]] COLLSCAN risk). **Good:** index `{ affiliateId: 1, createdAt: -1 }`, cursor = last `createdAt + _id`, query `{ affiliateId, createdAt: { $lt: cursor } }` — ~10 ms.

> [!success] Pros / Cons
> **Offset — Pros:** random page access, total count easy. **Cons:** slow deep pages, unstable. **Cursor — Pros:** fast, stable, infinite scroll friendly. **Cons:** no jump to page 47, opaque cursor, filtering + cursor needs careful index design. **Rule:** cursor for large/infinite lists; offset only for small admin tables.

---

### 16. Webhook delivery and consumption

> [!question] Q16
> Design a webhook delivery/consumption system with retries and idempotency. (your CV: webhooks)

**As consumer (receiving webhooks):**
1. Verify **HMAC signature** (`X-Signature`) with shared secret — reject forgeries ([[09-security]]).
2. Respond **2xx quickly** (< 3 s) — providers timeout and retry.
3. Enqueue payload for async processing — do not do heavy work in the handler.
4. **Idempotency:** store processed event IDs (`eventId` or hash); duplicates return 200 without reprocessing. Providers retry on failure — duplicates are normal.

**As producer (sending webhooks):**
1. Persist event before delivery attempt.
2. Deliver async with signed payload, timeout, retries + exponential backoff.
3. After N failures → DLQ; expose replay tooling.
4. Document ordering is **not guaranteed** — consumers must handle out-of-order events (use event timestamps/version).

Your Crisp/WebChat integration follows this pattern — see [[07-cv-deep-dive]].

> [!example] Crisp → WebChat webhook
> ```
> POST /webhooks/crisp
>   → verify HMAC
>   → check Redis SET processed_events:{id} — exists? return 200
>   → enqueue to RabbitMQ
>   → return 200
> Worker: process message, sync to MongoDB, mark event processed
> ```

> [!success] Pros / Cons
> **Pros:** reliable integration with third parties, async processing keeps endpoints fast, idempotency handles at-least-once delivery. **Cons:** signature verification bugs are security holes; DLQ maintenance; clock skew in ordering; debugging distributed retries is hard.

---

### 17. Idempotent payment/order endpoint

> [!question] Q17
> How do you design an idempotent payment/order endpoint?

**Problem:** Clients retry on timeout; load balancers replay; users double-click "Pay". Without idempotency, you charge twice or create duplicate orders.

**Solution:** Client sends **`Idempotency-Key`** header (UUID) with each payment/order request. Server logic:
1. Begin transaction or atomic upsert on `idempotency_keys` table: `(key, userId) → status, response, createdAt`.
2. If key exists and **completed** → return stored response (same HTTP status + body).
3. If key exists and **in_progress** → return 409 or wait (optional lock).
4. If new → process payment, store result, mark completed.

Combine with DB **unique constraints** (`UNIQUE(orderId)`, `UNIQUE(paymentIntentId)`) as a safety net. TTL old keys after 24–72 hours.

Payment providers (Stripe) have built-in idempotency keys — mirror the same pattern at your API layer.

> [!example] Checkout POST
> ```
> POST /orders  Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
> → INSERT idempotency_key (pending) — duplicate key → return cached 201 + order body
> → charge payment gateway
> → create order + decrement inventory (transaction)
> → UPDATE idempotency_key (completed, response_body)
> ```

> [!success] Pros / Cons
> **Pros:** safe retries, better UX, fewer support tickets, aligns with at-least-once queues ([[#25 Exactly-once vs at-least-once]]). **Cons:** storage for keys, client must generate stable keys, in-progress race needs locking, does not replace full distributed transaction if multiple services involved — consider [[#28 Dual-write and outbox pattern]].

---

### 18. Caching a read-heavy API

> [!question] Q18
> How would you add a caching layer to a read-heavy API and keep it consistent? (your CV: Redis)

**Pattern: cache-aside (lazy loading)**
1. Read: check Redis for key `entity:123`.
2. Hit → return cached value.
3. Miss → query [[16-mongodb|MongoDB]], write to Redis with TTL, return.
4. Write/update/delete: update DB, then **delete** cache key (not update — avoids race where another request caches stale data between your DB write and cache update).

**Consistency tactics:**
- **TTL as safety net** — even if invalidation misses, data expires (5–60 min depending on tolerance).
- **Jittered TTLs** — prevent thundering herd when many keys expire together.
- **Stampede lock** — on miss, one request rebuilds while others wait or serve stale ([[05-databases#19 Cache stampede]]).
- **Multi-layer:** CDN for public GETs; Redis for personalised/computed data.

Your WebChat and affiliate reporting used this to cut DB load and improve read latency — see [[07-cv-deep-dive]].

> [!example] Product detail API
> ```
> GET /products/99
>   → Redis GET product:99 — hit → 5 ms
>   → miss → MongoDB findById → SET product:99 EX 600 → 40 ms
> Admin PATCH /products/99
>   → update MongoDB → DEL product:99
> Next GET repopulates fresh data
> ```

> [!success] Pros / Cons
> **Pros:** large read latency reduction, protects DB under flash traffic, simple to implement. **Cons:** stale reads between write and invalidation, cache/DB mismatch bugs, memory limits, cold-start after deploy. **Write-through:** stronger consistency but slower writes. **Event-driven invalidation:** pub/sub on writes — more complex, tighter consistency.

---

## Advanced (mapped to your CV projects)

### 19. Affiliate/referral system

> [!question] Q19
> Design an affiliate/referral system: link generation, click tracking, attribution, commission calculation, fraud prevention, at 100k+ affiliates and 600+ daily orders. (your CV: Rokomari Affiliate)

> [!warning] CV template
> Adapt every detail to your real Affiliate implementation. Interviewers will ask about indexes, queue configs, and fraud rules you actually shipped. Full STAR stories: [[07-cv-deep-dive]].

**Requirements clarification:** 100k+ affiliates, 600+ daily orders (higher at peak), unique referral links, click → order attribution, commission accrual, payout reporting, fraud resistance, sub-second dashboards for affiliates.

**Link generation:** Each affiliate gets a unique `refCode`. Product URLs append `?ref=CODE` or path prefix `/r/CODE/product-slug`. Store mapping in [[16-mongodb|MongoDB]]: `{ affiliateId, refCode, status, createdAt }`. Index on `refCode` for lookups.

**Click tracking:** Lightweight redirect endpoint `GET /r/:code` → log click asynchronously (enqueue to RabbitMQ — do not block redirect) → set attribution cookie (`aff_ref`, 30-day window) → 302 to product page. Store: `{ clickId, affiliateId, timestamp, ip, userAgent, landingUrl }`.

**Attribution:** On order creation, read `aff_ref` cookie (or last-click within window). Policy: **last-click** (most common) or first-click. Persist `{ orderId, affiliateId, attributedAt, model }` atomically with the order.

**Commission calculation:** Async worker consumes `orders.created` events. **Idempotent** upsert keyed by `orderId` — replays never double-credit. Rate × order amount → immutable commission record with audit fields (rate, amount, currency, timestamp). Use **computed/pre-aggregated** daily rollups for fast reports (`{ affiliateId, date, totalCommission, orderCount }`).

**Scale & performance:** Compound indexes ESR on `{ affiliateId, status, createdAt }` for report queries (your 37s → 1s optimisation). Cache hot dashboards in Redis. Heavy aggregation off the request path via [[04-nestjs|NestJS]] workers + RabbitMQ.

**Fraud prevention:** Block self-referral (affiliate email = buyer email), rate-limit clicks per IP, flag abnormal conversion rates, hold commissions for review period, device fingerprinting for click spam.

> [!example] End-to-end flow
> ```
> Affiliate dashboard → generate link ?ref=ALI42
> User clicks → /r/ALI42 → queue: clicks → cookie set → product page
> User orders → order service reads cookie → attributes to ALI42
> → queue: orders.created → commission worker → MongoDB commission doc
> → invalidate Redis report cache → affiliate sees updated earnings
> ```

> [!success] Pros / Cons
> **Pros:** async pipeline keeps checkout fast; idempotent commissions safe under retries; indexed + cached reports scale to 100k affiliates. **Cons:** eventual consistency in dashboards; attribution cookie loss (Safari ITP) under-credits; fraud rules need tuning; last-click vs first-click business disputes. **MongoDB vs SQL:** flexible schema for evolving commission rules; use transactions for payout batches if moving money.

> [!tip] Numbers to mention
> 600+ daily orders, 13% sales lift, query 37s → 1s via compound index — from [[07-cv-deep-dive]].

---

### 20. Real-time chat / customer support

> [!question] Q20
> Design a real-time chat / customer support system (WebSocket connections, presence, message delivery guarantees, scaling across servers with Redis pub/sub, persistence, bot replies). (your CV: WebChat)

> [!warning] CV template
> Ground this in your actual WebChat/Crisp integration — connection counts, Redis adapter choice, and bot rules you deployed. See [[07-cv-deep-dive#24 WebChat architecture]].

**Components:** [[04-nestjs|NestJS]] WebSocket gateway (Socket.IO), MongoDB for message persistence, Redis for cache/presence/pub/sub, RabbitMQ for async tasks, Crisp (or similar) for agent tooling.

**Connection flow:** Client connects → authenticate JWT on handshake → join room `conversation:{id}`. Messages sent over WebSocket with client-generated temp IDs for optimistic UI.

**Multi-server scaling:** Users land on different Node instances. Use **Redis pub/sub adapter** (Socket.IO Redis adapter): message received on server A is published to Redis channel → all servers deliver to their local sockets in that room.

**Delivery guarantees:** Persist message to MongoDB **before or while** broadcasting (write-then-publish or transactional outbox). Track status: `sent → delivered → read` via client ACKs. Offline users catch up on reconnect from DB cursor.

**Presence & typing:** Redis keys `presence:user:{id}` with TTL heartbeat; `typing:conversation:{id}` expires in 3 s.

**Bot replies:** RabbitMQ consumer matches intents/keywords → instant automated reply; unknown queries escalate to human agent via Crisp sync.

**Webhooks:** Crisp events ingested idempotently ([[#16 Webhook delivery and consumption]]).

> [!example] Message path
> ```
> Client WS → Gateway (Node 2) → save MongoDB → Redis PUBLISH room:conv:99
>     → all nodes subscribed → clients in room receive message
> Parallel: enqueue bot-check → worker replies if FAQ match
> ```

> [!success] Pros / Cons
> **Pros:** sub-second messaging, horizontal scale via Redis backplane, reliable async side effects via queue, 78% satisfaction metric from your CV. **Cons:** WebSocket ops complexity, ordering across reconnects, Redis pub/sub is fire-and-forget (not durable — persistence is in MongoDB), Crisp sync lag, memory per connection limits node capacity.

---

### 21. Ad server

> [!question] Q21
> Design an ad server: ad selection, budget-based prioritization, pacing, impression/click tracking, and analytics at scale. (your CV: Rokomari Ads)

> [!warning] CV template
> Tie ad selection rules and budget logic to what you actually built in Rokomari Ads. See [[07-cv-deep-dive#13 Three ad features]].

**Ad request:** Publisher page requests slot `homepage_banner` with context (category, geo, device). Latency budget: **< 50 ms** for selection.

**Selection pipeline:**
1. Filter **eligible ads** — active campaign, targeting match (category, geo), budget remaining > 0, schedule window.
2. **Rank** by score combining bid, campaign priority, historical CTR (explore/exploit), and pacing factor.
3. Return winner creative URL + tracking pixel URLs.

**Budget prioritization & pacing:** Each campaign has total budget and daily cap. Track spend with atomic `$inc` on `campaign.remainingBudget`. **Pacing** spreads daily budget evenly (`expectedSpendByNow = dailyCap × hoursElapsed/24`) — under-paced campaigns get boosted; over-paced get deprioritised. Stop serving when budget hits zero.

**Tracking:** Impression/click endpoints are **fire-and-forget** — enqueue events, never block ad render. High write volume → RabbitMQ/Kafka → batch writers + aggregate counters by `{ adId, hour }`.

**Analytics:** Stream/batch aggregation into reporting collections; serve dashboards from precomputed rollups + Redis cache. Raw events in cheap storage for replay.

**Scale:** Cache eligible ad lists per slot/context in Redis (short TTL). Precompute targeting inverted indexes. Separate read-heavy serving path from write-heavy analytics path (CQRS).

> [!example] Serve + track
> ```
> GET /ads/serve?slot=sidebar&cat=books
>   → Redis cache candidate set → rank → return ad JSON (< 30 ms)
> Browser loads `<img src="/track/impression?ad=7&sig=...">` (1×1)
> Click → /track/click?ad=7 → queue → aggregate pipeline → dashboard
> ```

> [!success] Pros / Cons
> **Pros:** async tracking never slows delivery; pacing prevents budget blowout in hour one; atomic counters prevent overspend race. **Cons:** real-time budget is eventually consistent at extreme QPS; fraud clicks inflate metrics; cold-start ads need exploration traffic; ranking complexity grows quickly.

---

### 22. E-commerce backend

> [!question] Q22
> Design the backend for a high-traffic e-commerce site (catalog, cart, checkout, inventory, order processing, handling flash sales). (your CV: Rokomari)

> [!warning] CV template
> Reference your Rokomari Next.js migration, load testing, and performance numbers honestly. See [[07-cv-deep-dive#17 Incremental migration]].

**Service modules:** Catalog, Cart, Checkout/Orders, Inventory, Payments, Users — can be modules in one [[04-nestjs|NestJS]] monolith initially, split later by bottleneck.

**Catalog (read-heavy):** Product docs in [[16-mongodb|MongoDB]]; serve via Next.js SSG/ISR + CDN ([[#5 CDN]]); Redis for hot products and category trees. Search via Elasticsearch/Atlas Search optional.

**Cart:** Redis hash per `userId` or session for speed; periodic sync to DB for logged-in users. TTL abandoned carts.

**Checkout/orders:** Synchronous path: validate cart → reserve inventory → charge payment ([[#17 Idempotent payment/order endpoint]]) → create order → publish `OrderCreated`. Idempotency keys mandatory.

**Inventory:** **Atomic decrement** — MongoDB `findOneAndUpdate({ sku, stock: { $gte: qty } }, { $inc: { stock: -qty } })` or reserved stock during checkout timeout. Never oversell.

**Flash sales:** Pre-scale nodes, aggressive caching, queue order creation if needed, rate limit add-to-cart, load test beforehand (your k6/Artillery experience). Consider virtual waiting room for extreme drops.

**Order processing (async):** RabbitMQ workers for email, commission, analytics, warehouse — [[#23 Reliable order processing with queues]].

> [!example] Flash sale architecture
> ```
> CDN/ISR → product page (static shell)
> Add to cart → Redis cart + rate limit
> Checkout → inventory atomic dec → payment → order DB → queue
> Workers → fulfillment, affiliate, notifications
> Monitor: queue depth, p99 checkout latency, inventory oversell alerts
> ```

> [!success] Pros / Cons
> **Pros:** separates fast checkout from slow fulfillment; CDN + ISR handles catalog thundering herd; idempotent orders safe under retries. **Cons:** inventory reservation timeouts add complexity; flash sales expose every weak link; monolith vs microservices trade-off — start modular monolith. **Consistency:** strong for payment/inventory; eventual for analytics OK.

---

### 23. Reliable order processing with queues

> [!question] Q23
> How would you design a system to process 600+ daily orders reliably with eventual reporting, using queues? (your CV)

> [!warning] CV template
> Use your actual RabbitMQ exchange/queue names and retry config if asked. Story: [[07-cv-deep-dive#8 600+ daily orders reliably]].

**Goal:** Order API stays fast (< 200 ms); downstream work (commission, email, reporting, warehouse) completes reliably even if workers crash or spike to 10× volume.

**Pattern:**
1. **Accept order** — validate, persist to MongoDB with status `confirmed`, within transaction where needed.
2. **Publish event** — `orders.created` to RabbitMQ with durable queue and persistent messages. Prefer [[#28 Dual-write and outbox pattern]] if publish must match DB commit.
3. **Workers consume** — separate queues per concern (commission, notifications, reporting) for independent scaling and failure domains.
4. **Retries** — transient failures (DB timeout, email provider 503) retry with exponential backoff.
5. **Dead-letter queue** — after N failures, message moves to DLQ; alert ops; manual replay tool.
6. **Idempotent consumers** — dedup by `orderId` ([[#25 Exactly-once vs at-least-once]]); upsert, not blind insert.

**Reporting:** Eventually consistent — batch/computed aggregates updated by worker every minute or on event. Affiliates see near-real-time; finance reconciliation runs nightly against source orders.

> [!example] Queue topology
> ```
> orders.created (topic exchange)
>   ├─ queue: commissions   → CommissionWorker
>   ├─ queue: notifications → EmailWorker
>   └─ queue: reporting     → AggregateWorker → daily rollup docs
> Each queue: x-dead-letter-exchange → DLQ
> ```

> [!success] Pros / Cons
> **Pros:** order path decoupled from slow workers; natural backpressure; retries + DLQ give operational safety; handles 600+/day with headroom for 10× peaks. **Cons:** reporting lags seconds/minutes; duplicate messages require idempotent design; queue monitoring becomes critical; ordering not guaranteed across queues.

---

### 24. Scaling WebSockets to millions

> [!question] Q24
> How would you scale WebSocket connections to millions of concurrent users?

> [!warning] CV template
> Extend your WebChat Redis pub/sub story with honest connection counts and node sizing — do not claim millions unless you load-tested it. See [[07-cv-deep-dive#24 WebChat architecture]].

**Challenge:** Each connection consumes memory (typically 10–50 KB per socket depending on buffers) and file descriptors on a server. One machine tops out at ~100k–500k connections depending on hardware.

**Architecture:**
1. **Many stateless WebSocket gateway nodes** — autoscale on connection count / CPU.
2. **Load balancer with TCP/WebSocket support** — L7 upgrade; optional stickiness (connection stays on one node for its lifetime).
3. **Pub/sub backplane** — Redis Pub/Sub for moderate scale; Redis Cluster; at very high scale consider dedicated message bus (NATS, Kafka) or managed services (Ably, Pusher).
4. **Externalise state** — presence, session metadata, room membership in Redis; not in process memory alone.
5. **Efficient protocol** — binary frames (MessagePack/Protobuf), heartbeat tuning, compress only if CPU allows.
6. **Reconnection strategy** — client exponential backoff; resume token; sync missed messages from DB on reconnect.
7. **Geographic distribution** — edge POPs or regional clusters to reduce latency.

Your WebChat uses Redis pub/sub at smaller scale; same pattern extends with more nodes and connection sharding.

> [!example] Million-connection layout
> ```
> Global LB → Region LB → WS nodes (50k each × 20 = 1M)
>                ↕
>           Redis Cluster (pub/sub + presence)
>                ↕
>           MongoDB (message log)
> Autoscale: CPU > 60% OR connections > 40k → add node
> ```

> [!success] Pros / Cons
> **Pros:** linear horizontal growth, regional isolation limits blast radius. **Cons:** sticky connection complicates deploys (graceful drain); Redis pub/sub not durable; cross-region fan-out adds latency; debugging connection leaks is hard; millions × memory = significant infra cost. **Managed real-time SaaS:** faster to ship, less ops, vendor cost.

---

### 25. Exactly-once vs at-least-once processing

> [!question] Q25
> How do you ensure exactly-once / at-least-once processing with a message queue? How do you handle duplicate messages?

> [!warning] CV template
> This pattern underpins your Affiliate commission workers and order processing — be ready to describe your actual dedup keys (e.g. `orderId`). See [[07-cv-deep-dive#8 600+ daily orders reliably]].

**At-least-once delivery:** Broker delivers message → consumer processes → consumer **ACKs**. If worker crashes before ACK, message redelivered. **Duplicates are expected.**

**Exactly-once** end-to-end is extremely hard in distributed systems (requires distributed transactions across broker + DB). Practical approach: **at-least-once + idempotent consumers = effectively-once**.

**Idempotent consumer patterns:**
1. **Dedup table** — `processed_messages(messageId)` insert before work; duplicate insert fails → skip.
2. **Business-key upsert** — `commission.updateOne({ orderId }, ..., { upsert: true })` — same order never double-credits.
3. **Unique DB constraints** — `UNIQUE(idempotency_key)` catches duplicates at storage layer.
4. **Idempotent API design** — [[#17 Idempotent payment/order endpoint]].

**Ack timing:** ACK **only after** successful processing + durable write. Premature ACK loses message on crash; late ACK causes redelivery (safe if idempotent).

**Kafka extras:** Idempotent producer (`enable.idempotence=true`), transactional writes for exactly-once *within Kafka* — still need idempotent sinks for external DB.

> [!example] Commission worker duplicate
> ```
> Message orderId=789 arrives (1st time) → upsert commission → ACK
> Same message redelivered (worker died pre-ACK) → upsert same orderId → no op → ACK
> Result: one commission row, correct balance
> ```

> [!success] Pros / Cons
> **At-least-once — Pros:** simple broker semantics, no message loss, industry default. **Cons:** duplicates without idempotency cause bugs (double charge). **True exactly-once — Pros:** no dedup logic. **Cons:** complex, limited scope, performance cost. **Recommendation:** design for at-least-once; make every consumer idempotent by default.

---

### 26. Click/impression tracking and analytics pipeline

> [!question] Q26
> Design a click/impression tracking + analytics pipeline (high write throughput, aggregation, near-real-time dashboards). (your CV: ads analytics, GA)

> [!warning] CV template
> Connect to your GA/Pixel accuracy work and Rokomari Ads analytics. See [[07-cv-deep-dive#2 Improved GA/Facebook Pixel data accuracy]].

**Write path (high throughput):** Tracking pixel/beacon `GET /t/i?ad=7&...` returns 1×1 GIF immediately. Server validates signature, enriches (timestamp, geo), **appends to queue** (Kafka/RabbitMQ) — never synchronous DB write on hot path. Target: **100k+ events/sec** with horizontal collectors.

**Processing layer:**
- **Stream consumers** batch events (500 ms or 1000 records) → write raw to cheap storage (S3, MongoDB time-series, ClickHouse).
- **Aggregation jobs** roll up into buckets: `{ adId, campaignId, hour } → impressions, clicks, spend`.
- **Lambda architecture option:** speed layer (Redis counters for last hour) + batch layer (nightly correct totals).

**Read path (dashboards):** Query pre-aggregated tables, not raw events. Cache dashboard API in Redis. Near-real-time = 1–5 minute delay acceptable for ops; sub-second for pacing decisions uses Redis counters.

**Data quality:** Dedup via event UUID; bot filtering; clock skew tolerance; reconcile raw vs aggregates nightly.

> [!example] Pipeline
> ```
> Browser pixel → Collector LB → Kafka topic `ad.events`
>     → Stream worker → ClickHouse (raw) + Redis INCR (live counters)
>     → Hourly rollup job → MongoDB `analytics_daily`
> Dashboard API → Redis (last hour) + MongoDB (history)
> ```

> [!success] Pros / Cons
> **Pros:** serving path never blocked by analytics; horizontal scale on Kafka; pre-aggregation makes dashboards fast. **Cons:** eventual consistency in reports; raw storage cost; duplicate/bot inflation without dedup; GDPR/privacy on IP/user tracking ([[09-security]]).

---

### 27. Commission calculation (accurate, auditable, idempotent)

> [!question] Q27
> How would you design commission calculation to be accurate, auditable, and idempotent?

> [!warning] CV template
> Align rates, hold periods, and payout rules with your Affiliate platform. See [[07-cv-deep-dive#9 Database structure & workflows]].

**Trigger:** Confirmed order event (`orders.created` or `orders.paid`) from queue — not on click or cart add.

**Calculation worker:**
1. Load order + attribution record + affiliate tier/rate rules.
2. Compute `commission = eligibleAmount × rate` (exclude tax/shipping per policy).
3. **Idempotent write:** `upsert({ orderId }, { affiliateId, amount, rate, inputs, calculatedAt })` — replays overwrite same result, never add twice.
4. Update running balance or emit `commission.accrued` event for downstream payout service.

**Auditability:** Commission records are **immutable** — store full inputs snapshot `{ orderId, orderTotal, eligibleAmount, rate, ruleVersion, affiliateId }`. Corrections are **adjustment records** (negative/positive delta with reason), never silent edits. Append-only ledger pattern.

**Reconciliation:** Nightly job compares sum(commissions) vs sum(attributed orders × rate); flags discrepancies. Payout batch uses locked period (commissions before cutoff only).

**Transactions:** Where balance + commission insert must be atomic, use [[16-mongodb|MongoDB sessions]] (replica set required) or single-document embedding.

> [!example] Audit trail
> ```
> Order 1001 → Commission C-1001: +৳50 (rate 5%, rule v3)
> Refund on 1001 → Adjustment A-1001-R: -৳50 (reason: full refund)
> Payout batch PB-2026-07 → pays net of adjustments
> Auditor queries: all rows for affiliate 42 in July
> ```

> [!success] Pros / Cons
> **Pros:** idempotent by orderId prevents double-pay; immutable audit satisfies finance; rule versioning explains historical rate changes. **Cons:** adjustment workflow adds complexity; holding period delays affiliate cashflow; refund/chargeback handling must be explicit; floating-point money — use integer minor units (poisha/cents).

---

### 28. Dual-write problem and outbox pattern

> [!question] Q28
> How do you handle the dual-write problem (DB + message queue) and ensure consistency (outbox pattern)?

**Dual-write problem:** Order service saves order to MongoDB, then publishes to RabbitMQ. If DB commit succeeds but publish fails (network blip), downstream never processes the order. If publish succeeds but DB rolls back, ghost events appear. Two separate systems cannot share one transaction.

**Outbox pattern (recommended):**
1. In the **same DB transaction** as the business write, insert a row into an **`outbox`** collection/table: `{ eventId, aggregateType, aggregateId, payload, createdAt, published: false }`.
2. Transaction commits → both order and outbox row are durable together.
3. **Relay process** (polling worker or Change Stream / CDC) reads unpublished outbox rows, publishes to queue, marks `published: true` (or deletes row).
4. Consumers process as usual (still idempotent).

**Variants:** MongoDB **Change Streams** watch outbox inserts and trigger relay without polling. Debezium CDC for SQL.

**At-least-once publish:** Relay may publish twice (crash after publish, before mark) — consumers remain idempotent ([[#25 Exactly-once vs at-least-once]]).

This is the production-grade fix for [[#23 Reliable order processing with queues]] and affiliate order events.

> [!example] Outbox flow
> ```
> BEGIN TRANSACTION
>   INSERT orders ...
>   INSERT outbox { type: 'OrderCreated', orderId: 789 }
> COMMIT
> OutboxRelay (every 100ms):
>   SELECT * FROM outbox WHERE published=false LIMIT 100
>   → publish to RabbitMQ
>   → UPDATE published=true
> ```

> [!success] Pros / Cons
> **Pros:** atomicity between state change and event intent; no lost events on publish failure; works with any broker. **Cons:** extra table and relay service; slight publish latency (poll interval); relay is a component to monitor; does not give exactly-once delivery alone — still need idempotent consumers. **Alternatives:** Saga orchestration for multi-service; transactional outbox vs two-phase commit (2PC) — 2PC is rare in modern microservices due to availability cost.

> [!info] When to introduce
> Start with simple publish-after-write for low-criticality events. Adopt outbox when lost events cause money or compliance bugs (orders, commissions, payments).

---

## Related notes

- [[07-cv-deep-dive]] — STAR stories for Affiliate, WebChat, Ads, and Next.js migration tied to Q19–28.
- [[05-databases]] — CAP, ACID, replication, sharding, caching strategies, Redis data structures.
- [[16-mongodb]] — indexes (ESR), embed vs reference, transactions, query optimisation (37s → 1s pattern).
- [[04-nestjs]] — modules, guards, WebSocket gateways, JWT auth, rate limiting, queue integration.
- [[09-security]] — JWT hardening, webhook HMAC verification, bcrypt, XSS/CSRF considerations.
- [[10-networking-http]] — HTTP caching headers, status codes, TLS.
- [[11-web-performance]] — CDN, lazy loading, Core Web Vitals (pairs with e-commerce design).
- [[14-docker-deployment]] — container scaling and deploy strategies for horizontally scaled services.

---

## References & Further Study

- [System Design Primer (GitHub)](https://github.com/donnemartin/system-design-primer) — comprehensive free resource with trade-off discussions and example diagrams.
- [Designing Data-Intensive Applications (Kleppmann)](https://dataintensive.net/) — the definitive book on replication, partitioning, streams, and consistency.
- [Redis documentation](https://redis.io/docs/) — data structures, pub/sub, persistence (RDB/AOF), cluster mode.
- [RabbitMQ tutorials](https://www.rabbitmq.com/tutorials) — exchanges, queues, acks, dead-letter exchanges.
- [Apache Kafka documentation](https://kafka.apache.org/documentation/) — log-based messaging, consumer groups, idempotent producers.
- [AWS Architecture Blog — Cache strategies](https://aws.amazon.com/caching/best-practices/) — cache-aside, write-through, invalidation patterns.
- [Martin Fowler — Patterns of Distributed Systems](https://martinfowler.com/articles/patterns-of-distributed-systems/) — outbox, event sourcing, sagas.
- [Socket.IO Redis adapter](https://socket.io/docs/v4/redis-adapter/) — scaling WebSocket servers with pub/sub backplane.
- [Stripe — Idempotent requests](https://stripe.com/docs/api/idempotent_requests) — industry reference for payment idempotency.
- [Google SRE Book — Handling overload](https://sre.google/sre-book/handling-overload/) — load shedding, queuing, and graceful degradation.
