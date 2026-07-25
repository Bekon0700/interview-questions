---
title: MongoDB — Answers
topic: mongodb
tags: [interview, fullstack, mongodb, mongoose]
related: ["[[05-databases]]", "[[07-cv-deep-dive]]", "[[04-nestjs]]", "[[15-sql]]"]
---

> [!abstract] Overview
> This note answers all **30 MongoDB interview questions** in plain English, with examples and Pros/Cons for each. It covers the document model, CRUD, indexes (including **ESR**), `explain()`, aggregation, Mongoose, replica sets, sharding, transactions, and schema design patterns. Answers **10, 11, 28, and 29** tie directly to your CV (Affiliate reporting, AdServer, WebChat, **37s → 1s** query). General database concepts live in [[05-databases]]; your spoken STAR stories live in [[07-cv-deep-dive]]; NestJS + Redis + RabbitMQ context lives in [[04-nestjs]].

---

## Beginner

### 1. MongoDB and the document model (BSON)

> [!question] Q1
> What is MongoDB and what is the document model (BSON)? How does it map to relational concepts?

**MongoDB** is a **NoSQL document database**. Instead of rows in tables, you store **documents** — flexible records that look like JSON but use **BSON** (Binary JSON) on disk. BSON adds extra types that JSON does not have: `ObjectId`, `Date`, `Decimal128`, binary data, and more.

Think of it as storing one JSON object per record, and many records grouped in a **collection**. Documents in the same collection can have different fields — there is no fixed table schema enforced by the database.

**Mapping to relational ideas:**

| Relational (SQL) | MongoDB |
|------------------|---------|
| Database | Database |
| Table | Collection |
| Row | Document |
| Column | Field |
| JOIN | Embedded document or array (denormalized), or `$lookup` / reference |

The flexible schema helps when product data changes often (e.g., different optional fields per product). The trade-off is that **your application** must enforce structure and consistency — often with **Mongoose schemas** in a [[04-nestjs]] backend. See [[05-databases#1 SQL vs NoSQL]] for when to pick SQL vs document stores.

> [!example]
> **Relational:** `users` table row `{ id: 1, name: "Alice", email: "a@x.com" }` and `orders` table with `user_id` foreign key.
>
> **MongoDB:** An `orders` document embeds the customer snapshot:
> ```json
> {
>   "_id": ObjectId("..."),
>   "affiliateId": ObjectId("..."),
>   "status": "completed",
>   "total": 49.99,
>   "customer": { "name": "Alice", "email": "a@x.com" },
>   "items": [{ "sku": "BOOK-1", "qty": 2, "price": 24.99 }]
> }
> ```
> One read returns the order and line items — no JOIN needed.

> [!success] Pros / Cons
> **Pros:** Flexible schema; nested data in one document; horizontal scaling via sharding; good fit for read-heavy, document-shaped workloads (catalogs, logs, dashboards).  
> **Cons:** No built-in JOIN like SQL; duplication if you embed; 16 MB document size limit; you must design indexes and schema around **query patterns** (see [[07-cv-deep-dive#9 Database structure and business workflows]]).

> [!tip] Interview tip
> Say "MongoDB is schema-flexible, not schema-less — we enforce shape in the app layer with Mongoose." Mention **polyglot persistence**: orders/payments in PostgreSQL, affiliate reporting in MongoDB — see [[05-databases]].

---

### 2. Collection vs document vs field

> [!question] Q2
> What is a collection vs a document vs a field?

In plain terms:

- A **document** is one record — a single BSON object, like one JSON object `{ key: value, ... }`.
- A **collection** is a group of documents — similar to a table, but **without a fixed schema**. Every document must have an `_id`, but other fields can differ between documents.
- A **field** is one key–value pair inside a document — like one column value, but fields can hold scalars, arrays, or nested objects.

> [!example]
> Collection: `commissions`
>
> Document:
> ```json
> {
>   "_id": ObjectId("665a1b2c3d4e5f6789012345"),
>   "affiliateId": ObjectId("665a0000000000000000001"),
>   "orderId": ObjectId("665a9999999999999999999"),
>   "amount": 12.50,
>   "status": "pending",
>   "createdAt": ISODate("2024-06-15T10:30:00Z")
> }
> ```
> Fields: `_id`, `affiliateId`, `orderId`, `amount`, `status`, `createdAt`.

> [!success] Pros / Cons
> **Pros:** Simple mental model; nested fields avoid extra collections for small related data.  
> **Cons:** Same field name can mean different things in different documents if you are careless; large nested arrays can hit document size limits (see Q27).

---

### 3. `_id` and ObjectId

> [!question] Q3
> What is `_id` and what is an ObjectId? What does it encode?

Every MongoDB document **must** have an `_id` field — it is the **primary key**. If you insert without one, MongoDB creates an **ObjectId** automatically.

An **ObjectId** is a 12-byte value, usually shown as a 24-character hex string. It encodes:

1. **4 bytes** — Unix timestamp (seconds since epoch) → creation time, roughly time-ordered
2. **5 bytes** — random value unique to the machine/process
3. **3 bytes** — incrementing counter

So ObjectIds are **globally unique without a central ID server**, and sorting by `_id` often approximates sorting by creation time. You can extract the timestamp: `ObjectId("665a1b2c3d4e5f6789012345").getTimestamp()`.

> [!example]
> ```javascript
> // Insert without _id — MongoDB generates ObjectId
> db.orders.insertOne({ affiliateId: affiliateId, total: 99.00 });
> // Result _id might be: ObjectId("665a1b2c3d4e5f6789012345")
> ```

> [!success] Pros / Cons
> **Pros:** No coordination needed for unique IDs; time-ordered inserts help range queries on `_id`.  
> **Cons:** Not human-readable; monotonically increasing `_id` as a **shard key** causes write hotspots (see Q22); use business keys (UUID, order number) when you need readable IDs.

> [!tip] Interview tip
> For **keyset pagination** (Q30), use `_id` as a tiebreaker when sorting by a non-unique field like `createdAt`.

---

### 4. Basic CRUD operations

> [!question] Q4
> What are the basic CRUD operations (`insertOne`, `find`, `updateOne`, `deleteOne`)?

**CRUD** = Create, Read, Update, Delete. In MongoDB shell / driver:

| Operation | Methods | What it does |
|-----------|---------|--------------|
| **Create** | `insertOne`, `insertMany` | Add new document(s) to a collection |
| **Read** | `find`, `findOne` | Query with a filter document |
| **Update** | `updateOne`, `updateMany`, `replaceOne` | Modify matching document(s) |
| **Delete** | `deleteOne`, `deleteMany` | Remove matching document(s) |

Queries use a **filter** (which documents) and **options** (projection, sort, limit). Updates use **update operators** like `$set` — you do not replace the whole document unless you use `replaceOne`.

In [[04-nestjs]] with Mongoose: `OrderModel.create()`, `.find()`, `.findOneAndUpdate()`, `.deleteOne()` wrap these with schema validation.

> [!example]
> ```javascript
> // Create
> db.orders.insertOne({ affiliateId: id, status: "pending", total: 50 });

> // Read
> db.orders.find({ affiliateId: id, status: "completed" }).sort({ createdAt: -1 }).limit(20);

> // Update
> db.orders.updateOne(
>   { _id: orderId },
>   { $set: { status: "completed" }, $inc: { viewCount: 1 } }
> );

> // Delete
> db.orders.deleteOne({ _id: orderId });
> ```

> [!success] Pros / Cons
> **Pros:** Simple, JSON-like API; single-document updates are **atomic**.  
> **Cons:** `updateMany` without careful filters can touch too many docs; no `JOIN` in basic find — use aggregation or `$lookup` (Q12, Q14).

---

### 5. Common query operators

> [!question] Q5
> What are common query operators (`$eq`, `$gt`, `$in`, `$and`, `$or`, `$regex`)?

Query operators go **inside the filter document** to express conditions beyond simple equality.

**Comparison:** `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, `$nin`  
**Logical:** `$and`, `$or`, `$not`, `$nor`  
**Element:** `$exists`, `$type`  
**Array:** `$elemMatch`, `$all`, `$size`  
**Evaluation:** `$regex`, `$expr` (compare fields in the same document)

> [!example]
> ```javascript
> // Affiliates with status active or trial, created after a date, email ending in @company.com
> db.affiliates.find({
>   $and: [
>     { status: { $in: ["active", "trial"] } },
>     { createdAt: { $gte: ISODate("2024-01-01") } },
>     { email: { $regex: "@company\\.com$", $options: "i" } }
>   ]
> });
> ```
> **Note:** `$regex` often **cannot use a normal B-tree index** efficiently — prefer exact match or text index for search.

> [!success] Pros / Cons
> **Pros:** Expressive filters in one document; `$in` for multi-value equality works well with indexes.  
> **Cons:** `$or` may run multiple index scans; `$regex` with leading wildcard is slow; complex `$expr` may prevent index use.

> [!tip] Interview tip
> For report date ranges, `$gte` / `$lte` on `createdAt` are **Range** in the **ESR** index rule (Q9) — put them **after** equality and sort fields in compound indexes.

---

### 6. `find()` vs `findOne()` and cursors

> [!question] Q6
> What is the difference between `find()` and `findOne()`? What does a cursor do?

- **`find(filter)`** returns a **cursor** — a pointer that fetches documents in **batches** as you iterate. It does **not** load the entire result set into memory at once.
- **`findOne(filter)`** returns **one document** (or `null`) — the first match according to natural or sort order.

A **cursor** lets you chain `.sort()`, `.limit()`, `.skip()`, `.project()` (or pass projection in options). Drivers fetch documents lazily (default batch size ~101), which matters for large result sets.

> [!example]
> ```javascript
> // Cursor — good for many rows
> const cursor = db.orders.find({ affiliateId: id }).sort({ createdAt: -1 }).limit(100);
> for await (const doc of cursor) { /* process */ }

> // Single doc — good for "get by id"
> const order = db.orders.findOne({ _id: orderId });
> ```

In Mongoose, `Model.find()` returns a Query (thenable cursor-like); `.lean()` (Q18) makes reads faster for API responses.

> [!success] Pros / Cons
> **Pros:** Cursors scale to large reads; batching reduces memory.  
> **Cons:** Cursors time out if idle too long (default 10 min); deep `skip()` on cursors is slow at scale (Q30).

---

### 7. Projection

> [!question] Q7
> What is projection and why use it?

**Projection** chooses **which fields** MongoDB returns — like `SELECT col1, col2` in SQL instead of `SELECT *`.

Include fields with `1`, exclude with `0` (cannot mix except `_id`):

```javascript
db.orders.find(
  { affiliateId: id },
  { orderId: 1, amount: 1, createdAt: 1, status: 1, _id: 0 }
);
```

**Why use it:**
- **Less network traffic** — especially important for wide documents
- **Less memory** in the app and driver
- **Covered queries** — if **all** requested fields exist in an index, MongoDB can answer from the index alone without fetching the full document (see Q10)

> [!example]
> Affiliate dashboard list needs only summary fields, not full line items:
> ```javascript
> db.orders.find(
>   { affiliateId: id, status: "completed" },
>   { total: 1, commission: 1, createdAt: 1 }
> ).sort({ createdAt: -1 }).limit(50);
> ```

> [!success] Pros / Cons
> **Pros:** Faster responses; enables covered queries; clearer API contracts.  
> **Cons:** Easy to forget a field the UI needs; covered queries require careful index + projection design.

> [!tip] Interview tip
> Mention projection when telling the **37s → 1s** story — it was one lever after fixing the index ([[07-cv-deep-dive#10 Query optimization 37s → 1s]]).

---

## Intermediate

### 8. Index types

> [!question] Q8
> What types of indexes does MongoDB support (single, compound, multikey, text, geospatial, hashed, TTL, partial, unique)? (commonly asked)

An **index** is a separate data structure (usually **B-tree**) that maps field values to document locations — same idea as [[05-databases#3 Database indexes]], but on document fields.

| Type | Purpose |
|------|---------|
| **Single-field** | One field — basic equality/range/sort |
| **Compound** | Multiple fields in order — critical for real queries (Q9) |
| **Multikey** | Automatic when indexing an **array** field — one index entry per array element |
| **Text** | Full-text search on string fields |
| **Geospatial** | `2dsphere` for location queries |
| **Hashed** | Hash of field value — even distribution for **hashed sharding** |
| **TTL** | Auto-delete documents after a time — sessions, logs, temp tokens |
| **Partial** | Index only documents matching a filter — smaller, cheaper |
| **Unique** | Enforce uniqueness — e.g., affiliate referral code |

> [!example]
> ```javascript
> // Unique referral code
> db.referralLinks.createIndex({ code: 1 }, { unique: true });

> // TTL — delete sessions after 24 hours
> db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 86400 });

> // Partial — index only active affiliates (smaller index)
> db.affiliates.createIndex(
>   { email: 1 },
>   { partialFilterExpression: { status: "active" } }
> );
> ```

> [!success] Pros / Cons
> **Pros:** Right indexes turn `COLLSCAN` into `IXSCAN` (Q10); TTL and partial indexes reduce ops cost.  
> **Cons:** Every index slows writes and uses RAM/disk; wrong indexes waste space and mislead the planner.

> [!tip] Interview tip
> Name **compound + ESR** as your go-to for reporting queries — not "index every field." See [[05-databases#3 Database indexes]] for general index trade-offs.

---

### 9. Compound index and the ESR rule

> [!question] Q9
> What is a compound index and the ESR (Equality, Sort, Range) rule for ordering its fields?

A **compound index** includes **multiple fields in a fixed order**, e.g. `{ affiliateId: 1, status: 1, createdAt: -1 }`. MongoDB can use it efficiently when your query uses a **prefix** of those fields — left to right.

The **ESR rule** tells you how to **order** fields in the index:

1. **E — Equality** first: fields matched with exact values (`status: "completed"`, `affiliateId: ObjectId(...)`)
2. **S — Sort** next: fields in your `.sort()` — so MongoDB returns rows **already ordered** from the index (no in-memory SORT)
3. **R — Range** last: fields with `$gt`, `$gte`, `$lt`, `$lte`, `$in` on a non-equality range

**Wrong order** → collection scan or expensive in-memory sort even if "an index exists."

> [!example]
> Query:
> ```javascript
> db.orders.find({
>   affiliateId: affiliateId,
>   status: "completed",
>   createdAt: { $gte: startDate, $lte: endDate }
> }).sort({ createdAt: -1 });
> ```
>
> **Good index (ESR):** `{ affiliateId: 1, status: 1, createdAt: -1 }`  
> - Equality: `affiliateId`, `status`  
> - Sort + range: same field `createdAt` — index direction `-1` matches sort  
>
> **Bad index:** `{ createdAt: -1, affiliateId: 1 }` — range/sort on `createdAt` before equality on `affiliateId` may not narrow the scan for one affiliate.

> [!success] Pros / Cons
> **Pros:** One well-designed compound index serves the hottest dashboard query; sort from index avoids RAM sort.  
> **Cons:** One index per query pattern — you cannot one-size-fit-all; wrong field order is a silent performance killer.

> [!tip] Interview tip — ESR
> Whiteboard flow: (1) list equality filters, (2) list sort fields, (3) list range filters, (4) build index E → S → R. Say: "On the affiliate platform, `{ affiliateId: 1, status: 1, createdAt: -1 }` matched how affiliates filter **their** orders." Cross-ref [[07-cv-deep-dive#9 Database structure and business workflows]] and Q29.

---

### 10. `explain()` — COLLSCAN vs IXSCAN

> [!question] Q10
> How does `explain()` work? What do `COLLSCAN` vs `IXSCAN` mean, and what fields do you look at? (your CV: 37s → 1s optimization)

**`explain()`** asks MongoDB to show **how** it will run (or ran) a query — the **query plan** and **execution statistics**. Always use **`explain("executionStats")`** for real numbers, not just the plan.

```javascript
db.orders.explain("executionStats").find({
  affiliateId: id,
  status: "completed",
  createdAt: { $gte: start, $lte: end }
}).sort({ createdAt: -1 });
```

**Key plan stages:**

| Stage | Meaning |
|-------|---------|
| **`COLLSCAN`** | **Collection scan** — reads every document. Bad on large collections. |
| **`IXSCAN`** | **Index scan** — reads index entries, then fetches docs (unless covered). Good. |
| **`SORT`** | In-memory sort — sort **not** satisfied by index. Red flag on big sets. |
| **`FETCH`** | Load full document after index lookup |

**Fields to inspect in `executionStats`:**

- **`nReturned`** — documents returned to the client
- **`totalDocsExamined`** — documents scanned (want this **close to** `nReturned`)
- **`totalKeysExamined`** — index keys scanned
- **`executionTimeMillis`** — wall time (compare before/after index)
- **`winningPlan`** — the chosen plan (look for COLLSCAN vs IXSCAN)

**Healthy query:** `IXSCAN`, no `SORT` stage (or sort from index), `totalDocsExamined ≈ nReturned`.  
**Sick query:** `COLLSCAN`, `totalDocsExamined` in millions, `nReturned` in hundreds, plus `SORT` — exactly what caused **~37s** on the affiliate report before indexing.

> [!example]
> **Before index (bad):**
> - `stage: "COLLSCAN"`
> - `totalDocsExamined: 2,000,000`, `nReturned: 487`
> - `SORT` stage — 500 MB sort in memory
> - `executionTimeMillis: 37200`
>
> **After compound index `{ affiliateId: 1, status: 1, createdAt: -1 }` (good):**
> - `stage: "IXSCAN"`
> - `totalDocsExamined: 490`, `nReturned: 487`
> - No separate `SORT` stage
> - `executionTimeMillis: 980`

> [!success] Pros / Cons
> **Pros:** Evidence-based optimization — you fix the plan, not guess; interviewers love a measured story.  
> **Cons:** Stats on empty dev DB lie — reproduce with production-like volume; `explain()` on aggregations needs `aggregate(..., { explain: true })`.

> [!tip] Interview tip — explain() & 37s → 1s
> Structure your answer: **Symptom** (37s timeout) → **Tool** (`explain('executionStats')`) → **Diagnosis** (COLLSCAN + SORT, examined >> returned) → **Fix** (ESR compound index + projection + `$match` early in pipeline) → **Proof** (IXSCAN, examined ≈ returned, ~1s). Full walkthrough in Q11 and [[07-cv-deep-dive#10 Query optimization 37s → 1s]]. Contrast with SQL `EXPLAIN` in [[15-sql]].

---

### 11. Walkthrough: 37 seconds → 1 second (CV deep dive)

> [!question] Q11
> Walk through how you reduced a query from 37 seconds to 1 second. (your CV)

> [!warning] Adapt to your real fields
> The collection names, field names, and numbers below are **templates** based on your CV (Rokomari Affiliate / NestJS). **Replace them with what you actually built** before the interview. Interviewers will ask: "What was the exact index? Show me the query." Only claim steps you performed.

**Situation:** On the affiliate platform ([[04-nestjs]]), dashboard **report endpoints** loaded an affiliate's completed orders for a date range, sorted newest first. At production data volume, the MongoDB query took **~37 seconds**. DB CPU spiked during business hours; affiliates saw timeouts. This was unacceptable for a product selling trust in earnings data.

**Task:** Find the root cause with evidence, fix it without breaking correctness, and verify under realistic load — same discipline as general DB tuning in [[05-databases]], but MongoDB-specific via `explain()` and compound indexes.

---

#### Step 1 — Reproduce with realistic volume

Staging with 50 documents hides the problem. I copied **production-like row counts** (or ran against a sanitized prod snapshot) so `explain()` numbers meant something. The slow path was a **Mongoose** `find()` plus `.sort()` — and in some cases an **aggregation** that had `$match` **after** `$lookup` (fixing stage order came later).

---

#### Step 2 — Run `explain("executionStats")`

```javascript
db.orders.explain("executionStats").find({
  affiliateId: ObjectId("665a0000000000000000001"),
  status: "completed",
  createdAt: { $gte: ISODate("2024-06-01"), $lte: ISODate("2024-06-30") }
}).sort({ createdAt: -1 }).limit(500);
```

**What I saw (template numbers — use yours):**

| Metric | Value | Meaning |
|--------|-------|---------|
| Winning stage | `COLLSCAN` | Full collection scan — no useful index |
| `totalDocsExamined` | ~2,000,000 | Every order in the system scanned |
| `nReturned` | ~487 | Only this affiliate's June orders |
| Extra stage | `SORT` | Sort not from index — large in-memory sort |
| `executionTimeMillis` | ~37,000 | ~37 seconds |

**Diagnosis in one sentence:** The database read **millions** of documents to return **hundreds**, because filters and sort did not match any compound index.

---

#### Step 3 — Map the query to ESR

Query pattern (affiliate dashboard — see [[07-cv-deep-dive#9 Database structure and business workflows]]):

- **Equality:** `affiliateId`, `status`
- **Sort:** `createdAt` descending
- **Range:** `createdAt` between `$gte` and `$lte`

**ESR index:**

```javascript
db.orders.createIndex({ affiliateId: 1, status: 1, createdAt: -1 });
```

Why this order:
- `affiliateId` + `status` narrow to one affiliate's completed orders first
- `createdAt: -1` supports **both** the range filter and `.sort({ createdAt: -1 })` without a separate SORT stage

> [!warning] Adapt to your real fields
> If your query filters on `paymentStatus` not `status`, or sorts by `orderDate` not `createdAt`, the index must use **your** field names. Wrong names on a whiteboard fail the interview.

---

#### Step 4 — Add projection (and `.lean()` in Mongoose)

The API did not need full line items for the list view. I added projection:

```javascript
OrderModel.find(filter, { total: 1, commission: 1, createdAt: 1, orderNumber: 1 })
  .sort({ createdAt: -1 })
  .limit(500)
  .lean();
```

`.lean()` skips Mongoose document hydration — meaningful on read-heavy report paths (Q18). Where the projection matched index keys, the plan moved toward a **covered query** (index only, no FETCH).

---

#### Step 5 — Fix aggregation pipelines (if applicable)

For summary reports using aggregation, I moved **`$match` on `affiliateId`, `status`, and date range to the first stage** — before `$lookup`, `$unwind`, or `$group`. Early `$match` lets MongoDB use the same compound index and shrinks every later stage.

```javascript
[
  { $match: { affiliateId: id, status: "completed", createdAt: { $gte: start, $lte: end } } },
  { $sort: { createdAt: -1 } },
  { $limit: 500 },
  { $project: { total: 1, commission: 1, createdAt: 1 } }
]
```

---

#### Step 6 — Verify with `explain()` again

| Metric | Before | After |
|--------|--------|-------|
| Plan | `COLLSCAN` + `SORT` | `IXSCAN` |
| `totalDocsExamined` | ~2,000,000 | ~490 |
| `nReturned` | ~487 | ~487 |
| `executionTimeMillis` | ~37,000 | **~980 (~1s)** |

Also checked: p95 API latency on the report endpoint, MongoDB CPU during peak — both dropped. Added this index pattern to our **performance checklist** for new report queries.

---

#### Spoken answer (60–90 seconds)

> "Affiliate reports were timing out at ~37 seconds. I ran `explain('executionStats')` and saw a collection scan over two million orders plus an in-memory sort, while we only returned a few hundred rows for one affiliate. The filter and sort fields weren't covered by a compound index. I added `{ affiliateId: 1, status: 1, createdAt: -1 }` following ESR — equality fields first, then sort/range on `createdAt`. I trimmed the payload with projection and `.lean()`, and moved `$match` to the start of any aggregation. Re-running explain showed index scan, documents examined close to returned, and execution time dropped to about one second. I validated on production-like data, not an empty staging DB."

> [!example]
> See also the STAR bullets in [[07-cv-deep-dive#10 Query optimization 37s → 1s]] and the NestJS report module in [[04-nestjs]] (caching with Redis came **after** fixing the underlying query — cache hides bad plans temporarily).

> [!success] Pros / Cons
> **Strengths of this story:** Shows **method** (measure → diagnose → index design → verify), ties to **business pain** (affiliate trust, DB CPU), demonstrates **ESR** and **explain()** fluency.  
> **Watch-outs:** Do not memorize fake field names; know **write cost** of the new index (slightly slower inserts on `orders` — acceptable for read-heavy reports); if asked "why not cache only?" say cache helps **after** the query is sane — a 37s cold query still kills you on cache miss.

> [!tip] Interview tip — 37s → 1s
> Bring a **before/after** pair: COLLSCAN vs IXSCAN, examined vs returned counts, millis. Offer: "Happy to draw the index on a whiteboard." Link to [[05-databases#3 Database indexes]] for "indexes speed reads, cost writes."

---

### 12. Aggregation pipeline

> [!question] Q12
> What is the aggregation pipeline? Explain common stages (`$match`, `$group`, `$project`, `$sort`, `$lookup`, `$unwind`, `$facet`).

The **aggregation pipeline** runs documents through a **sequence of stages** — each stage transforms the stream and passes output to the next. Think of it as a conveyor belt of operations, like Unix pipes.

| Stage | Purpose |
|-------|---------|
| **`$match`** | Filter documents — like `find()`. **Put early** to use indexes. |
| **`$project`** | Reshape fields — include, exclude, compute |
| **`$group`** | Aggregate by `_id` key — `$sum`, `$avg`, `$count`, etc. |
| **`$sort`** | Order documents |
| **`$limit` / `$skip`** | Pagination (prefer keyset over deep skip — Q30) |
| **`$lookup`** | Join another collection (Q14) |
| **`$unwind`** | One doc per array element |
| **`$facet`** | Multiple sub-pipelines in parallel (e.g., page of results + total count) |
| **`$addFields`** | Add computed fields without full project |

**Order matters** for correctness and performance — always `$match` and index-backed `$sort` as early as possible (Q26).

> [!example]
> Daily commission totals per affiliate for June:
> ```javascript
> db.orders.aggregate([
>   { $match: { status: "completed", createdAt: { $gte: start, $lt: end } } },
>   { $group: {
>       _id: "$affiliateId",
>       orderCount: { $sum: 1 },
>       totalCommission: { $sum: "$commission" }
>   }},
>   { $sort: { totalCommission: -1 } },
>   { $limit: 20 }
> ]);
> ```

> [!success] Pros / Cons
> **Pros:** Powerful server-side analytics; `$facet` avoids double round-trips for count + page.  
> **Cons:** Easy to build slow pipelines (`$lookup` on huge sets, `$unwind` on big arrays); harder to debug than simple `find()` — use `explain` on aggregations.

> [!tip] Interview tip
> Say: "I treat `$match` as the first line of defense — same index discipline as find()." Affiliate heavy rollups may run in [[04-nestjs]] **workers** off the request path ([[07-cv-deep-dive#12 RabbitMQ in the affiliate system]]).

---

### 13. Embed vs reference

> [!question] Q13
> When should you embed documents vs reference them? What are the trade-offs?

**Embed** — store related data **inside** the parent document (subdocument or array).  
**Reference** — store an **`ObjectId`** (or key) pointing to another document.

| Embed when | Reference when |
|------------|----------------|
| Data is always read **together** | Data is **large** or shared by many parents |
| **One-to-few** relationship | **One-to-many-unbounded** (millions of events) |
| Updates are **local** to the parent | Entity is updated **independently** often |
| You want **one read, no join** | Avoid duplication and 16 MB doc limit |

**Rule of thumb:** *Data that is queried together should be stored together* — but respect the **16 MB document limit** and unbounded array growth (Q27).

> [!example]
> **Embed:** Order line items inside `orders` — always shown with the order, bounded count.  
> **Reference:** `affiliateId` on `orders` pointing to `affiliates` — affiliate profile shared across thousands of orders; update affiliate name in one place.

> [!success] Pros / Cons
> **Embed pros:** Single read; atomic single-document update. **Embed cons:** Duplication; document growth.  
> **Reference pros:** Normalized; smaller documents. **Reference cons:** Extra queries, `$lookup`, or `populate()` (Q28).

> [!tip] Interview tip
> Affiliate platform: embed **order snapshot** fields needed for reports; reference **affiliate** master data — see Q29 and [[06-system-design]] affiliate section.

---

### 14. `$lookup` vs SQL JOIN

> [!question] Q14
> What is `$lookup` and how does it compare to a SQL join? What are its performance implications?

**`$lookup`** performs a **left outer join** in an aggregation pipeline: for each input document, it matches documents in another collection on a local/foreign field pair.

```javascript
{
  $lookup: {
    from: "affiliates",
    localField: "affiliateId",
    foreignField: "_id",
    as: "affiliate"
  }
}
```

**Compared to SQL JOIN:**
- SQL optimizers have decades of join algorithms (hash join, merge join)
- MongoDB `$lookup` is **less sophisticated** — can be slow on large collections
- No automatic "join order" optimization like mature RDBMS
- **`foreignField` should be indexed** — critical

**Performance implications:**
- `$lookup` **after** a huge `$match` on the wrong collection = disaster
- Always **`$match` first** to shrink input; index the join keys
- Prefer **schema design** (embedding, extended reference pattern) over frequent large joins

> [!example]
> **Slow:** `$lookup` all orders → then `$match` one affiliate.  
> **Fast:** `$match` `{ affiliateId: id }` first (uses compound index) → small `$lookup` to enrich with affiliate name if not denormalized.

> [!success] Pros / Cons
> **Pros:** Join without a SQL database; flexible pipelines.  
> **Cons:** Expensive at scale; easy to misuse; often a sign schema could denormalize hot fields.

> [!tip] Interview tip
> Contrast with [[15-sql]] joins for hybrid architectures; on affiliate reports you preferred indexed finds + denormalized fields over heavy `$lookup` ([[07-cv-deep-dive#9 Database structure and business workflows]]).

---

### 15. `updateOne`, `updateMany`, `replaceOne`, upserts

> [!question] Q15
> What is the difference between `updateOne`, `updateMany`, `replaceOne`, and upserts?

| Method | Behavior |
|--------|----------|
| **`updateOne`** | Update **first** matching document using operators (`$set`, `$inc`, …) |
| **`updateMany`** | Update **all** matching documents |
| **`replaceOne`** | Replace **entire document** (except `_id`) — omit fields = gone |
| **Upsert** | Option `{ upsert: true }` on update — **insert** if no match |

Upserts avoid race-prone "check exists then insert" patterns — one atomic operation.

> [!example]
> ```javascript
> // Idempotent commission status — create or update
> db.commissions.updateOne(
>   { orderId: orderId, affiliateId: affiliateId },
>   { $set: { amount: 12.50, status: "pending", updatedAt: new Date() } },
>   { upsert: true }
> );
> ```

> [!success] Pros / Cons
> **Pros:** Upserts simplify idempotent workers ([[04-nestjs]] RabbitMQ consumers); operators are atomic per document.  
> **Cons:** `updateMany` without strict filter can touch wrong docs; `replaceOne` drops fields if you pass a full new object carelessly.

---

### 16. Atomic update operators

> [!question] Q16
> What are atomic operators like `$set`, `$inc`, `$push`, `$pull`, `$addToSet`?

Update operators modify documents **atomically at the document level** — concurrent updates to the **same document** are safe without application locks.

| Operator | Action |
|----------|--------|
| **`$set`** | Set field value |
| **`$unset`** | Remove field |
| **`$inc` / `$mul`** | Increment / multiply numbers |
| **`$push`** | Append to array |
| **`$pull`** | Remove array elements matching condition |
| **`$addToSet`** | Append only if not already present |
| **`$pop`** | Remove first/last array element |

> [!example]
> AdServer / click tracking (CV): concurrent `$inc` on impression counters:
> ```javascript
> db.adStats.updateOne(
>   { adId: id, date: "2024-06-15" },
>   { $inc: { impressions: 1, clicks: 1 } },
>   { upsert: true }
> );
> ```

> [!success] Pros / Cons
> **Pros:** Race-safe counters and tallies; no read-modify-write in app code.  
> **Cons:** Atomicity is **per document** — multi-document consistency needs transactions (Q23).

---

### 17. Mongoose schemas, models, middleware, virtuals

> [!question] Q17
> What are Mongoose schemas, models, middleware (hooks), and virtuals?

**Mongoose** adds an **application-level schema** on top of schema-flexible MongoDB — standard in [[04-nestjs]] services.

- **Schema** — defines field types, defaults, validation, indexes
- **Model** — compiled schema constructor — `OrderModel.find()`, etc.
- **Middleware (hooks)** — `pre` / `post` on `save`, `find`, `updateOne`, … — hashing passwords, timestamps, logging
- **Virtuals** — computed properties **not stored** in MongoDB (e.g., `fullName` from `firstName` + `lastName`)

> [!example]
> ```typescript
> @Schema({ timestamps: true })
> export class Order {
>   @Prop({ type: Types.ObjectId, ref: 'Affiliate', index: true })
>   affiliateId: Types.ObjectId;

>   @Prop({ enum: ['pending', 'completed', 'cancelled'] })
>   status: string;
> }
> // pre('save') hook to compute commission before write
> ```

> [!success] Pros / Cons
> **Pros:** Validation and structure in code; hooks centralize cross-cutting logic.  
> **Cons:** Extra overhead vs raw driver; schema drift if out of sync with actual documents.

---

### 18. Mongoose `.lean()`

> [!question] Q18
> What does `.lean()` do in Mongoose and when should you use it?

By default, Mongoose returns **full Mongoose documents** — with change tracking, getters, setters, and methods. **`.lean()`** returns **plain JavaScript objects** — faster and lighter.

**Use `.lean()` when:**
- Read-only API responses (lists, reports, dashboards)
- High-throughput reads where you do not call `.save()`
- Affiliate report endpoints after index fix ([[07-cv-deep-dive#11 Achieving low server load]])

**Do not use when:**
- You need document methods, virtuals, or middleware on save
- You will mutate and `.save()` the result

> [!example]
> ```typescript
> return this.orderModel
>   .find({ affiliateId, status: 'completed' })
>   .select('total commission createdAt')
>   .sort({ createdAt: -1 })
>   .limit(50)
>   .lean()
>   .exec();
> ```

> [!success] Pros / Cons
> **Pros:** Lower CPU and memory on hot read paths; pairs well with projection.  
> **Cons:** No virtuals (unless `virtuals: true` lean option); plain objects bypass some Mongoose safety rails.

---

## Advanced

### 19. Replica set architecture

> [!question] Q19
> What is the MongoDB replica set architecture? Explain primary, secondaries, elections, and oplog.

A **replica set** is a group of **`mongod`** nodes with the **same data** for **high availability**:

- **Primary** — receives all **writes**; applies them to data and **oplog**
- **Secondaries** — replicate the **oplog** (operations log) and apply operations to stay in sync
- **Election** — if primary dies, secondaries **vote** (Raft-like) to promote a new primary — automatic failover
- **Oplog** — capped collection of idempotent operations; replication tail + basis for **change streams** (Q24)

Minimum production setup: **3 nodes** (or 2 nodes + arbiter — arbiter does not hold data).

> [!example]
> ```
> App → Primary (writes)
>         │ oplog
>         ├── Secondary 1 (replicate, optional reads)
>         └── Secondary 2 (replicate, optional reads)
> Primary fails → election → Secondary 1 becomes primary
> ```

> [!success] Pros / Cons
> **Pros:** Automatic failover; read scaling via secondaries (with caveats — Q21); durability with write concern `majority`.  
> **Cons:** Replication **lag** on secondaries; not a backup substitute — you still need backups; split-brain rare but ops must monitor.

> [!info] Further study
> [MongoDB — Replication](https://www.mongodb.com/docs/manual/replication/)

---

### 20. Read concerns and write concerns

> [!question] Q20
> What are read concerns and write concerns? How do they affect consistency and durability?

These knobs control **how safe** reads and writes are in a replica set — related to CAP trade-offs in [[05-databases]].

**Write concern** — when is a write "successful"?
- `w: 1` — primary acknowledged only (fast, less durable if primary dies before replicate)
- `w: "majority"` — majority of nodes acknowledged (durable, survives failover)
- `j: true` — wait for journal flush to disk

**Read concern** — what data can a read see?
- `local` — latest on that node (may be rolled back on primary failure)
- `majority` — only data committed by majority (no rolled-back reads)
- `linearizable` — strongest (with performance cost)

> [!example]
> Financial commission **payout record** write: `{ w: "majority", j: true }`  
> Affiliate **dashboard list** (can tolerate slight lag): `readConcern: "local"` on primary

> [!success] Pros / Cons
> **Higher concern pros:** Stronger durability/consistency. **Higher concern cons:** More latency; may wait for replication.

---

### 21. Read preference

> [!question] Q21
> What is read preference (primary, secondary, nearest) and when would you read from secondaries?

**Read preference** tells the driver **which replica set members** can serve reads:

| Mode | Behavior |
|------|----------|
| **`primary`** | Default — reads from primary (strongest consistency) |
| **`primaryPreferred`** | Primary if up, else secondary |
| **`secondary`** | Only secondaries |
| **`secondaryPreferred`** | Secondary if available, else primary |
| **`nearest`** | Lowest network latency member |

**Read from secondaries when:**
- Scaling **read throughput** (reports, analytics)
- Geo-routing users to a nearby replica

**Risks:** **Replication lag** — stale reads. Never use secondaries for "did payment succeed?" immediately after write unless you understand lag.

> [!example]
> Affiliate **historical** report (yesterday's data): `secondaryPreferred` OK.  
> **Just placed order** visibility on dashboard: read from **primary**.

> [!success] Pros / Cons
> **Pros:** Offloads read traffic from primary. **Cons:** Stale data; lag spikes under load.

---

### 22. Sharding

> [!question] Q22
> How does sharding work in MongoDB? How do you choose a shard key, and what makes a bad one?

**Sharding** splits one collection across **multiple shards** (each usually a replica set) for horizontal scale.

- **`mongos`** — query router, directs ops to right shard(s)
- **Config servers** — store cluster metadata (chunk ranges → shards)
- **Shard key** — indexed field(s) determining which shard owns each document

**Good shard key:**
- **High cardinality** — many distinct values
- **Even distribution** — avoids one hot shard
- **Query isolation** — common queries include shard key so one shard answers (not scatter-gather)

**Bad shard key:**
- **Monotonic increasing** only (`_id`, timestamp alone) — all writes hit **one shard** (hotspot)
- **Low cardinality** (e.g., `country: "BD"`) — jumbo chunks, imbalance
- **Hashed** key — spreads writes but **hurts range queries** on that field

> [!example]
> Affiliate platform at huge scale: shard `orders` by `{ affiliateId: 1, createdAt: 1 }` — affiliate dashboard queries hit **one shard** when filtered by `affiliateId`. See [[06-system-design]].

> [!success] Pros / Cons
> **Pros:** Horizontal scale beyond one machine. **Cons:** Shard key is **almost immutable** choice; scatter-gather queries are slow; ops complexity.

> [!info] Further study
> [MongoDB — Sharding](https://www.mongodb.com/docs/manual/sharding/)

---

### 23. Transactions

> [!question] Q23
> How do transactions work in MongoDB (multi-document, sessions)? What are the limitations?

MongoDB **multi-document ACID transactions** use **sessions** (`startSession()`, `withTransaction()`). Supported on **replica sets** and **sharded clusters** (MongoDB 4.0+ / 4.2+).

```javascript
const session = client.startSession();
await session.withTransaction(async () => {
  await commissions.updateOne({ ... }, { $set: { status: 'paid' } }, { session });
  await payouts.insertOne({ ... }, { session });
});
```

**Limitations:**
- **Performance overhead** — do not wrap every request
- **60 second default** time limit (configurable with care)
- Lock contention under high concurrency
- Sharded transactions have extra constraints

**Best practice:** Design so **single-document updates** suffice (embed related fields). Use transactions for rare cross-collection invariants — compare [[05-databases#5 ACID transactions]].

> [!example]
> Payout batch: mark commission **paid** and insert payout record atomically — transaction justified.  
> Daily counter increment — single-doc `$inc`, no transaction needed.

> [!success] Pros / Cons
> **Pros:** ACID across documents when truly needed. **Cons:** Slower; should be short and rare.

---

### 24. Change streams

> [!question] Q24
> What are change streams and what problems do they solve? (real-time features)

**Change streams** let applications **subscribe to real-time data changes** (insert, update, delete, replace) on a collection, database, or deployment. Built on the **replica set oplog**.

```javascript
const stream = db.orders.watch([{ $match: { 'fullDocument.status': 'completed' } }]);
stream.on('change', (change) => { /* notify, invalidate cache, sync search index */ });
```

**Problems they solve:**
- **No polling** — efficient event-driven reactions
- Real-time UI updates (WebChat — CV)
- Cache invalidation when underlying data changes
- Sync to Elasticsearch / analytics
- **Resume tokens** — continue after disconnect

> [!example]
> WebChat ([[07-cv-deep-dive]]): watch `messages` collection → push new messages to connected WebSocket clients via [[04-nestjs]] gateway.

> [!success] Pros / Cons
> **Pros:** Real-time, resumable, MongoDB-native. **Cons:** Requires replica set; ordering guarantees within one shard; not a full message queue replacement.

> [!info] Further study
> [MongoDB — Change Streams](https://www.mongodb.com/docs/manual/changeStreams/)

---

### 25. Schema design patterns

> [!question] Q25
> What schema design patterns do you know (bucket, outlier, computed, subset, extended reference)?

| Pattern | Idea | Use case |
|---------|------|----------|
| **Bucket** | Group many small events into one doc per time window | IoT, logs, metrics |
| **Outlier** | Main doc + overflow collection for rare huge cases | Celebrity with millions of followers |
| **Computed** | Store pre-aggregated totals | Dashboard KPIs without full scan |
| **Subset** | Keep last N items embedded, rest elsewhere | Last 10 reviews in product doc |
| **Extended reference** | Duplicate few hot fields from referenced doc | Avoid `$lookup` for name/status |

> [!example]
> **Computed:** `affiliateDailyStats` collection updated by worker — `{ affiliateId, date, orderCount, commissionSum }` — powers fast month view (Q29).  
> **Extended reference:** Store `affiliateName` on `orders` at write time for report list without join.

> [!success] Pros / Cons
> **Pros:** Each pattern fixes a concrete access-pattern problem. **Cons:** Duplication and eventual consistency on computed fields — workers must be idempotent.

---

### 26. Optimizing aggregation pipelines

> [!question] Q26
> How do you optimize aggregation pipelines (index usage, `$match` early, `allowDiskUse`, `$project` to reduce docs)?

1. **`$match` early** — same indexes as `find()`; reduces all downstream work
2. **Index-backed `$sort`** after selective `$match`
3. **`$project` / `$unset` early** — shrink documents flowing through pipeline
4. **Index `$lookup` foreign fields** — `foreignField: "_id"` or indexed key
5. **Avoid `$unwind`** on huge arrays unless necessary
6. **`allowDiskUse: true`** — large `$sort` / `$group` spilling to disk instead of 100 MB memory limit
7. **`explain`** on aggregation — confirm `IXSCAN` in early stages
8. **Precompute** hot rollups (computed pattern) — affiliate monthly stats off the live request

> [!example]
> **Before:** `$lookup` → `$unwind` → `$match` (millions of joined docs filtered late)  
> **After:** `$match` (indexed) → `$lookup` on hundreds of docs → `$group`

> [!success] Pros / Cons
> **Pros:** Same pipeline, 10×–100× faster with stage order + indexes. **Cons:** `allowDiskUse` trades RAM for disk latency — fix indexes first.

> [!tip] Interview tip
> Tie to **37s → 1s**: moving `$match` first was step 5 in Q11.

---

### 27. Unbounded array growth

> [!question] Q27
> How do you avoid the pitfalls of unbounded array growth in a document?

Arrays that grow forever (all click events on a user, every chat message in a room document) cause:

- Documents approaching **16 MB** limit
- Slower reads/writes (whole document rewritten)
- **Multikey index** bloat

**Solutions:**
- **Separate collection** — `messages` with `roomId` index, not one giant `messages[]` on `rooms`
- **Bucket pattern** — one doc per hour/day bucket of events
- **Cap array** — `$push` with `$slice: -N` (keep last N)
- **Outlier pattern** — overflow collection when count exceeds threshold

> [!example]
> **Bad:** `user.events[]` with millions of entries.  
> **Good:** `events` collection `{ userId, createdAt, type }` with index `{ userId: 1, createdAt: -1 }`.

> [!success] Pros / Cons
> **Pros:** Stable document size; better index locality. **Cons:** More collections/queries — acceptable trade-off at scale.

---

### 28. `populate` vs `$lookup`

> [!question] Q28
> How does `populate` differ from `$lookup`, and when would you choose each? (your CV)

| | **`populate` (Mongoose)** | **`$lookup` (aggregation)** |
|--|---------------------------|------------------------------|
| **Where** | Application layer — extra queries | Server-side in one pipeline |
| **Round trips** | Often N+1 or batched queries | Single aggregation command |
| **Flexibility** | Easy in controllers/services | Better for join + filter + group |
| **Index needs** | Foreign field indexed | Same |

**Choose `populate`:** Simple detail pages — load order, populate affiliate name for display.  
**Choose `$lookup`:** Server-side reports that join, filter, aggregate in **one** database round trip — or when you cannot N+1 in API.

On affiliate **list reports**, you often **denormalize** + indexed `find()` instead of either join ([[07-cv-deep-dive#9 Database structure and business workflows]]). `$lookup` after a selective `$match` is OK for admin cross-affiliate reports.

> [!example]
> ```typescript
> // populate — simple
> orders = await Order.find({ affiliateId }).populate('affiliateId', 'name email').lean();

> // $lookup — aggregate then group
> db.orders.aggregate([
>   { $match: { createdAt: { $gte: start } } },
>   { $lookup: { from: 'affiliates', localField: 'affiliateId', foreignField: '_id', as: 'aff' } },
>   { $unwind: '$aff' },
>   { $group: { _id: '$aff.region', total: { $sum: '$commission' } } }
> ]);
> ```

> [!success] Pros / Cons
> **`populate` pros:** Ergonomic in [[04-nestjs]]. **`populate` cons:** Multiple round trips; easy N+1 bug.  
> **`$lookup` pros:** One server pipeline. **`$lookup` cons:** Verbose; performance traps at scale.

---

### 29. Affiliate reporting schema and indexes (CV deep dive)

> [!question] Q29
> How would you design the MongoDB schema and indexes for the affiliate reporting queries that need to be fast? (your CV)

> [!warning] Adapt to your real fields
> Collection names, field names, and index definitions below match **typical** affiliate-platform patterns from your CV ([[07-cv-deep-dive#4 Affiliate platform architecture end to end]], [[04-nestjs]]). **Swap in your actual schema** — payout states, commission rules, admin filters — before claiming this in an interview.

**Goal:** Affiliates open a dashboard and see **their** orders, commissions, and conversion metrics for a date range — **sub-second**, not 37 seconds. Admins may need cross-affiliate rollups, but the **dominant hot path** is per-affiliate reads at scale (100k+ affiliates, hundreds of orders/day peak — see [[06-system-design]]).

---

#### Design principle: schema follows query patterns

Do not normalize like PostgreSQL by default. List the **top 5 report queries**, then shape collections and indexes around them. From [[07-cv-deep-dive#9 Database structure and business workflows]]:

1. "My completed orders this month, newest first" — paginated list  
2. "My commission total for date range" — sum aggregation  
3. "Orders by status" — filtered counts  
4. "Daily breakdown chart" — group by day  
5. Admin: "Top affiliates by GMV this week" — separate cold path / worker

---

#### Collections (template)

| Collection | Holds | Notes |
|------------|-------|-------|
| **`affiliates`** | Profile, status, payout info | Reference target; rarely joined if denormalized |
| **`referralLinks`** | `code`, `affiliateId`, campaign | Unique index on `code` |
| **`orders`** | Attribution, totals, status, timestamps | **Denormalized report fields** at write time |
| **`commissions`** | Calculated commission per order | Written by async worker ([[04-nestjs]] + RabbitMQ) |
| **`affiliateDailyStats`** | Pre-aggregated `{ affiliateId, date, orderCount, commissionSum }` | **Computed pattern** for charts |

**On `orders` at attribution time (write once, read many):**
- `affiliateId`, `referralCode`
- `status`, `paymentStatus` (whatever you actually filter on)
- `total`, `commission` (or compute in worker → `commissions`)
- `createdAt`, `completedAt`
- **Extended reference:** `affiliateName` snapshot — avoids `$lookup` on list

Embed **line items** only if bounded; otherwise reference `orderItems` collection.

---

#### Indexes (ESR per query pattern)

**Hot query — affiliate order list:**
```javascript
db.orders.createIndex({ affiliateId: 1, status: 1, createdAt: -1 });
```
- **E:** `affiliateId`, `status`  
- **S/R:** `createdAt` for sort + date range  

This is the index behind **37s → 1s** ([[07-cv-deep-dive#10 Query optimization 37s → 1s]], Q11).

**Commission lookup by order (idempotent worker):**
```javascript
db.commissions.createIndex({ orderId: 1, affiliateId: 1 }, { unique: true });
```

**Referral link resolution at checkout:**
```javascript
db.referralLinks.createIndex({ code: 1 }, { unique: true });
```

**Daily stats chart:**
```javascript
db.affiliateDailyStats.createIndex({ affiliateId: 1, date: -1 });
```

**Partial index** (optional — active affiliates only):
```javascript
db.orders.createIndex(
  { affiliateId: 1, createdAt: -1 },
  { partialFilterExpression: { status: "completed" } }
);
```

> [!warning] Adapt to your real fields
> If reports filter on `completedAt` instead of `createdAt`, index **`completedAt`**. If status values differ (`paid`, `confirmed`), index must match **exact filter values**. Run `explain()` on each hot query after creating indexes.

---

#### Read path architecture

```
Affiliate dashboard request
        │
        ▼
   NestJS report service ([[04-nestjs]])
        │
   ┌────┴────────────────────────────┐
   ▼                                 ▼
Redis cache                    MongoDB
(summary TTL 60–300s)          indexed find / dailyStats
invalidate on new order        projection + .lean()
```

1. **List view:** indexed `find` + projection + `.lean()` + **keyset pagination** (Q30)  
2. **Chart / month total:** read `affiliateDailyStats` (updated by worker) — not `$group` over raw orders on every request  
3. **Heavy admin rollup:** async job + materialized collection or export — not blocking API thread  
4. **Redis:** cache `{ affiliateId, period } → summary` with TTL; invalidate on `order.completed` event

This matches "low server load" in [[07-cv-deep-dive#11 Achieving low server load]] — indexes fix scans; cache absorbs repeat reads; queue moves commission math off checkout.

---

#### Write path (keep reads fast)

1. Order placed with referral code → resolve `affiliateId` → write denormalized fields on `orders`  
2. Publish event to RabbitMQ → commission worker **upserts** `commissions` (idempotent)  
3. Worker increments **`affiliateDailyStats`** for that UTC date (`$inc` — atomic)  
4. Invalidate Redis cache keys for that affiliate

Denormalization cost: if affiliate name changes, either accept stale snapshot on old orders or run a bounded backfill — explain this trade-off if asked.

---

#### What not to do

- `$lookup` entire `orders` × `affiliates` on every dashboard load  
- `$group` over millions of raw orders per HTTP request  
- `{ createdAt: -1 }` only index for a query that always filters by `affiliateId` first  
- Deep `skip(10000)` pagination on affiliate order history (Q30)

---

#### Spoken answer (schema question)

> "I designed around affiliate dashboard access patterns, not ER diagrams. Orders carry denormalized attribution and commission fields at write time. The main list query filters by affiliateId and status, sorts by createdAt, often with a date range — so I used a compound index `{ affiliateId: 1, status: 1, createdAt: -1 }` following ESR. That index took a report query from about 37 seconds to about one second after explain showed a collection scan. For charts I pre-aggregate into daily stats via a queue worker, cache summaries in Redis, and use lean projection on list endpoints. Admin cross-affiliate reports run async, not on the hot path."

> [!example]
> Whiteboard drawing: three boxes — `orders` (indexed), `affiliateDailyStats` (computed), Redis (cache) — with arrows from NestJS report API. Cross-ref [[06-system-design]] affiliate section.

> [!success] Pros / Cons
> **Strengths:** Shows end-to-end thinking (write path, read path, indexes, cache, async); ties schema to **measured** win (37s → 1s).  
> **Watch-outs:** Denormalization = duplication and invalidation complexity; pre-aggregates can lag seconds behind real time — state that clearly; every extra index slows order inserts slightly.

> [!tip] Interview tip — ESR + affiliate schema
> When asked "how would you design this?", answer in order: **(1)** name the #1 query, **(2)** fields on the document, **(3)** compound index with ESR, **(4)** explain() proof, **(5)** computed/cache layer for heavier views. Point to [[05-databases]] for general read vs write optimization trade-offs.

---

### 30. Pagination at scale

> [!question] Q30
> How do you handle pagination efficiently at scale (skip/limit vs range/keyset pagination)?

**Offset pagination (`skip` + `limit`):**
```javascript
db.orders.find({ affiliateId: id }).sort({ createdAt: -1 }).skip(1000).limit(20);
```
Simple but **O(skip)** — MongoDB must walk and discard 1000 documents every page. Page 500 is painfully slow.

**Keyset (cursor) pagination — preferred:**
```javascript
// First page
db.orders.find({ affiliateId: id, status: "completed" })
  .sort({ createdAt: -1, _id: -1 })
  .limit(20);

// Next page — pass last row's createdAt + _id from previous response
db.orders.find({
  affiliateId: id,
  status: "completed",
  $or: [
    { createdAt: { $lt: lastCreatedAt } },
    { createdAt: lastCreatedAt, _id: { $lt: lastId } }
  ]
}).sort({ createdAt: -1, _id: -1 }).limit(20);
```

Uses the **same compound index** as Q29 (`affiliateId`, `status`, `createdAt`). Constant time per page. **`_id` tiebreaker** prevents duplicates when `createdAt` values collide.

> [!example]
> Affiliate order history API returns `{ items, nextCursor: { createdAt, id } }` — client sends cursor, not `page=51`.

> [!success] Pros / Cons
> **Skip/limit pros:** Easy "jump to page 7" UX. **Skip/limit cons:** Does not scale.  
> **Keyset pros:** Fast at any depth; stable under inserts. **Keyset cons:** No random page jump without cursors; API contract slightly richer.

> [!tip] Interview tip
> Mention `$facet` only when you need **total count + page** — count is expensive; often approximate or separate cached count for affiliate UI.

---

## Related notes

- [[05-databases]] — SQL vs NoSQL, general indexes, ACID, CAP, caching, read/write scaling
- [[07-cv-deep-dive]] — Affiliate architecture, **37s → 1s** STAR story, RabbitMQ, Redis, low server load
- [[04-nestjs]] — Mongoose models, report modules, Redis cache, RabbitMQ workers
- [[15-sql]] — SQL `EXPLAIN`, joins, relational modeling (contrast with `$lookup`)
- [[06-system-design]] — Affiliate system design, sharding by `affiliateId`, scaling reads
- [[08-behavioral]] — Performance improvement stories (37s → 1s, 11s → 3s)
- [[questions/16-mongodb]] — Question list (companion to this answer note)

---

## References & Further Study

### MongoDB core documentation
- [MongoDB Manual](https://www.mongodb.com/docs/manual/)
- [MongoDB University](https://learn.mongodb.com/) — free courses on schema, performance, and ops

### Indexes & query performance
- [Indexes](https://www.mongodb.com/docs/manual/indexes/)
- [Compound Indexes](https://www.mongodb.com/docs/manual/core/index-compound/)
- [ESR (Equality, Sort, Range) Guide](https://www.mongodb.com/docs/manual/tutorial/equality-sort-range-rule/)
- [Explain Results](https://www.mongodb.com/docs/manual/reference/explain-results/) — backing for **37s → 1s** diagnosis
- [Covered Queries](https://www.mongodb.com/docs/manual/core/query-optimization/#covered-query)

### Aggregation
- [Aggregation Pipeline](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/)
- [Aggregation Pipeline Optimization](https://www.mongodb.com/docs/manual/core/aggregation-pipeline-optimization/)
- [`$lookup`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/lookup/)

### Replication, sharding & consistency
- [Replication](https://www.mongodb.com/docs/manual/replication/)
- [Read Preferences](https://www.mongodb.com/docs/manual/core/read-preference/)
- [Read Concern](https://www.mongodb.com/docs/manual/reference/read-concern/) & [Write Concern](https://www.mongodb.com/docs/manual/reference/write-concern/)
- [Sharding](https://www.mongodb.com/docs/manual/sharding/)
- [Shard Keys](https://www.mongodb.com/docs/manual/core/sharding-shard-key/)

### Transactions & real-time
- [Transactions](https://www.mongodb.com/docs/manual/core/transactions/)
- [Change Streams](https://www.mongodb.com/docs/manual/changeStreams/)

### Schema design
- [Data Modeling Introduction](https://www.mongodb.com/docs/manual/core/data-modeling-introduction/)
- [Schema Design Patterns (MongoDB Blog)](https://www.mongodb.com/blog/post/building-with-patterns-a-summary)

### Mongoose
- [Mongoose Documentation](https://mongoosejs.com/docs/guide.html)
- [Mongoose `.lean()`](https://mongoosejs.com/docs/tutorials/lean.html)
