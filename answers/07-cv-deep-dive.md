---
title: CV Deep Dive — Answers
topic: cv-deep-dive
tags: [interview, cv, star]
related: ["[[02-nextjs]]", "[[04-nestjs]]", "[[06-system-design]]", "[[16-mongodb]]", "[[08-behavioral]]"]
---

# CV Deep Dive — Answers

> [!abstract] How to use this note
> Each item has the **question**, a plain-English **explanation**, an **example** (spoken answer or STAR bullets), and a **Pros / Cons** box. Numbers match [[questions/07-cv-deep-dive|the questions file]].

> [!warning] Templates based on your CV
> These answers are **model templates** derived from what your resume claims. Interviewers will probe every metric — **only claim what you actually did**. Replace placeholders (field names, tools, numbers) with your real details before the interview.

---

## Rokomari Frontend (jQuery era)

### 1. Removed 1,800 lines of unoptimized code

> [!question] Q1
> You removed 1,800 lines of unoptimized code. What was wrong with it, how did you identify it, and how did you verify nothing broke?

The legacy jQuery frontend had accumulated years of dead code, duplicated logic, and inefficient DOM work. Your job was to shrink and simplify without breaking checkout, search, or analytics flows. You audited systematically, consolidated patterns, and verified with regression testing and post-release monitoring.

> [!example]
> **Situation:** The jQuery codebase had grown organically — duplicate event handlers, unused functions, and redundant AJAX calls bloated the bundle and made changes risky.
>
> **Task:** Remove ~1,800 lines of waste while keeping every user-facing flow working.
>
> **Action:** I started with static analysis (grep for unused exports, DevTools coverage to find dead JS), then traced live flows in checkout, cart, and product pages. I grouped duplicated DOM/AJAX patterns into shared helpers, removed dead branches, and batch-tested in staging across Chrome/Firefox/Safari. After deploy I watched error rates and GA event volume for 48 hours.
>
> **Result:** ~1,800 lines removed, smaller payload, faster parse time, and easier maintenance. No spike in client errors or missing analytics events.
>
> *Spoken:* "The code wasn't one bad file — it was death by a thousand cuts: three near-identical cart helpers, event listeners bound on every AJAX refresh, and functions left over from deprecated features. I used coverage plus manual flow testing, consolidated the duplicates, and monitored analytics after release to confirm nothing broke."

> [!success] Pros / Cons
> **Strengths of this answer:** Shows systematic auditing (not random deletion), risk awareness, and verification with real signals (errors + analytics). **Watch-outs:** Don't invent a single dramatic file — interviewers may ask you to open it. Have one concrete before/after example ready (e.g., "three cart modules → one helper"). Don't claim automated test coverage you didn't have.

> [!tip] Be ready to name one specific module you unified and one metric you watched post-release (error rate, bundle size, Lighthouse score).

See also [[11-web-performance#1 Core Web Vitals]].

---

### 2. Improved GA and Facebook Pixel accuracy

> [!question] Q2
> How did you improve the accuracy of data sent to Google Analytics and Facebook Pixel? What was inaccurate before?

Management relied on analytics for merchandising and ad spend, but tracking was unreliable — duplicate fires, missing purchase events, wrong parameters. You audited the data layer and event lifecycle, standardized payloads, and validated with debug tools until leadership could trust the numbers.

> [!example]
> **Situation:** Reports showed inflated page views and under-counted purchases; Facebook Pixel Helper flagged mismatched `value`/`currency` on checkout.
>
> **Task:** Make GA and Pixel data trustworthy enough for business decisions.
>
> **Action:** I mapped every tracked event to a user action (page view, add-to-cart, purchase). I fixed double-firing from both inline tags and GTM, ensured purchase events fired once on order confirmation with correct `transaction_id`, `value`, and line items, and standardized the data layer shape. I validated with GA DebugView, Pixel Helper, and side-by-side comparison against order DB samples.
>
> **Result:** Clean event streams; management reported decisions roughly **twice as fast** because they stopped second-guessing the data.
>
> *Spoken:* "The problem wasn't missing analytics — it was noisy analytics. Duplicate add-to-cart on AJAX refresh, purchase firing on payment start instead of confirmation. I aligned events to the actual lifecycle and validated with debug tools until DB order counts matched Pixel purchase counts within expected variance."

> [!success] Pros / Cons
> **Strengths:** Connects engineering work to business impact (faster decisions), shows you understand event lifecycle and validation. **Watch-outs:** Name real events you fixed — don't say "I fixed everything." If asked about "2× faster decisions," clarify it was qualitative feedback from management, not a formal A/B test, unless you have data.

---

### 3. Frontend performance optimizations (jQuery)

> [!question] Q3
> What specific frontend performance optimizations did you make in the jQuery codebase?

Before Next.js, the jQuery stack paid a high cost on every page: heavy scripts, layout thrashing, and render-blocking assets. You applied classic front-end performance techniques — deferral, delegation, lazy loading, bundling — and measured before/after with Lighthouse and DevTools.

> [!example]
> - **Script diet:** Removed render-blocking scripts; deferred non-critical JS; minified/bundled with Webpack.
> - **DOM efficiency:** Batched DOM reads/writes; replaced dozens of per-element listeners with **event delegation** on stable parents.
> - **Lazy loading:** Deferred below-the-fold images and widgets until scroll or idle.
> - **Network:** Reduced duplicate AJAX; cached static responses where safe.
> - **Measurement:** Lighthouse + Performance panel before/after on homepage, PLP, and checkout.
>
> *Spoken:* "I treated it like a performance budget problem — profile first, fix the biggest line items. The wins were fewer reflows from batched DOM updates, smaller JS from dead-code removal, and lazy-loading images below the fold."

> [!success] Pros / Cons
> **Strengths:** Practical, measurable techniques interviewers recognize. **Watch-outs:** Don't claim Core Web Vitals "green" unless you measured it. Tie at least one optimization to a number (bundle KB, Lighthouse performance score delta).

See [[11-web-performance]] and [[02-nextjs#21 Page load 11s → 3s]] for the later migration wins.

---

## Rokomari Affiliate (NestJS)

### 4. Affiliate platform architecture end to end

> [!question] Q4
> Walk me through the affiliate platform architecture end to end. What were the main components?

You bootstrapped a full affiliate system: affiliates sign up, get referral links, drive traffic, orders get attributed, commissions accrue asynchronously, and dashboards show earnings. NestJS modules, MongoDB for persistence, Redis for cache, RabbitMQ for async work, and a dashboard frontend with JWT auth.

> [!example]
> ```
> Affiliate browser / dashboard
>         │
>         ▼
>   NestJS API (auth, links, orders, commissions, reports)
>         │
>    ┌────┴────┬──────────┐
>    ▼         ▼          ▼
> MongoDB   Redis     RabbitMQ ──► Workers (commission, notify, aggregate)
> ```
>
> **Flow:** Affiliate registers → approved → generates link with ref code → click tracked → order placed with attribution → order persisted → event published to queue → worker calculates commission idempotently → report served from indexed query + Redis cache.
>
> *Spoken:* "I designed it as a modular NestJS monolith — feature modules per domain, MongoDB shaped around report queries, Redis for hot affiliate stats, and RabbitMQ so the order API stays fast while commission math runs async."

> [!success] Pros / Cons
> **Strengths:** Clear boxes-and-arrows story; separates sync path from async. **Watch-outs:** Don't overclaim microservices — it was a modular monolith unless you actually split services. Be ready to draw this on a whiteboard in 60 seconds.

See [[04-nestjs]], [[06-system-design#Affiliate / referral]], [[16-mongodb#29 Affiliate reporting schema]].

---

### 5. Access/refresh token lifecycle and invalidation

> [!question] Q5
> You built the auth flow with access/refresh tokens. Explain the full token lifecycle and how you handle invalidation.

Short-lived **access tokens** (JWT, Bearer header) authorize API calls statelessly. Long-lived **refresh tokens** live in httpOnly cookies or secure storage and mint new access tokens. Rotation + server-side storage of the current refresh hash lets you revoke sessions on logout, password change, or token reuse (theft detection).

> [!example]
> 1. **Login:** Validate credentials → issue access JWT (e.g., 15 min) + refresh token (e.g., 7 days) → store **hash** of refresh token server-side (Redis/DB) linked to user/session ID.
> 2. **API call:** Client sends `Authorization: Bearer <access>`. Gateway validates signature + expiry.
> 3. **Refresh:** Access expired → client POSTs refresh token → server verifies hash, **rotates** refresh (new token, invalidate old hash), issues new access.
> 4. **Logout / password change:** Delete session record / refresh hash → all tokens in that family dead.
> 5. **Reuse detection:** If an old rotated refresh appears again → revoke entire session family (possible theft).
>
> *Spoken:* "Access tokens stay short so compromise window is small. Refresh rotation with server-side invalidation gives me revocation without hitting the DB on every request."

> [!success] Pros / Cons
> **Strengths:** Shows security awareness beyond "we used JWT." **Watch-outs:** State exact TTLs and storage (Redis vs MongoDB) only if true. Mention httpOnly/Secure/SameSite if you used cookie-based refresh. See [[09-security#JWT]] and [[06-system-design#13 JWT auth system]].

---

### 6. Promise-based request queueing

> [!question] Q6
> Explain the JavaScript Promise-based request queueing you built for concurrent API requests. Why was it needed and how does it work?

Affiliate dashboards and bulk operations fired many parallel API calls. Uncapped concurrency could overwhelm downstream services, hit rate limits, or cause race conditions on ordered work. You built a **concurrency-limited pool** — at most N promises in flight; when one settles, the next starts.

> [!example]
> **Why:** Bulk link generation / report exports triggered 50+ simultaneous requests → 429s and timeouts.
>
> **How:** A queue drains tasks with a fixed concurrency limit:
> ```js
> async function promisePool(tasks, limit) {
>   const executing = new Set();
>   const results = [];
>   for (const task of tasks) {
>     const p = Promise.resolve().then(task);
>     results.push(p);
>     executing.add(p);
>     p.finally(() => executing.delete(p));
>     if (executing.size >= limit) await Promise.race(executing);
>   }
>   return Promise.all(results);
> }
> ```
> **Result:** Smooth throughput, no downstream overload, predictable p95 on bulk operations.
>
> *Spoken:* "It is like a bouncer at a club — only five requests inside at once. As each finishes, the next waiting task enters. Same outcome as p-limit, but I implemented it to control affiliate bulk exports."

> [!success] Pros / Cons
> **Strengths:** Concrete code + clear problem/solution. **Watch-outs:** Don't claim you invented the pattern — acknowledge libraries like `p-limit` exist; explain why custom fit (ordering, retry hooks, NestJS integration).

See [[03-nodejs#Concurrency]].

---

### 7. 13% sales increase attribution

> [!question] Q7
> How did the affiliate program contribute to a 13% sales increase? How do you attribute that?

The ~13% figure is a **program-level business metric**, not something one engineer measured alone. Affiliates drove incremental orders via tracked referral links; attribution compares affiliate-attributed revenue to a pre-program or non-affiliate baseline over the same period.

> [!example]
> **Situation:** Rokomari launched the affiliate channel to reach audiences outside owned marketing.
>
> **Task:** Build tracking trustworthy enough for business to measure channel ROI.
>
> **Action:** Every order stores `affiliateId` / ref code at checkout. Reporting aggregates affiliate-attributed GMV, order count, and conversion by cohort. Business compared year-over-year or channel-incrementality (affiliate orders that wouldn't exist without the link).
>
> **Result:** Program attributed to ~**13% sales increase** over the measurement window (confirm exact definition with your team).
>
> *Spoken:* "I own the engineering side — reliable attribution and reports. The 13% is a business metric from comparing affiliate-driven revenue to baseline; I can explain how we track ref code → order, but I'd defer exact methodology to product/finance."

> [!success] Pros / Cons
> **Strengths:** Honest separation of engineering vs business attribution. **Watch-outs:** **Never** imply you ran a rigorous incrementality study unless you did. If pressed, say: "Orders with affiliate ref codes summed to X% of total GMV; incremental lift was estimated by the business team."

---

### 8. Handling 600+ daily orders reliably

> [!question] Q8
> How did you handle 600+ daily orders reliably? What happens if a step fails?

The order API path stays **fast and non-blocking**: persist the order, publish an event, return 200. Heavy work — commission calculation, notifications, reporting updates — runs in **RabbitMQ consumers** with retries, backoff, and dead-letter queues. Consumers are **idempotent** (dedup by `orderId`) because delivery is at-least-once.

> [!example]
> **Happy path:** Order created → MongoDB write → message to `order.created` queue → worker computes commission → updates ledger → sends notification.
>
> **Failure path:** Worker throws → retry with exponential backoff (e.g., 3 attempts) → still failing → **DLQ** for manual inspection → alert on DLQ depth. API already succeeded; order exists; commission is eventually consistent.
>
> **Idempotency:** Worker checks `processedOrders` or uses upsert on `orderId` so a redelivered message never double-pays commission.
>
> *Spoken:* "600 orders/day isn't huge throughput — the challenge is correctness under retries. I never block checkout on commission math; queues absorb spikes and DLQ catches poison messages."

> [!success] Pros / Cons
> **Strengths:** Shows distributed-systems basics (async, idempotency, DLQ). **Watch-outs:** Don't claim exactly-once delivery — RabbitMQ is at-least-once. Know your retry count and DLQ monitoring.

See [[04-nestjs#RabbitMQ]], [[06-system-design#Message queues]].

---

### 9. Database structure and business workflows

> [!question] Q9
> How did you design the database structure and business workflows?

Schema followed **query patterns**, not textbook normalization. Collections for affiliates, links, orders, commissions; embed data read together, reference shared/growing data; compound indexes (ESR) for dashboard reports. Workflows map to clear service methods with async steps on the queue.

> [!example]
> **Collections (example):** `affiliates`, `referralLinks`, `orders`, `commissions`, `payouts`.
>
> **Workflows:**
> 1. Registration → admin approval → active affiliate
> 2. Link generation → unique ref code → click tracking
> 3. Order with ref → attribution on write
> 4. Commission accrual (async worker) → payout batch (scheduled)
>
> **Indexing:** `{ affiliateId: 1, status: 1, createdAt: -1 }` for "my orders this month, newest first."
>
> *Spoken:* "I designed MongoDB around how affiliates query their dashboard, not around ER diagrams. Hot reports hit compound indexes; heavy aggregation runs off the request path."

> [!success] Pros / Cons
> **Strengths:** Ties schema to access patterns — senior signal. **Watch-outs:** Know your actual collection names and one real index definition. See [[16-mongodb#9 Compound index & ESR]].

---

### 10. Query optimization 37s → 1s

> [!question] Q10
> You optimized a query from 37s to 1s using explain() and indexes. Walk through exactly what you did.

A reporting query scanned the entire collection and sorted in memory because filter/sort fields lacked a supporting index. `explain('executionStats')` showed `COLLSCAN`, high `totalDocsExamined`, and an in-memory `SORT`. A compound index following **ESR** (Equality, Sort, Range) turned it into `IXSCAN` with index-backed sort.

> [!example]
> 1. Reproduced slow query in staging with production-like data volume.
> 2. Ran `db.orders.explain('executionStats').find({ ... }).sort({ createdAt: -1 })`.
> 3. Saw `COLLSCAN`, `totalDocsExamined: 2M`, `nReturned: 500`, `SORT` stage in memory.
> 4. Added compound index: `{ affiliateId: 1, status: 1, createdAt: -1 }` (adjust to your real fields).
> 5. Added **projection** to return only fields needed for the report.
> 6. Re-ran explain → `IXSCAN`, `totalDocsExamined ≈ nReturned`, **~37s → ~1s**.
>
> *Spoken:* "The database wasn't slow — the plan was wrong. One index aligned with how we filter and sort, and the query went from scanning millions of docs to hundreds."

> [!success] Pros / Cons
> **Strengths:** Gold-standard MongoDB interview story; shows method, not luck. **Watch-outs:** **Replace field names with your real ones.** Interviewer may ask you to write the index on a whiteboard. Full version also in [[16-mongodb#11 37s → 1s walkthrough]].

---

### 11. Achieving low server load

> [!question] Q11
> How did you achieve "low server load"? What did you measure and change?

"Low server load" means you moved work off the hot request path and measured CPU, memory, and latency before/after. Caching hot reads in Redis, async processing via RabbitMQ, optimized queries/indexes, and avoiding blocking I/O on API threads.

> [!example]
> **Measured:** CPU %, memory, p95 API latency, MongoDB ops/sec (via monitoring — Datadog, PM2, or cloud metrics).
>
> **Changed:**
> - Redis cache for affiliate dashboard summaries (TTL + invalidate on new order)
> - Queue for commission/notifications
> - Compound indexes on report queries ([[#10 Query optimization 37s → 1s]])
> - `.lean()` on read-heavy Mongoose queries
>
> **Result:** Lower CPU during peak order windows; p95 report endpoint dropped after cache + index work.
>
> *Spoken:* "I didn't optimize blindly — I looked at what the API was doing per request, moved heavy math async, cached what repeats, and indexed what scans."

> [!success] Pros / Cons
> **Strengths:** Measurement + specific levers. **Watch-outs:** Have at least one real metric (e.g., "CPU dropped from ~70% to ~40% at peak" or "p95 from 800ms to 120ms") — or say "we monitored qualitatively" if you lack numbers.

---

### 12. RabbitMQ in the affiliate system

> [!question] Q12
> How does RabbitMQ fit into the affiliate system?

RabbitMQ **decouples** the synchronous order API from slow or failure-prone downstream work. It absorbs traffic spikes, enables retries, and keeps checkout responsive while commissions and notifications process reliably in the background.

> [!example]
> **Producers:** Order service publishes `{ orderId, affiliateId, amount }` to `order.created`.
>
> **Consumers:** Commission worker, email notifier, analytics aggregator — each separate queue/consumer group.
>
> **Reliability:** Ack after successful processing; nack + requeue on transient failure; DLQ after max retries; prefetch limit to avoid one consumer hoarding messages.
>
> *Spoken:* "Without the queue, every checkout would wait for commission math and emails. RabbitMQ lets the API return in milliseconds and workers catch up at their pace."

> [!success] Pros / Cons
> **Strengths:** Clear role in architecture. **Watch-outs:** Know exchange type (direct/topic), queue names if asked, and why you chose RabbitMQ over Redis streams or SQS.

See [[04-nestjs#Message queues]].

---

## Rokomari Ads (NestJS)

### 13. Three backend ad features

> [!question] Q13
> Explain the three backend features: automated ad placement, budget-based ad prioritization, and performance analytics tracking.

The AdServer automates where ads appear, ranks them by business value while respecting budgets, and tracks impressions/clicks for reporting — all without slowing the ad-serving path.

> [!example]
> 1. **Automated ad placement:** Rules engine selects eligible ads for a slot (page type, category, targeting) and injects creative metadata into the response — no manual placement per page.
> 2. **Budget-based prioritization:** Campaigns ranked by priority/remaining budget; higher-value ads win slots; pacing spreads spend across the day ([[#14 Budget-based prioritization]]).
> 3. **Performance analytics:** Impression/click events recorded async ([[#15 Analytics at scale]]); aggregated into dashboards for CTR, spend, ROI.
>
> *Spoken:* "Three problems: where to show, which ad wins when multiple qualify, and how to measure — without blocking the millisecond serve path."

> [!success] Pros / Cons
> **Strengths:** Structured triad easy to remember. **Watch-outs:** Be ready to go deeper on any one pillar — interviewers often pick one.

---

### 14. Budget-based prioritization and overspend prevention

> [!question] Q14
> How does budget-based prioritization work? How do you prevent overspending a budget?

Eligible ads compete for limited slots. Ranking considers campaign priority and remaining budget. **Pacing** spreads daily budget across hours; **atomic decrements** on serve/impression ensure you stop before overspend even under concurrency.

> [!example]
> **Selection:** `eligibleAds.filter(hasBudget).sort(by priority, then remainingBudget)` → top N for slot.
>
> **Pacing:** Daily budget / hours remaining = max spend this hour; throttle low-priority serves near cap.
>
> **Overspend guard:** Atomic `$inc: { spent: cost }` with condition `{ spent: { $lte: budget - cost } }` — if modify count is 0, budget exhausted → exclude from rotation.
>
> *Spoken:* "Race conditions are the trap — two requests can't both spend the last taka. Atomic updates in MongoDB make budget checks and decrements one operation."

> [!success] Pros / Cons
> **Strengths:** Shows concurrency awareness + business logic. **Watch-outs:** Don't claim real-time bidding unless you built it. Clarify impression vs click billing if applicable.

---

### 15. Analytics at scale without slowing ad delivery

> [!question] Q15
> How did you track performance analytics at scale without slowing down ad delivery?

The serve path returns the ad creative immediately. Impression/click tracking is **fire-and-forget**: enqueue events or atomic counter increments async; aggregation runs in batch workers or scheduled jobs; hot summary stats cached in Redis.

> [!example]
> **Serve path (< 50ms target):** Lookup ad → return JSON/HTML → **do not** await analytics write.
>
> **Tracking:** Publish `{ adId, slot, timestamp, type: 'impression' }` to queue OR `$inc` on sharded counter collection.
>
> **Aggregation:** Worker rolls up hourly/daily stats; dashboard reads pre-aggregated docs or Redis cache.
>
> *Spoken:* "Analytics is eventually consistent by design — delivery is synchronous, counting is async. That separation kept ad latency flat as event volume grew."

> [!success] Pros / Cons
> **Strengths:** Classic latency/throughput trade-off done right. **Watch-outs:** Acknowledge eventual consistency in reports (minutes delay acceptable?).

See [[11-web-performance#Analytics]].

---

### 16. Refactoring REST APIs for maintainability

> [!question] Q16
> How did you refactor the REST APIs for maintainability?

Legacy endpoints had inconsistent shapes, duplicated validation, and fat controllers. You introduced DTOs + class-validator, module/service separation, standardized error responses, and removed duplication so new ad features shipped faster.

> [!example]
> **Before:** Controllers with inline logic, mixed response shapes, copy-pasted Mongo queries.
>
> **After:**
> - DTOs for every input (`CreateCampaignDto`) with validation pipes
> - Thin controllers → services → repositories
> - Global exception filter → consistent `{ statusCode, message, error }`
> - Shared utilities for pagination, auth guards, logging
>
> **Result:** New endpoints follow a template; review time dropped; fewer production bugs from bad input.
>
> *Spoken:* "Maintainability is consistency — same validation, same errors, same layer boundaries. Refactoring wasn't cosmetic; it made the next feature cheaper."

> [!success] Pros / Cons
> **Strengths:** Shows engineering maturity beyond features. **Watch-outs:** Have one before/after endpoint example. Don't claim 100% test coverage unless true.

See [[04-nestjs#DTOs and validation]].

---

## Rokomari Next.js Migration

### 17. What leading the migration involved

> [!question] Q17
> You LED the migration. What did leading involve — planning, coordination, decisions?

Leading meant owning the **technical direction** and **rollout risk**, not just writing React components. You planned the strangler-fig strategy, made architecture calls, coordinated UI/UX and backend, set performance budgets, and ensured SEO/analytics parity.

> [!example]
> **Planning:** Inventory pages by risk/traffic; define migration waves; performance budget (LCP < 2.5s).
>
> **Coordination:** Weekly sync with design (component parity), backend (API contracts), QA (regression matrix).
>
> **Decisions:** App vs Pages Router (or hybrid), SSR vs ISR per page type, shared component library, Docker/CI pipeline.
>
> **De-risking:** Load tests before cutover; feature flags / rewrites for instant rollback; analytics validation checklist per page.
>
> *Spoken:* "Leadership here was making the migration boring — incremental cutover, measurable parity, clear rollback. I coded too, but my main job was ensuring we didn't break revenue."

> [!success] Pros / Cons
> **Strengths:** Shows lead scope beyond IC coding. **Watch-outs:** Use "I led" only if true — clarify if a manager co-owned decisions. Pair with [[08-behavioral#6. Led Next.js migration]].

See [[02-nextjs#17–27 CV migration answers]].

---

### 18. Why migrate to Next.js

> [!question] Q18
> Why migrate to Next.js at all? What was the business/technical justification?

The legacy jQuery/SSR stack was slow to change and slow to load. Next.js offered React component reuse, TypeScript safety, built-in routing/code-splitting, SSR/SSG/ISR for SEO, and image/font optimization — improving both **developer velocity** and **Core Web Vitals** on a high-traffic e-commerce site.

> [!example]
> **Business case:** Faster pages → better conversion and SEO ranking; faster feature delivery → competitive catalog/UX.
>
> **Technical case:** 11s page loads, 2.1GB Docker images, duplicated jQuery patterns, hard-to-test UI.
>
> **Next.js wins:** File-based routing, automatic code splitting, `next/image`, `next/font`, ISR for product catalog, standalone Docker output.
>
> *Spoken:* "We didn't migrate for hype — the old stack was a tax on every feature and every page view. Next.js addressed performance and maintainability in one platform."

> [!success] Pros / Cons
> **Strengths:** Business + technical dual justification. **Watch-outs:** Acknowledge migration cost (months, dual maintenance during strangler phase).

---

### 19. Incremental migration without breaking the live site

> [!question] Q19
> How did you migrate incrementally without breaking the live site?

**Strangler-fig pattern:** route traffic page-by-page to Next.js via reverse-proxy rewrites while legacy serves unmigrated routes. Low-risk pages first; validate SEO/analytics/performance parity; shared design system for visual consistency; redirects preserve URL equity.

> [!example]
> 1. Stand up Next.js behind nginx/load balancer.
> 2. Rewrite `/about`, `/blog/*` → Next.js; everything else → legacy.
> 3. Per-page checklist: meta tags, canonical, JSON-LD, GA events, visual diff.
> 4. Migrate PLP → PDP → checkout funnel last (highest revenue risk).
> 5. Shared component library (buttons, cards, header) for pixel parity.
>
> *Spoken:* "The site never switched off — we peeled routes one at a time. Any page could roll back to legacy with a rewrite rule change in minutes."

> [!success] Pros / Cons
> **Strengths:** Industry-standard pattern; shows risk management. **Watch-outs:** Mention cookie/session sharing if checkout spanned both stacks.

See [[02-nextjs#19 Incremental migration]].

---

### 20. Docker optimization 2.1GB → 170MB

> [!question] Q20
> Explain the Docker optimization from 2.1GB to 170MB in detail.

The original image copied full `node_modules`, dev dependencies, and build cache into production. **Multi-stage build** + Next.js **`output: 'standalone'`** copies only traced runtime deps into a minimal Alpine runner.

> [!example]
> **Stages:**
> 1. **deps** — `npm ci` production deps only
> 2. **builder** — `next build` with standalone output
> 3. **runner** — Alpine, copy `.next/standalone`, `.next/static`, `public` only
>
> **Also:** `.dockerignore` (exclude `node_modules`, `.git`, tests); non-root user; no devDependencies in final layer.
>
> **Result:** ~2.1GB → ~170MB (~12× smaller) → faster deploys, less registry/storage cost, quicker cold starts.
>
> *Spoken:* "Standalone tracing asks Next.js which files the server actually imports — you stop shipping the entire node_modules universe."

> [!success] Pros / Cons
> **Strengths:** Quantified, reproducible, ties to ops wins. **Watch-outs:** Know your actual base image and final size. Full Dockerfile in [[14-docker-deployment]] and [[02-nextjs#23 Docker 2.1GB → 170MB]].

---

### 21. Page load 11s → 3s

> [!question] Q21
> Explain how you reduced page load from 11s to 3s.

Data-driven: Lighthouse + DevTools identified the biggest costs — oversized JS bundles, unoptimized images, blocking fonts, no code splitting. You attacked top contributors iteratively and re-measured after each change.

> [!example]
> **Actions:**
> - Code splitting + dynamic imports for below-fold components
> - `next/image` with proper sizes/formats (WebP/AVIF)
> - `next/font` to eliminate render-blocking font CSS
> - ISR/SSG for catalog pages; SSR only where needed
> - Aggressive caching headers + CDN for static assets
> - Enhanced lazy loading ([[#23 Lazy loading enhancements]])
>
> **Measurement:** Lighthouse on 3G throttled + RUM field data if available.
>
> **Result:** ~**11s → ~3s** on key templates (confirm which pages — homepage, PLP, PDP).
>
> *Spoken:* "We didn't guess — Lighthouse said JS and images were 80% of the problem. Each sprint knocked off the next biggest bar on the waterfall."

> [!success] Pros / Cons
> **Strengths:** Methodical profiling story. **Watch-outs:** Specify **which page** and **which metric** (LCP, fully loaded, TTI). Lab vs field can differ.

See [[11-web-performance]], [[02-nextjs#24 Page load 11s → 3s]].

---

### 22. Load testing

> [!question] Q22
> How did you conduct load testing and what did you find?

Before peak traffic events (sales, campaigns), you simulated concurrent users with k6 or Artillery, measured p95/p99 latency and error rates, found bottlenecks, optimized, and re-tested until SLOs held.

> [!example]
> **Setup:** k6 script — browse homepage → category → product → add to cart; ramp 50 → 500 VUs over 10 min.
>
> **Found:** p95 spiked at 400 VUs — slow Mongo queries on recommendation endpoint; memory climb on SSR pods; missing CDN cache headers on static chunks.
>
> **Fixed:** Index + Redis cache on hot query; horizontal pod scaling; CDN cache rules.
>
> **Re-test:** p95 under target at 500 VUs; error rate < 0.1%.
>
> *Spoken:* "Load testing told us where the migration wasn't finished — not CPU on Next.js itself, but a backend endpoint the new pages called more aggressively."

> [!success] Pros / Cons
> **Strengths:** Shows production mindset. **Watch-outs:** Name the tool you actually used. Don't claim 10k RPS unless you tested it.

---

### 23. Lazy loading enhancements

> [!question] Q23
> How did you enhance the lazy loading system?

Initial payload only loads above-the-fold content. Below-fold components, heavy widgets, and images load on scroll, idle, or interaction via dynamic imports and `next/image` native lazy loading.

> [!example]
> - `dynamic(() => import('./Reviews'), { loading: () => <Skeleton /> })` for below-fold sections
> - `next/image` without `priority` lazy-loads offscreen images automatically
> - Split large vendor chunks (charts, rich editors) out of the main bundle
> - Intersection Observer for custom widgets (legacy jQuery parity during migration)
>
> *Spoken:* "First paint wins — reviews, recommendations, and footer widgets don't belong in the critical path."

> [!success] Pros / Cons
> **Strengths:** Concrete techniques linked to LCP/TBT improvement. **Watch-outs:** Mention skeleton/placeholder to avoid CLS from lazy content popping in.

See [[02-nextjs#6 next/image]].

---

## Rokomari WebChat (NestJS)

### 24. Real-time WebChat architecture

> [!question] Q24
> Walk me through the real-time WebChat architecture (NestJS, Redis, RabbitMQ, Crisp).

A NestJS WebSocket gateway handles real-time client connections. Redis provides caching, presence, and pub/sub for horizontal scale. RabbitMQ processes async work (bot replies, Crisp sync, notifications). MongoDB persists messages. Crisp integrates external support workflows.

> [!example]
> ```
> Browser (WebSocket)
>       │
>       ▼
> NestJS Gateway (rooms, auth on connect, ack delivery)
>       │
>  ┌────┴─────┬──────────┐
>  ▼          ▼          ▼
> Redis    MongoDB    RabbitMQ ──► Bot worker / Crisp sync / Email
> (cache,   (messages)
>  pub/sub,
>  presence)
>       │
>       ▼
> Crisp API (external support platform)
> ```
>
> **Scale:** Multiple NestJS instances subscribe to Redis pub/sub — message to room X publishes to channel, all instances push to their local sockets in room X.
>
> *Spoken:* "WebSockets for realtime, Redis so multiple servers act as one, RabbitMQ for anything that can be async, Crisp for the support team's tooling."

> [!success] Pros / Cons
> **Strengths:** Full-stack realtime story. **Watch-outs:** Explain auth on WebSocket connect (JWT query param or cookie). Know reconnect/backoff behavior.

See [[04-nestjs#WebSockets]], [[06-system-design#Chat system]].

---

### 25. Automated bot reply system

> [!question] Q25
> How does the automated bot reply system work?

Common customer questions (order status, return policy, shipping) get instant automated replies without waiting for a human agent. Intent matching routes known queries to templated or rule-based responses; unknown queries escalate to the human queue.

> [!example]
> 1. Message arrives → persisted → published to `message.received` queue.
> 2. Bot worker classifies intent (keyword rules, regex, or simple ML — use what you built).
> 3. **Match:** Generate reply from template + user context (order lookup by phone/email) → send via gateway → mark resolved-by-bot.
> 4. **No match:** Assign to human agent queue in Crisp → notify agent.
>
> *Spoken:* "The bot handles the repetitive 40% so agents focus on exceptions. It fails open — if unsure, it never guesses; it escalates."

> [!success] Pros / Cons
> **Strengths:** Shows product thinking (deflection + escalation). **Watch-outs:** Don't overclaim NLP/LLM unless you used one. Be honest about rule-based vs ML.

---

### 26. Redis caching in WebChat

> [!question] Q26
> How did you use Redis caching here and what improved?

Redis cached hot session/conversation context, user profile snippets, and recent messages so every inbound message didn't hit MongoDB. Pub/sub and presence tracking also lived in Redis for multi-instance coordination.

> [!example]
> **Cached:** `conv:{id}:meta`, `user:{id}:profile`, last N messages per conversation (TTL 15–30 min).
>
> **Invalidation:** On new message → update cache + persist MongoDB async or sync depending on criticality.
>
> **Pub/sub:** `channel:room:{id}` broadcasts to all NestJS instances.
>
> **Result:** Lower MongoDB read ops; faster bot context lookup; sub-100ms reply prep for cached paths.
>
> *Spoken:* "Chat is read-heavy — the same conversation context gets fetched on every message. Redis cut repeated DB reads dramatically."

> [!success] Pros / Cons
> **Strengths:** Clear cache-aside pattern. **Watch-outs:** Mention cache invalidation strategy — stale context is a UX bug.

See [[05-databases#Redis]].

---

### 27. Webhook reliability

> [!question] Q27
> How did you handle webhooks and ensure reliability?

Webhook providers (Crisp, payment, etc.) retry on non-2xx and may deliver duplicates out of order. You verify signatures, respond **200 immediately**, process async on the queue, and enforce **idempotency** with processed event IDs.

> [!example]
> 1. Receive webhook → verify HMAC signature ([[09-security#Webhooks]]).
> 2. Check `processedEvents` for `eventId` — if seen, return 200 (duplicate).
> 3. Return 200 within timeout (< 3s).
> 4. Enqueue payload for worker processing.
> 5. Worker processes → marks `eventId` processed (upsert).
>
> *Spoken:* "Fast ack, slow work — providers stop retry-storming you, and idempotency handles the duplicates their retries create."

> [!success] Pros / Cons
> **Strengths:** Production-grade webhook pattern. **Watch-outs:** Know your signature scheme and where idempotency keys are stored.

See [[04-nestjs#21 Reliable webhooks]].

---

### 28. 78% customer satisfaction

> [!question] Q28
> How did you achieve 78% customer satisfaction? How is that measured and what did you change?

Post-chat CSAT surveys (thumbs up/down or 1–5 stars) measured satisfaction. Engineering improvements — faster bot replies, Redis caching, reliable delivery, lower latency — contributed to a better support experience and the **78%** positive rating.

> [!example]
> **Measurement:** Crisp (or in-app) post-session survey → % positive responses over rolling 30 days.
>
> **Changes:**
> - Bot deflection for common queries (instant vs 5-min wait)
> - Redis caching → faster agent context load
> - Reliable WebSocket delivery with acks/reconnect
> - Reduced server load → fewer timeouts during peak
>
> **Result:** CSAT improved to ~**78%** (confirm baseline and timeframe with your team).
>
> *Spoken:* "CSAT is a product metric — I connect engineering work to its inputs: speed, reliability, and first-response time. I didn't own the survey, but I owned the systems that agents and customers feel."

> [!success] Pros / Cons
> **Strengths:** Links tech to customer outcome. **Watch-outs:** Separate correlation from causation — "contributed to" not "I single-handedly achieved." Know survey methodology (sample size, selection bias).

---

## General / cross-cutting

### 29. Project you're most proud of

> [!question] Q29
> Which project are you most proud of and why?

Pick **one** project with clear ownership, difficulty, and measurable impact. Strong choices: **Next.js migration** (leadership + 11s→3s + 170MB Docker) or **affiliate platform** (zero-to-one + 100k+ affiliates + business impact).

> [!example]
> *Next.js migration:*
> **S:** Legacy frontend slowing the business; high traffic e-commerce.
> **T:** Lead migration without downtime.
> **A:** Strangler-fig rollout, performance work, cross-team coordination.
> **R:** 11s→3s, 2.1GB→170MB, no major outage — site faster and team shipping faster in React.
>
> *Why proud:* "It combined technical depth, leadership, and user-visible wins — rare to get all three."
>
> *Spoken:* "I'm most proud of leading the Next.js migration because the risk was real — break checkout and you break the company — and we pulled it off with measurable wins."

> [!success] Pros / Cons
> **Strengths:** Passion + evidence. **Watch-outs:** One project only — don't ramble through your whole CV. End with **your specific role**, not "the team."

Also pairs with [[08-behavioral#5. Ownership]] and [[08-behavioral#6. Led migration]].

---

### 30. Hardest production bug or incident

> [!question] Q30
> Tell me about the hardest bug or incident you faced in production and how you resolved it.

Structure with STAR. Good CV-aligned stories: the **37s report query** grinding production DB, a **RabbitMQ DLQ flood** from a bad deploy, a **WebSocket reconnect storm**, or a **tracking double-fire** corrupting analytics.

> [!example]
> **S:** Affiliate dashboard reports timed out at peak; DB CPU pegged 100%.
> **T:** Restore reports without taking checkout offline.
> **A:** Checked slow query log → ran `explain()` → found COLLSCAN on 2M orders. Added compound index in staging, validated plan, deployed during low-traffic window. Added query timeout + monitoring alert on `totalDocsExamined`.
> **R:** Reports back in ~1s; CPU normalized; added index review to PR checklist.
>
> *Spoken:* "The hardest part wasn't the fix — it was diagnosing under pressure while stakeholders wanted hourly updates. I communicated status every 30 minutes and fixed root cause, not symptoms."

> [!success] Pros / Cons
> **Strengths:** Shows calm debugging + prevention. **Watch-outs:** Pick an incident **you** drove, not vague "we had an outage." Include what you'd do differently (alerting, runbook).

See [[08-behavioral#14. Production incident]], [[16-mongodb#10 explain()]].

---

### 31. Fresher to Software Engineer — biggest learning

> [!question] Q31
> You went from fresher to Software Engineer. What was the biggest thing you learned?

The shift from **task executor** to **outcome owner** — designing for scale, communicating trade-offs to non-engineers, validating with metrics, and taking responsibility for production behavior, not just merged PRs.

> [!example]
> *Spoken:* "As a fresher I optimized for 'does my PR work?' As an engineer I optimize for 'does the system work for users at 2 AM?' The affiliate platform taught me that — my first backend project where my schema choices and queue design mattered to real money. The migration taught me leadership is making other people's work easier, not being the fastest coder."
>
> **Concrete moments:** Bootstrapping affiliate alone; leading migration; owning the 37s query in production.

> [!success] Pros / Cons
> **Strengths:** Shows growth mindset with evidence. **Watch-outs:** Avoid humble-brag without substance. Tie learning to a **specific story**, not platitudes.

See [[08-behavioral#16. Fresher → engineer journey]].

---

### 32. Redesign one system today

> [!question] Q32
> If you could redesign one of these systems today, what would you do differently?

Pick **one system** and name **concrete, justified** improvements — shows you've grown since shipping v1. Good answers: observability, outbox pattern, automated testing, event-driven boundaries, or ISR + tag revalidation.

> [!example]
> **System:** Affiliate platform.
>
> **Today I'd add:**
> 1. **Transactional outbox** — publish to RabbitMQ in same DB transaction as order write (no lost events on crash).
> 2. **OpenTelemetry tracing** — correlate API → queue → worker in one trace ID.
> 3. **Contract tests** on queue message schemas — prevent deploy mismatches.
> 4. **Read models** — precomputed commission summaries instead of live aggregation.
>
> *Spoken:* "V1 optimized for shipping fast. V2 would optimize for operability — I'd know exactly which order failed commission calc and why, without grep-ing logs."

> [!success] Pros / Cons
> **Strengths:** Shows architectural maturity and humility about v1 trade-offs. **Watch-outs:** Don't trash your old work — frame as "what I'd add with more time/budget." One system, 2–4 concrete changes max.

See [[06-system-design#Outbox pattern]], [[02-nextjs#32 Caching/revalidation]].

---

## Related notes

- [[02-nextjs]] — migration mechanics, Docker standalone, rendering, lazy loading.
- [[04-nestjs]] — auth, RabbitMQ, WebSockets, DTOs, webhooks.
- [[06-system-design]] — affiliate architecture, chat, JWT auth, outbox pattern.
- [[16-mongodb]] — explain(), ESR indexes, affiliate reporting schema (Q10).
- [[08-behavioral]] — STAR delivery of the same stories (ownership, leadership, incidents).
- [[11-web-performance]] — theory behind 11s→3s and jQuery optimizations.
- [[14-docker-deployment]] — multi-stage Dockerfile for 2.1GB→170MB.
- [[09-security]] — JWT, webhook signatures, cookie auth.

## References & Further Study

- [STAR method (MIT CAPD)](https://capd.mit.edu/resources/the-star-method-for-behavioral-interviews/) — Situation, Task, Action, Result framework for every CV story.
- [Amazon Leadership Principles](https://www.amazon.jobs/content/en/our-workplace/leadership-principles) — map your stories to Customer Obsession, Ownership, Dive Deep, Deliver Results.
- [MongoDB explain docs](https://www.mongodb.com/docs/manual/reference/explain-results/) — backing for the 37s→1s story.
- [Next.js standalone output](https://nextjs.org/docs/app/api-reference/config/next-config-js/output) — Docker size optimization.
- [Strangler fig pattern (Martin Fowler)](https://martinfowler.com/bliki/StranglerFigApplication.html) — incremental migration pattern.
