---
title: Databases (General Concepts) — Answers
topic: databases
tags: [interview, fullstack, databases, redis, caching]
related: ["[[16-mongodb]]", "[[15-sql]]", "[[06-system-design]]"]
---

> [!abstract] Overview
> This note covers **general database and caching concepts** for full-stack interviews: SQL vs NoSQL, indexes, transactions, CAP, replication, sharding, and Redis caching patterns. MongoDB-specific depth lives in [[16-mongodb]]; SQL query and schema depth lives in [[15-sql]]. System-design trade-offs tie into [[06-system-design]].

---

## Beginner

### 1. SQL vs NoSQL

> [!question] Q1
> What is the difference between SQL and NoSQL databases? When would you choose each? (commonly asked at Amazon, Microsoft)

In plain terms: **SQL databases** (also called **relational databases**) store data in **tables** with rows and columns. The structure is defined by a **schema** — you decide column names and types up front. **NoSQL databases** use other models (documents, key-value pairs, graphs, etc.) and usually allow more flexible shapes of data.

**SQL** is strong at **relationships** (linking orders to customers) and **ACID transactions** — a group of changes that either all succeed or all fail. Examples: PostgreSQL, MySQL.

**NoSQL** is often chosen for **flexible schemas**, **horizontal scaling** (adding more servers), and **specific access patterns** (fast key lookups, large documents). Examples: MongoDB (document), Redis (key-value).

Choose **SQL** when data is structured, relationships matter, and you need strong consistency — e.g. orders, payments, inventory. Choose **NoSQL** when the schema evolves often, you need massive scale, or one database type fits the access pattern better — e.g. product catalogs, session storage, real-time feeds.

> [!example]
> **E-commerce:** Store **orders and payments** in PostgreSQL (SQL) — every line item must match the order total, and money must never be half-updated. Store **product descriptions** in MongoDB (NoSQL) — each product can have different optional fields (size chart, ingredients, specs) without altering a rigid table.

> [!success] Pros / Cons
> **SQL pros:** Strong consistency, joins across tables, mature tooling, ACID transactions.  
> **SQL cons:** Harder to scale writes horizontally; schema changes need migrations.  
> **NoSQL pros:** Flexible schema, horizontal scaling, optimized for specific patterns.  
> **NoSQL cons:** Weaker default consistency guarantees; fewer built-in join features; app must enforce more rules.

> [!tip] Interview tip
> Amazon and Microsoft often want a **concrete feature**, not a generic "SQL is better." Say: "Orders → SQL; catalog → document DB; cache → Redis." Mention **polyglot persistence** — using the right store per job.

> [!info] Further study
> - [PostgreSQL documentation](https://www.postgresql.org/docs/)
> - [MongoDB use cases](https://www.mongodb.com/use-cases)

---

### 2. NoSQL database types

> [!question] Q2
> What are the main types of NoSQL databases (document, key-value, column-family, graph)?

In plain terms: NoSQL is an umbrella term. Different **types** optimize for different ways of reading and writing data.

| Type | Idea | Examples | Best for |
|------|------|----------|----------|
| **Document** | JSON-like documents in collections | MongoDB | Flexible records, catalogs, content |
| **Key-value** | Simple lookup: key → value | Redis, DynamoDB | Cache, sessions, config |
| **Column-family** | Wide rows, columns grouped in families | Cassandra, HBase | Huge write-heavy analytics |
| **Graph** | Nodes and edges (relationships first) | Neo4j | Social networks, recommendations, fraud paths |

Pick the type that matches how your app **reads** data most often.

> [!example]
> - **Document:** A `products` collection where each document holds name, price, and optional nested `reviews`.  
> - **Key-value:** `session:abc123` → `{ userId: 42, role: "admin" }` in Redis.  
> - **Column-family:** Time-series sensor data — billions of writes per day, queried by device + time range.  
> - **Graph:** "Friends of friends who bought this product" — traverse edges, not join many SQL tables.

> [!success] Pros / Cons
> **Document:** Flexible schema; watch out for unbounded document growth.  
> **Key-value:** Extremely fast; no rich queries on values.  
> **Column-family:** Scales writes well; less friendly for ad-hoc queries.  
> **Graph:** Natural for connected data; not ideal as a general-purpose store.

> [!info] Further study
> - [MongoDB — document model](https://www.mongodb.com/docs/manual/core/document-databases/)
> - [Redis data types](https://redis.io/docs/latest/develop/data-types/)

---

### 3. Database indexes

> [!question] Q3
> What is an index and why does it speed up reads? What is the cost of an index?

In plain terms: An **index** is a separate lookup structure — often a **B-tree** — that maps column or field values to where rows live in storage. Without an index, the database does a **full table scan**: it reads every row to find matches. With an index, it jumps to matching rows in roughly **O(log n)** time instead of **O(n)**.

Think of a book index: you look up a word and go straight to the page instead of reading every page.

**Cost of indexes:**
- **Extra storage** — the index structure takes disk/memory.
- **Slower writes** — every INSERT, UPDATE, or DELETE must update the index too.
- **Wrong indexes hurt** — indexing every column wastes space and slows writes without helping reads.

Index columns you **filter**, **sort**, or **join** on in real queries. See [[15-sql]] for index design in SQL.

> [!example]
> ```sql
> -- Without index on email: scans all users
> SELECT * FROM users WHERE email = 'alice@example.com';

> -- With index on email: fast lookup
> CREATE INDEX idx_users_email ON users(email);
> ```

> [!success] Pros / Cons
> **Pros:** Much faster reads, sorts, and joins on indexed columns.  
> **Cons:** More storage, slower writes, need maintenance and monitoring.

> [!tip] Interview tip
> If asked "when not to index," say: small tables, columns rarely used in WHERE/ORDER BY, or tables with very heavy writes and few reads.

---

### 4. Primary key vs foreign key

> [!question] Q4
> What is a primary key vs a foreign key?

In plain terms:

- A **primary key (PK)** uniquely identifies **one row** in a table. Rules: unique, not null, one per table (or composite — multiple columns together). Example: `user_id = 42`.
- A **foreign key (FK)** is a column (or set) that **points to** a primary key in **another table**. It enforces **referential integrity** — you cannot reference a row that does not exist.

Together they model **relationships**: "This order belongs to this user."

> [!example]
> ```sql
> CREATE TABLE users (
>   id         INT PRIMARY KEY,
>   email      VARCHAR(255) UNIQUE
> );

> CREATE TABLE orders (
>   id         INT PRIMARY KEY,
>   user_id    INT NOT NULL,
>   total      DECIMAL(10,2),
>   FOREIGN KEY (user_id) REFERENCES users(id)
> );
> -- Cannot insert order with user_id = 999 if user 999 does not exist.
> ```

> [!success] Pros / Cons
> **Primary keys:** Guarantee uniqueness; enable fast joins and lookups.  
> **Foreign keys:** Protect data integrity; add cost on deletes/updates (cascade rules matter).

> [!info] Further study
> - [PostgreSQL — constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)

---

### 5. Normalization vs denormalization

> [!question] Q5
> What is normalization and denormalization? Give a trade-off.

In plain terms:

- **Normalization** splits data into related tables so each fact is stored **once**. This removes **redundancy** (duplicate data) and **update anomalies** (changing data in one place but forgetting another).
- **Denormalization** intentionally **duplicates** data across tables or documents to **avoid joins** and speed up reads.

**Trade-off:** Normalized data is easier to keep **consistent** but reads may need many joins. Denormalized data reads faster but you must update copies in multiple places — risk of **stale or inconsistent** duplicates.

> [!example]
> **Normalized:** `users` table + `orders` table linked by `user_id`. Change a user's email in one place.  
> **Denormalized:** Each order document embeds `{ customerName, customerEmail }` for a fast order history page — but if the user changes email, old orders still show the old email unless you update them too.

> [!success] Pros / Cons
> **Normalization — strengths:** Data integrity, single source of truth. **Watch-outs:** More joins, slower read-heavy dashboards.  
> **Denormalization — strengths:** Fast reads, simpler queries. **Watch-outs:** Duplicate updates, inconsistency.

> [!tip] Interview tip
> OLTP (online transactions) tends **normalized**; analytics and read-heavy APIs often **denormalize** or use materialized views.

---

### 6. Transactions and ACID

> [!question] Q6
> What is a transaction? What does ACID stand for?

In plain terms: A **transaction** is a group of database operations treated as **one unit** — either **all succeed** or **all are rolled back** (undone). Used when partial updates would be dangerous (transfer money: debit one account, credit another — both or neither).

**ACID** describes guarantees:

| Letter | Meaning | Plain English |
|--------|---------|---------------|
| **A** — Atomicity | All-or-nothing | No half-finished work |
| **C** — Consistency | Valid state → valid state | Rules and constraints hold after commit |
| **I** — Isolation | Concurrent transactions don't corrupt each other | Your work isn't mixed up with others mid-flight |
| **D** — Durability | Committed data survives crashes | After "success," data is on disk (or equivalent) |

> [!example]
> ```sql
> BEGIN;
>   UPDATE accounts SET balance = balance - 100 WHERE id = 1;
>   UPDATE accounts SET balance = balance + 100 WHERE id = 2;
> COMMIT;  -- both apply
> -- If anything fails: ROLLBACK; -- neither applies
> ```

> [!success] Pros / Cons
> **Strengths:** Safe money, inventory, and multi-step workflows.  
> **Watch-outs:** Higher isolation = more locking and lower concurrency; distributed transactions are harder.

> [!info] Further study
> - [PostgreSQL — transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html)

---

### 7. Schema and schema-less

> [!question] Q7
> What is a database schema? What does schema-less mean for document databases?

In plain terms: A **schema** is the **structure** of your data — table names, column names, data types, and constraints (required fields, max length, etc.).

**Relational databases** enforce schema at the database level: you cannot insert a string into an integer column.

**Document databases** (e.g. MongoDB) are often called **schema-less** or **schemaless** — the database does not force every document in a collection to have the same fields. One product can have `color`; another can have `weightKg`.

In practice, apps still follow an **implicit schema** in code (validation libraries like Mongoose, TypeScript types). "Schema-less" means **flexibility to evolve** without running SQL migrations for every new optional field — not that structure does not matter. See [[16-mongodb]] for schema design patterns.

> [!example]
> ```javascript
> // Same collection, different shapes — allowed in MongoDB
> { "_id": 1, "name": "T-Shirt", "size": "M" }
> { "_id": 2, "name": "Laptop", "specs": { "ramGb": 16 } }
> ```

> [!success] Pros / Cons
> **Schema-less strengths:** Fast iteration, optional fields per record.  
> **Watch-outs:** Inconsistent data if the app does not validate; harder reporting without discipline.

---

## Intermediate

### 8. CAP theorem

> [!question] Q8
> Explain the CAP theorem. Where do MongoDB and PostgreSQL sit? (commonly asked at Amazon)

In plain terms: **CAP** applies to **distributed** systems — multiple nodes talking over a network.

During a **network partition** (nodes cannot all talk to each other), you can fully guarantee only **two of three**:

| Property | Meaning |
|----------|---------|
| **C** — Consistency | Every read sees the latest write |
| **A** — Availability | Every request gets a response (not an error) |
| **P** — Partition tolerance | System keeps working when the network splits |

Partitions happen in real systems, so **P is not optional**. The real choice is **CP** vs **AP**:

- **CP:** Stay consistent; sacrifice availability on the minority side during a partition.
- **AP:** Stay available; accept **eventual consistency** — replicas may temporarily disagree.

**Where systems sit:**
- **PostgreSQL (single node):** Not really a CAP trade-off — one server is CA in theory; partitions are not the main concern.
- **PostgreSQL with replication:** Typically **CP** — primary holds truth; failover promotes a replica.
- **MongoDB (replica set with primary):** Leans **CP** — writes go to the primary; a partitioned minority may be unavailable for writes until the partition heals.
- **AP examples:** Cassandra, DynamoDB (configurable) — favor availability and accept stale reads.

> [!example]
> Two data centers lose network link. **CP:** stop writes on the smaller side so data does not diverge. **AP:** both sides accept writes; merge conflicts later.

> [!success] Pros / Cons
> **CP:** Strong correctness; some clients may get errors during partitions.  
> **AP:** Always responsive; app must handle stale or conflicting data.

> [!tip] Interview tip
> Do not say "pick two of three always." Say: **"During a partition, you choose between consistency and availability."** Single-node Postgres is a different conversation.

> [!info] Further study
> - [MongoDB — consistency and availability](https://www.mongodb.com/docs/manual/core/write-operations-atomicity/)
> - [Brewer's CAP FAQ (Eric Brewer)](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/)

---

### 9. ACID vs BASE

> [!question] Q9
> What is the difference between ACID and BASE?

In plain terms: **ACID** (common in SQL) prioritizes **correctness and strong transactions**. **BASE** (common in large NoSQL systems) prioritizes **availability and scale**, accepting that data may be **temporarily inconsistent** but will **converge** over time (**eventual consistency**).

**BASE** expands to:
- **B**asically **A**vailable — system responds even under failure or load.
- **S**oft state — state may change over time without input (replication lag, background sync).
- **E**ventually consistent — given no new writes, all replicas will match eventually.

> [!example]
> **ACID:** Bank transfer — balance must be exact immediately.  
> **BASE:** Social media like count — may show 99 for a second, then 100 after replicas sync; users tolerate brief lag.

> [!success] Pros / Cons
> **ACID — when to use:** Payments, inventory, bookings. **When not:** Global scale feeds where strict global consistency is too slow.  
> **BASE — when to use:** High-traffic reads, analytics, caches. **When not:** Financial ledger without extra compensating logic.

---

### 10. Isolation levels

> [!question] Q10
> What are database isolation levels (read uncommitted, read committed, repeatable read, serializable) and the anomalies they prevent?

In plain terms: When many transactions run **at the same time**, weird things can happen unless the database **isolates** them. **Isolation levels** control how strict that separation is — stricter = fewer bugs, usually **less concurrency**.

| Level | Dirty read | Non-repeatable read | Phantom read |
|-------|------------|---------------------|--------------|
| Read uncommitted | Allowed | Allowed | Allowed |
| Read committed | Prevented | Allowed | Allowed |
| Repeatable read | Prevented | Prevented | Allowed* |
| Serializable | Prevented | Prevented | Prevented |

**Anomalies explained:**
- **Dirty read:** You read another transaction's **uncommitted** data that may roll back.
- **Non-repeatable read:** You read a row twice; another transaction **changed** it in between.
- **Phantom read:** You run the same query twice; **new rows** appeared (or vanished) because another transaction inserted/deleted.

\* PostgreSQL's repeatable read also prevents phantoms in practice for many cases.

Default in PostgreSQL: **read committed**. MySQL InnoDB default: **repeatable read**.

> [!example]
> Transaction A reads account balance = $100. Transaction B updates to $200 but has not committed. If A reads again at **read uncommitted**, A might see $200 (dirty) — then B rolls back. **Read committed** prevents that.

> [!success] Pros / Cons
> **Weaker levels:** More throughput; risk of anomalies.  
> **Serializable:** Safest; may use locks or retries; slowest under contention.

> [!info] Further study
> - [PostgreSQL — isolation levels](https://www.postgresql.org/docs/current/transaction-iso.html)

---

### 11. Optimistic vs pessimistic locking

> [!question] Q11
> What is optimistic vs pessimistic locking?

In plain terms: Both solve **"two users edit the same row at once."**

**Pessimistic locking:** Lock the row **when you read or update** (`SELECT ... FOR UPDATE`). Others **wait**. Safe under **high contention**; hurts concurrency; **deadlocks** possible.

**Optimistic locking:** **No lock** while reading. Each row has a **version** (number or timestamp). On update: `UPDATE ... WHERE id = 5 AND version = 3`. If zero rows updated, someone else changed it first — **retry or fail**. Good when **conflicts are rare**.

> [!example]
> ```sql
> -- Pessimistic
> BEGIN;
> SELECT * FROM seats WHERE id = 12 FOR UPDATE;
> UPDATE seats SET booked = true WHERE id = 12;
> COMMIT;

> -- Optimistic
> -- Read: { id: 12, version: 7, booked: false }
> UPDATE seats SET booked = true, version = 8
>   WHERE id = 12 AND version = 7;
> ```

> [!success] Pros / Cons
> **Pessimistic:** Strong guarantee under fight for same row; blocking and deadlocks.  
> **Optimistic:** High read concurrency; retries under heavy write conflicts.

> [!tip] Interview tip
> Seat booking / inventory → often **pessimistic**. Profile edit / blog post → often **optimistic**.

---

### 12. N+1 query problem

> [!question] Q12
> What is the N+1 query problem and how do you solve it?

In plain terms: You run **1 query** to load a list of N items, then **N more queries** — one per item — to load related data. Total: **N + 1 queries**. This kills performance under load.

Classic example: Load 100 orders (1 query), then for each order query the user (100 queries) = **101 round trips** to the database.

**Fixes:**
1. **Eager loading / join** — one query with JOIN or ORM `include`.
2. **Batch loading** — collect all IDs, one `WHERE id IN (...)` query.
3. **DataLoader** (GraphQL) — batches and caches per request.
4. In MongoDB: **`$lookup`** or **`populate`** instead of looping queries. See [[16-mongodb]].

> [!example]
> ```javascript
> // BAD — N+1
> const orders = await Order.find();
> for (const o of orders) {
>   o.user = await User.findById(o.userId);  // N queries
> }

> // GOOD — batch
> const orders = await Order.find();
> const userIds = [...new Set(orders.map(o => o.userId))];
> const users = await User.find({ _id: { $in: userIds } });
> ```

> [!success] Pros / Cons
> **Fixing N+1:** Fewer round trips, lower DB load.  
> **Over-eager loading:** Can fetch too much data — balance with projection and pagination.

---

### 13. Database replication

> [!question] Q13
> What is database replication (primary/replica) and what problems does it solve?

In plain terms: **Replication** copies data from one server (**primary** or **leader**) to one or more **replicas** (followers). Writes usually go to the primary; replicas apply changes asynchronously or semi-sync.

**Problems it solves:**
1. **High availability** — if the primary fails, promote a replica (**failover**).
2. **Read scaling** — route read queries to replicas.
3. **Disaster recovery** — copies in another region or data center.

**Trade-off:** Replicas **lag** behind the primary. A read from a replica may be **stale** (**eventual consistency**). Do not read-your-writes from a replica unless you handle that in the app.

> [!example]
> ```
> App writes → Primary (PostgreSQL)
> App reads  → Replica 1, Replica 2 (load balanced)
> Primary dies → Replica 1 promoted to new primary
> ```

> [!success] Pros / Cons
> **Pros:** Uptime, read throughput, geographic copies.  
> **Cons:** Replication lag, failover complexity, split-brain risk during partitions.

> [!info] Further study
> - [PostgreSQL — replication](https://www.postgresql.org/docs/current/high-availability.html)
> - [MongoDB replica sets](https://www.mongodb.com/docs/manual/replication/)

---

### 14. Sharding / horizontal partitioning

> [!question] Q14
> What is sharding / horizontal partitioning and when do you need it?

In plain terms: **Sharding** splits data **horizontally** across many servers (**shards**). Each shard holds a **subset** of rows/documents (e.g. users A–M on shard 1, N–Z on shard 2). Also called **horizontal partitioning**.

**Vertical partitioning** splits by **columns** or fields (profile in one table, logs in another) — different idea.

You need sharding when **one machine** cannot hold the data or handle the **write/read throughput** — disk full, CPU saturated, or connection limits hit.

**Hard parts:**
- **Shard key** — must spread load evenly; bad keys cause **hotspots** (one shard gets all traffic).
- **Cross-shard queries** — expensive or unsupported; design queries to hit one shard when possible.
- **Rebalancing** — moving data when shards grow unevenly.

> [!example]
> Shard key `user_id`: all data for user 42 lives on one shard — great for "my messages" queries.  
> Bad shard key `country` if 90% of users are in one country — one shard overloaded.

> [!success] Pros / Cons
> **When to use:** Outgrown single-node capacity.  
> **When not:** Premature — adds major ops complexity; try replication, caching, and better indexes first.

> [!tip] Interview tip
> Mention **MongoDB sharded clusters** ([[16-mongodb]]) vs **PostgreSQL** (often Citus or app-level sharding).

---

### 15. Connection pooling

> [!question] Q15
> What is connection pooling and why does it matter?

In plain terms: Opening a database **connection** is expensive — TCP handshake, SSL, authentication. **Connection pooling** keeps a **fixed pool** of open connections and **reuses** them for many requests instead of open/close per request.

**Why it matters:**
- **Lower latency** — no repeated handshake per HTTP request.
- **Bounded load** — pool size caps concurrent DB connections; prevents overwhelming the database.
- **Better throughput** — under high concurrency, reuse wins.

Size the pool from DB **max connections**, number of app servers, and expected concurrency. Too small → requests wait; too large → DB thrashing.

> [!example]
> ```javascript
> // Node.js — pg pool (conceptual)
> const pool = new Pool({ max: 20, connectionString: process.env.DATABASE_URL });
> const result = await pool.query('SELECT * FROM users WHERE id = $1', [userId]);
> // Connection returned to pool, not destroyed
> ```

> [!success] Pros / Cons
> **Strengths:** Fast, controlled resource use.  
> **Watch-outs:** Pool per process × many pods can exceed DB limits — use PgBouncer or similar at scale.

---

### 16. Relational vs document DB per feature

> [!question] Q16
> When would you use a relational DB vs a document DB for a specific feature (e.g., orders vs product catalog vs chat messages)?

In plain terms: Match the **database to the access pattern**, not the company logo. This is **polyglot persistence** — multiple database types in one system.

| Feature | Good fit | Why |
|---------|----------|-----|
| **Orders / payments** | Relational ([[15-sql]] / PostgreSQL) | ACID, foreign keys, strict invariants |
| **Product catalog** | Document (MongoDB — [[16-mongodb]]) | Flexible attributes, denormalized product docs, fast reads by ID |
| **Chat messages** | Document or append log + Redis | High write volume, fetch by room/time; Redis for presence/pub-sub |
| **Sessions / cache / rate limits** | Redis (key-value) | TTL, speed, simple key access |

> [!example]
> Checkout service: PostgreSQL transaction debits inventory and creates order row. Catalog service: MongoDB serves `{ sku, title, variants[], images[] }` in one document. Chat: MongoDB stores messages; Redis pub/sub pushes live updates to WebSocket servers.

> [!success] Pros / Cons
> **Polyglot strengths:** Right tool per job.  
> **Watch-outs:** More systems to operate, secure, and keep consistent across services.

> [!tip] Interview tip
> Tie to your CV if relevant: "We used Redis caching for hot catalog paths and PostgreSQL for orders."

---

## Advanced (Caching & Redis)

### 17. Caching strategies

> [!question] Q17
> What is caching and what are the main caching strategies (cache-aside, read-through, write-through, write-behind)? (your CV: Redis caching)

In plain terms: **Caching** stores a **copy** of data in a **fast layer** (memory — often **Redis**) so repeat reads avoid the slower database. A **cache hit** finds data in cache; a **cache miss** must load from the DB.

**Main strategies:**

| Strategy | Read path | Write path | Summary |
|----------|-----------|------------|---------|
| **Cache-aside (lazy)** | App checks cache; on miss, loads DB and fills cache | App writes DB; app deletes/updates cache | Most common; app controls logic |
| **Read-through** | Cache loads from DB on miss (library handles it) | App writes DB; cache invalidated separately | Simpler app read code |
| **Write-through** | Cache may serve reads | Write goes to cache **and** DB synchronously | Strong consistency; slower writes |
| **Write-behind (write-back)** | Reads from cache | Write to cache first; async flush to DB | Fast writes; risk of loss on crash |

> [!example]
> **Cache-aside (typical Redis pattern):**
> ```
> GET user:42 from Redis → miss
> SELECT * FROM users WHERE id = 42
> SET user:42 { ... } EX 3600
> return user
> ```

> [!success] Pros / Cons
> **Cache-aside:** Flexible; app must handle invalidation.  
> **Write-through:** Consistent cache; write latency.  
> **Write-behind:** High write throughput; durability risk until flush.

> [!info] Further study
> - [Redis caching concepts](https://redis.io/docs/latest/develop/use-cases/caching/)
> - [web.dev — HTTP caching](https://web.dev/articles/http-cache)

---

### 18. Cache-aside pattern

> [!question] Q18
> Explain the cache-aside pattern step by step. How do you handle invalidation?

In plain terms: **Cache-aside** means the **application** sits beside the cache and owns the logic — the cache does not automatically talk to the DB.

**Read steps:**
1. App requests data (e.g. `user:42`).
2. Check Redis — **hit?** Return cached value. Done.
3. **Miss?** Query the database.
4. Store result in Redis (usually with a **TTL** — time to live).
5. Return data to the client.

**Invalidation on write/update/delete:**
1. App writes to the **database** first (source of truth).
2. **Delete** the cache key (preferred) or update it.
3. Next read misses cache and reloads fresh data.

**Why delete instead of update?** Avoids **race conditions** where two writes reorder and leave stale values in cache.

> [!example]
> ```javascript
> async function getUser(id) {
>   const cached = await redis.get(`user:${id}`);
>   if (cached) return JSON.parse(cached);
>   const user = await db.users.findById(id);
>   await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 3600);
>   return user;
> }

> async function updateUser(id, data) {
>   await db.users.update(id, data);
>   await redis.del(`user:${id}`);  // invalidate
> }
> ```

> [!success] Pros / Cons
> **Pros:** Simple, works with any DB, very common with Redis.  
> **Cons:** Stale data possible between write and invalidation; cache and DB can temporarily disagree — mitigate with TTL.

> [!tip] Interview tip
> Mention **TTL as a safety net** even when you invalidate on write.

---

### 19. Cache stampede / thundering herd

> [!question] Q19
> What is a cache stampede / thundering herd and how do you prevent it?

In plain terms: A **cache stampede** (also **thundering herd**) happens when a **popular cache key expires** and **many requests at once** all miss, hammering the **database** to rebuild the same value — possibly crashing or slowing everything.

**Prevention techniques:**
1. **Mutex / lock** — only **one** request rebuilds; others wait or get stale data.
2. **Early recompute** — refresh cache **before** expiry (background job at 90% of TTL).
3. **Jittered TTLs** — randomize expiry so keys do not all expire together.
4. **Stale-while-revalidate** — serve old value while one worker refreshes in background.
5. **Never expire hot keys at once** — logical expiry inside value + soft TTL.

> [!example]
> ```
> 10:00:00 — key "homepage:feed" expires
> 10:00:00 — 5,000 requests miss → 5,000 identical DB queries

> With lock:
> Request 1 acquires lock → rebuilds → sets cache
> Requests 2–5000 wait or read stale until key exists
> ```

> [!success] Pros / Cons
> **Locking:** Protects DB; adds latency and lock complexity in Redis.  
> **Stale-while-revalidate:** Smooth UX; brief staleness acceptable for many feeds.

> [!info] Further study
> - [Redis distributed locks (Redlock pattern)](https://redis.io/docs/latest/develop/use-cases/distributed-locks/)

---

### 20. Redis data structures

> [!question] Q20
> What is Redis and what data structures does it provide? Give a use case for each (strings, hashes, lists, sets, sorted sets, streams).

In plain terms: **Redis** is an **in-memory** data store — extremely fast because data lives in RAM. It is often used as a **cache**, session store, message broker, or real-time leaderboard engine. It supports rich **data structures**, not just plain strings.

| Structure | What it is | Use case |
|-----------|------------|----------|
| **Strings** | Key → string/value (JSON, counters) | Cache HTML/JSON; `INCR page:views` |
| **Hashes** | Key → field/value map | User profile `{ name, email, tier }` under `user:42` |
| **Lists** | Ordered sequence | Recent notifications; job queue with `LPUSH`/`BRPOP` |
| **Sets** | Unordered unique members | Tags on a post; "users online today" |
| **Sorted sets (ZSET)** | Members with numeric **score** | Leaderboards; time-window rate limits (score = timestamp) |
| **Streams** | Append-only log with IDs | Event sourcing; consumer groups for workers |

Also: **Bitmaps**, **HyperLogLog** (approximate unique counts), **geospatial** indexes.

> [!example]
> ```redis
> SET product:99 "{\"name\":\"Widget\"}" EX 300
> HSET user:42 name "Alice" email "alice@example.com"
> LPUSH queue:emails "job-1"
> SADD post:7:tags "redis" "database"
> ZADD leaderboard 9850 "player-A"
> XADD orders * item "widget" qty "2"
> ```

> [!success] Pros / Cons
> **Strengths:** Sub-millisecond ops, versatile structures, pub/sub built in.  
> **Watch-outs:** Memory-bound; persistence trade-offs; not a replacement for full SQL analytics.

> [!info] Further study
> - [Redis data types overview](https://redis.io/docs/latest/develop/data-types/)

---

### 21. Redis for rate limiting, sessions, leaderboards, pub/sub

> [!question] Q21
> How do you use Redis for rate limiting, sessions, leaderboards, and pub/sub?

In plain terms: Redis is a common **real-time infrastructure** layer beyond simple caching.

**Rate limiting**
- **Fixed window:** `INCR user:42:requests` + `EXPIRE` per minute.
- **Sliding window:** sorted set with timestamps as scores; remove entries older than window; count members.
- Libraries: token bucket in app logic or Redis Cell module patterns.

**Sessions**
- Key `session:{token}` → hash or JSON blob with `userId`, roles.
- Set **TTL** matching session timeout; extend on activity if needed.
- Fast lookup on every authenticated request.

**Leaderboards**
- **Sorted set:** `ZADD game:scores 1500 "alice"`; `ZREVRANGE` for top 10; `ZRANK` for user rank.

**Pub/Sub**
- Publishers `PUBLISH channel message`; subscribers listen on channels.
- Use case: broadcast chat events to many WebSocket server instances (horizontal scale).
- Note: pub/sub is **fire-and-forget** — not durable; use **Streams** if you need persistence.

> [!example]
> ```redis
> # Rate limit — 100 req/min per IP
> INCR ratelimit:203.0.113.1:202607261200
> EXPIRE ratelimit:203.0.113.1:202607261200 60

> # Session
> SET session:abcxyz "{\"userId\":42}" EX 86400

> # Leaderboard
> ZADD leaderboard 9850 "player-A"
> ZREVRANGE leaderboard 0 9 WITHSCORES

> # Pub/Sub (chat room)
> PUBLISH room:general "{\"user\":\"Alice\",\"text\":\"Hi\"}"
> ```

> [!success] Pros / Cons
> **Pros:** Low latency, simple primitives, scales reads horizontally with app servers.  
> **Cons:** Pub/sub messages lost if no subscriber; session data lost if Redis fails unless persistence/replication configured.

> [!tip] Interview tip
> Connect to system design: "WebSocket servers subscribe to Redis channels so any instance can push to clients in a room."

---

### 22. Redis cache vs primary datastore

> [!question] Q22
> What is the difference between Redis as a cache vs as a primary datastore? How does persistence work (RDB, AOF)?

In plain terms:

**Redis as cache**
- Data is **expendable** — loss means reload from DB.
- Uses **TTLs** and eviction when memory is full.
- Goal: speed, not long-term source of truth.

**Redis as primary datastore**
- Redis **is** the system of record (sessions, real-time counters, job queues).
- Must configure **persistence** and **replication** because RAM alone loses data on restart.

**Persistence options:**

| Mode | How it works | Pros | Cons |
|------|--------------|------|------|
| **RDB** | Point-in-time **snapshots** to disk on schedule | Compact, fast restart | May lose writes since last snapshot |
| **AOF** | Append **log of every write** | More durable; fsync policies | Larger files; slower recovery |
| **Both** | RDB + AOF | Balance speed and durability | More disk and config |

Even as primary store, Redis is **memory-bound** — datasets should fit in RAM (or use Redis Enterprise / tiered options).

> [!example]
> Cache: product page JSON, TTL 5 min — OK if evicted.  
> Primary: session tokens, rate-limit counters — enable AOF `everysec` + replica for HA.

> [!success] Pros / Cons
> **Cache role:** Simple, fast, safe to lose with TTL.  
> **Primary role:** Extremely fast but plan persistence, memory limits, and backup strategy.

> [!info] Further study
> - [Redis persistence](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)

---

### 23. Cache and database consistency

> [!question] Q23
> How do you keep a cache consistent with the database? What are common invalidation pitfalls?

In plain terms: Perfect sync is hard. Goal: **acceptable staleness** — users see fresh enough data without overloading the DB.

**Common pattern (cache-aside):**
1. **Write to database** (source of truth).
2. **Invalidate cache** — delete keys affected.
3. Next read repopulates from DB.

**Pitfalls:**
1. **Updating cache instead of deleting** — concurrent writes can leave wrong final value.
2. **Invalidating before DB commit** — reader reloads old DB data into cache.
3. **Missing related keys** — update user but forget `user:42:posts` list cache.
4. **Read during write** — stale read populates cache after your delete (mitigate: short TTL, version stamps).
5. **Distributed transactions** — cache and DB are not one ACID transaction; design ordering carefully.

**Mitigations:** delete-on-write, **transaction commit then invalidate**, **TTL safety net**, **cache tags** (invalidate all keys tagged `product:99`), **version fields** in cached JSON.

> [!example]
> ```
> WRONG order:
>   DEL cache → COMMIT db (slow) → reader misses, loads OLD row, caches OLD

> BETTER:
>   COMMIT db → DEL cache → next read loads NEW
> ```

> [!success] Pros / Cons
> **Eventual consistency:** Simple and fast at scale.  
> **Strong consistency:** Write-through or skip cache on critical reads — higher latency.

---

### 24. TTLs and eviction policies

> [!question] Q24
> How would you choose TTLs and eviction policies (LRU, LFU) for a cache?

In plain terms:

**TTL (time to live)** — how long a cached entry lives before expiring.

Choose TTL by **staleness tolerance** vs **DB load**:
- **Hot, rarely changing** (country list, static config): long TTL (hours/days) + invalidate on admin change.
- **User-specific, changes often** (profile, cart): shorter TTL (minutes) + invalidate on update.
- **Highly volatile** (stock price): seconds or no cache.

Add **jitter** (random ±10%) so many keys do not expire simultaneously (helps avoid stampede — see Q19).

**Eviction policies** — when Redis **memory is full**, which keys to remove?

| Policy | Behavior | When to use |
|--------|----------|-------------|
| **allkeys-lru** | Evict least **recently used** keys | General-purpose cache (common default) |
| **allkeys-lfu** | Evict least **frequently** used | Some keys always hot; one-hit wonders should leave |
| **volatile-lru / volatile-lfu** | Only evict keys **with TTL set** | Mix of permanent and cache keys |
| **noeviction** | Return errors when full | When every key must stay (dangerous for pure cache) |

**LRU** = not used lately. **LFU** = rarely accessed overall.

> [!example]
> ```redis
> CONFIG SET maxmemory 2gb
> CONFIG SET maxmemory-policy allkeys-lru
> SET session:xyz "..." EX 3600
> SET product:99 "..." EX 300
> ```

> [!success] Pros / Cons
> **Long TTL:** Less DB load; staler data.  
> **LFU vs LRU:** LFU keeps truly popular keys under churn; LRU simpler and often good enough.

> [!info] Further study
> - [Redis key eviction](https://redis.io/docs/latest/operate/oss_and_stack/management/eviction/)

---

### 25. High read vs high write throughput

> [!question] Q25
> How do you design for high read throughput vs high write throughput?

In plain terms: Read-heavy and write-heavy systems need **different shapes**. Often you **separate** them (**CQRS** — Command Query Responsibility Segregation): one path optimized for writes, another for reads.

**High read throughput:**
- **Cache** aggressively (Redis, CDN for static assets).
- **Read replicas** — spread SELECT queries.
- **Denormalize** or **materialized views** — avoid expensive joins at read time.
- **Covering indexes** — index includes all columns the query needs.
- **Pagination** and **projection** — do not fetch entire wide rows.

**High write throughput:**
- **Batch inserts** and **write queues** ( absorb spikes ).
- **Sharding / partitioning** — spread writes across nodes.
- **Fewer indexes** on hot write paths (each index slows INSERT/UPDATE).
- **Append-only logs** (event log, MongoDB oplog patterns — [[16-mongodb]]).
- **Write-behind caching** where durability rules allow.
- **Avoid synchronous cross-service calls** on every write.

> [!example]
> **Social feed read path:** Precomputed fan-out stored in Redis + CDN for media; read replicas for history.  
> **Analytics write path:** Kafka ingest → batch write to column store; separate from OLTP PostgreSQL.

> [!success] Pros / Cons
> **Read optimization:** Fast UX; duplicated data and cache invalidation work.  
> **Write optimization:** Handles scale; eventual consistency and operational complexity.

> [!tip] Interview tip
> Draw two arrows: "writes → queue → DB" and "reads → cache → replica." Mention **CQRS** by name if the role is senior.

> [!info] Further study
> - [web.dev — Performance](https://web.dev/explore/performance)
> - [[06-system-design]] for scaling patterns end-to-end

---

## Related notes

- [[15-sql]] — SQL queries, joins, indexes, and relational modeling in depth
- [[16-mongodb]] — MongoDB schema design, aggregation, sharding, and replica sets
- [[06-system-design]] — Scaling reads/writes, CAP trade-offs, and caching in system design
- [[05-databases]] — Question list (companion to this answer note)

---

## References & Further Study

### Relational databases & SQL
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [PostgreSQL — Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [PostgreSQL — High Availability & Replication](https://www.postgresql.org/docs/current/high-availability.html)

### NoSQL & distributed systems
- [MongoDB Manual](https://www.mongodb.com/docs/manual/)
- [MongoDB — Replication](https://www.mongodb.com/docs/manual/replication/)
- [Eric Brewer — CAP Twelve Years Later](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/)

### Caching & Redis
- [Redis Documentation](https://redis.io/docs/latest/)
- [Redis — Caching use cases](https://redis.io/docs/latest/develop/use-cases/caching/)
- [Redis — Data types](https://redis.io/docs/latest/develop/data-types/)
- [Redis — Persistence (RDB & AOF)](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)
- [Redis — Eviction policies](https://redis.io/docs/latest/operate/oss_and_stack/management/eviction/)
- [Redis — Distributed locks](https://redis.io/docs/latest/develop/use-cases/distributed-locks/)
- [web.dev — HTTP Caching](https://web.dev/articles/http-cache)

### Patterns & architecture
- [Martin Fowler — CQRS](https://martinfowler.com/bliki/CQRS.html)
- [AWS — Caching Best Practices](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Strategies.html)
