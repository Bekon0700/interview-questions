---
title: Node.js — Answers
topic: nodejs
tags: [interview, fullstack, nodejs, backend]
related: ["[[00-javascript]]", "[[04-nestjs]]", "[[06-system-design]]", "[[14-docker-deployment]]"]
---

# Node.js — Answers

> [!abstract] How to use this note
> Each item has the **question**, a plain-English **explanation**, an **example**, and a **Pros / Cons** box. Numbers match [[questions/03-nodejs|the questions file]]. Node underpins your NestJS backend work.

---

## Beginner

### 1. What Node is

> [!question] Q1
> What is Node.js and what is it good/bad at? Why is it suitable for I/O-heavy apps?

Node.js is a way to run JavaScript **outside the browser** (on servers), built on Google's V8 engine. Its key trait is a **non-blocking, event-driven** model: instead of waiting for slow work (network, disk), it hands that work off and keeps serving other requests, being notified when it is done.

> [!example]
> ```js
> // handles thousands of these concurrently on one thread
> app.get('/user/:id', async (req, res) => {
>   const user = await db.find(req.params.id); // I/O offloaded, thread free
>   res.json(user);
> });
> ```

> [!success] Pros / Cons
> **Pros:** excellent for I/O-heavy, concurrent workloads (APIs, real-time, proxies); one language across frontend/backend; huge ecosystem. **Cons:** poor for CPU-heavy work (a heavy computation blocks the single thread and stalls everyone — see [[#14 CPU-bound tasks]]).

### 2. npm / npx and lockfiles

> [!question] Q2
> What is `npm` vs `npx`? What is `package.json` vs `package-lock.json`?

`npm` installs and manages packages; `npx` **runs** a package's command without installing it globally (e.g. `npx create-next-app`). `package.json` lists your dependencies with flexible version ranges; `package-lock.json` records the **exact** versions installed, so everyone (and CI) gets the same tree.

> [!success] Pros / Cons
> **Committing the lockfile (do this):** reproducible installs, no "works on my machine". **Con of ignoring it:** different machines resolve different versions → mysterious bugs. `npx`: convenient one-off runs; con: downloads each time if not cached.

### 3. CommonJS vs ES Modules

> [!question] Q3
> What is the difference between CommonJS (`require`) and ES Modules (`import`)?

**CommonJS** (`require`/`module.exports`) is Node's original, synchronous module system. **ES Modules** (`import`/`export`) are the modern standard — statically analysable (enables tree-shaking), support top-level `await`, and load asynchronously. Enable ESM via `"type": "module"` or `.mjs`.

> [!example]
> ```js
> // CommonJS
> const fs = require('fs'); module.exports = foo;
> // ESM
> import fs from 'fs'; export default foo;
> ```

> [!success] Pros / Cons
> **ESM (future):** standard, tree-shakeable, top-level await. Con: interop quirks (no `__dirname` by default, some CJS packages need special import). **CommonJS:** universal in the Node ecosystem, simple. Con: not statically analysable, being superseded.

### 4. Globals

> [!question] Q4
> What are `process.env`, `process.argv`, and `__dirname`?

`process.env` = environment variables (config/secrets). `process.argv` = the array of command-line arguments. `__dirname` = the absolute path of the current file's folder (CommonJS; in ESM derive it from `import.meta.url`).

> [!example]
> ```js
> const port = process.env.PORT || 3000;
> console.log(process.argv[2]); // first user arg
> ```

> [!success] Pros / Cons
> **`process.env` for config (12-factor):** flexible per-environment, keeps secrets out of code. **Con/watch-out:** env vars are strings (parse numbers/bools), and missing ones fail silently unless you validate at startup ([[#16 Config & secrets]]).

### 5. Sync vs async APIs

> [!question] Q5
> What is the difference between synchronous and asynchronous APIs in Node (e.g., `fs.readFileSync` vs `fs.readFile`)?

**Sync** APIs (`readFileSync`) **block** the single thread until they finish — fine at startup, harmful in request handlers because they freeze every other request. **Async** APIs (`readFile` callback, or `fs.promises`) offload the work and keep serving others.

> [!example]
> ```js
> const data = fs.readFileSync('big.txt'); // blocks everyone (bad in a handler)
> const data2 = await fs.promises.readFile('big.txt'); // non-blocking (good)
> ```

> [!success] Pros / Cons
> **Async (prefer in request paths):** keeps the server responsive under load. Con: slightly more complex code. **Sync:** simpler, acceptable in CLI scripts/startup. Con: a single sync call in a hot path can tank throughput.

### 6. Dependency types

> [!question] Q6
> What is `package.json`'s `dependencies` vs `devDependencies` vs `peerDependencies`?

`dependencies` = needed at runtime in production. `devDependencies` = only for development/build/test (compilers, linters, test runners). `peerDependencies` = a version of a package the **host app** must provide (used by libraries/plugins) to avoid duplicate copies.

> [!success] Pros / Cons
> **Correct classification:** smaller production installs/images (skip dev deps with `npm ci --omit=dev`), fewer conflicts. **Con of getting it wrong:** either bloated production images or missing runtime packages. Ties to [[14-docker-deployment#9 Reducing image size]].

### 7. Express middleware

> [!question] Q7
> What is middleware in Express? Give the signature and an example.

Middleware are functions `(req, res, next)` that run **in order** for each request. They can read/modify `req`/`res`, end the response, or call `next()` to pass control to the next one. Error middleware takes four args `(err, req, res, next)`.

> [!example]
> ```js
> app.use((req, res, next) => { console.log(req.method, req.url); next(); });
> ```

> [!success] Pros / Cons
> **Pros:** clean, composable cross-cutting concerns (logging, auth, parsing). **Con:** order matters (a misplaced middleware breaks the chain); forgetting `next()` hangs the request.

### 8. Express error handling

> [!question] Q8
> How do you handle errors in an Express app?

Wrap async handlers so rejections reach an error handler (a wrapper or `express-async-errors`), and define a **final** error middleware `(err, req, res, next)` that logs and returns a consistent JSON error with the right status code.

> [!example]
> ```js
> app.use((err, req, res, next) => {
>   logger.error(err);
>   res.status(err.status || 500).json({ error: 'Something went wrong' });
> });
> ```

> [!success] Pros / Cons
> **Central error handling:** consistent responses, one place to log, no leaked stack traces. **Con/watch-out:** async errors do **not** reach it automatically in older Express — you must forward them (`next(err)` or a wrapper). NestJS solves this with exception filters ([[04-nestjs#12 Exception filters]]).

---

## Intermediate

### 9. Event loop phases

> [!question] Q9
> Explain the Node.js event loop phases (timers, pending callbacks, poll, check, close). (commonly asked at Amazon, PayPal)

Node's event loop (via libuv) cycles through phases, each with its own queue:
1. **timers** — due `setTimeout`/`setInterval` callbacks.
2. **pending callbacks** — some deferred system callbacks.
3. **poll** — get new I/O events and run their callbacks (the loop mostly waits here).
4. **check** — `setImmediate` callbacks.
5. **close callbacks** — e.g. `socket.on('close')`.
After **each** callback, the microtask queues drain: `process.nextTick` first, then promise callbacks.

> [!success] Pros / Cons
> **Pro:** understanding phases explains ordering and why timers are not exact. **Con/watch-out:** it is more complex than the browser event loop; `nextTick` runs before everything and can starve I/O if abused. Browser version: [[00-javascript#16 Event loop]].

> [!info] Further study
> - [Node.js docs: The event loop](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick)

### 10. nextTick vs setImmediate vs setTimeout

> [!question] Q10
> What is the difference between `process.nextTick`, `setImmediate`, and `setTimeout`?

`process.nextTick` runs **immediately** after the current operation, before the loop continues (highest priority). `setImmediate` runs in the **check** phase (after I/O). `setTimeout(fn, 0)` runs in the **timers** phase on the next iteration.

> [!example]
> ```js
> setTimeout(() => console.log('timeout'));
> setImmediate(() => console.log('immediate'));
> process.nextTick(() => console.log('nextTick'));
> // nextTick first; timeout vs immediate order can vary
> ```

> [!success] Pros / Cons
> **`nextTick`:** run something before the loop proceeds. Con: overuse starves I/O and timers. **`setImmediate`:** run after I/O without starving. **`setTimeout(0)`:** roughly "next tick timer", least precise.

### 11. Streams

> [!question] Q11
> What are streams? What are the four types, and why use them? (commonly asked at Netflix)

Streams process data **piece by piece** instead of loading it all into memory. Four types: **Readable** (source, e.g. a file read), **Writable** (sink, e.g. an HTTP response), **Duplex** (both, e.g. a socket), **Transform** (modifies data as it flows, e.g. gzip).

> [!example]
> ```js
> const { pipeline } = require('stream/promises');
> await pipeline(fs.createReadStream('in.txt'), zlib.createGzip(), fs.createWriteStream('out.gz'));
> ```

> [!success] Pros / Cons
> **Pros:** constant low memory even for huge files, faster time-to-first-byte, composable via `pipe`/`pipeline`. **Cons:** the API and backpressure ([[#12 Backpressure]]) are harder to reason about than reading a whole buffer.

### 12. Backpressure

> [!question] Q12
> What is backpressure and how do streams handle it?

Backpressure is when a fast producer (Readable) outpaces a slow consumer (Writable). Streams signal it: `write()` returns `false` when the buffer is full (pause), and a `drain` event tells you to resume. `pipe()`/`pipeline` handle this automatically.

> [!success] Pros / Cons
> **Handling it (via pipeline):** bounded memory, no crashes on big data. **Con of ignoring it:** memory grows unbounded and the process can crash; `pipeline` also handles errors/cleanup that manual `pipe` does not.

### 13. cluster vs worker_threads

> [!question] Q13
> What is the difference between the `cluster` module and `worker_threads`? When use each?

**`cluster`** forks multiple Node **processes** (separate memory) sharing a port — for scaling I/O apps across CPU cores. **`worker_threads`** run JS on multiple **threads** in one process, able to share memory via `SharedArrayBuffer` — for CPU-bound work without blocking the main thread.

> [!success] Pros / Cons
> **`cluster`:** simple horizontal scaling on one machine, isolation (a crash kills one worker). Con: no shared memory, IPC overhead. **`worker_threads`:** true parallel computation, shared memory. Con: more complex, only helps CPU-bound work. **Rule:** cluster to scale I/O, worker_threads to parallelise computation.

### 14. CPU-bound tasks

> [!question] Q14
> How does Node handle CPU-bound tasks and why can they be problematic?

Node runs your JavaScript on **one** thread, so a heavy synchronous computation (image processing, big loops, crypto) blocks the event loop and stalls **all** other requests. Mitigate by offloading to `worker_threads`, chunking work to yield to the loop, moving it to a separate service/queue, or using native/streamed processing.

> [!success] Pros / Cons
> **Offloading (worker threads/queue):** keeps the API responsive. **Con of not doing it:** one heavy request freezes the whole server. Trade-off: worker threads/services add complexity vs the simplicity of single-threaded code.

### 15. Buffers

> [!question] Q15
> What are `Buffer`s and when do you need them?

A `Buffer` is a fixed-size container for **raw binary data**, stored outside the V8 heap. You need them for binary protocols, file/network I/O, cryptography, and encoding conversions — anything that is not naturally a UTF-8 string.

> [!success] Pros / Cons
> **Pros:** efficient binary handling, direct byte access. **Con/watch-out:** manual size management; misusing encodings corrupts data; security-sensitive (older `new Buffer` was unsafe — use `Buffer.alloc`).

### 16. Config & secrets

> [!question] Q16
> How do you manage environment configuration and secrets in a Node app?

Load config from **environment variables** (12-factor): `.env` + dotenv in development, injected env vars or a secrets manager in production. **Validate** config at startup with a schema so missing/invalid values fail fast. Never commit secrets.

> [!success] Pros / Cons
> **Pros:** same code across environments, secrets stay out of the repo, early failure on misconfig. **Con:** env vars are stringly-typed and flat; a secrets manager adds infrastructure. See [[09-security#12 Secrets handling]].

### 17. exports vs module.exports

> [!question] Q17
> What is the difference between `exports` and `module.exports`?

`module.exports` is the **actual** object returned by `require`. `exports` is just a shortcut initially pointing at the same object. Adding properties to `exports` works; **reassigning** `exports = ...` breaks the link (only reassigning `module.exports` changes what is exported).

> [!example]
> ```js
> exports.foo = 1;              // works
> module.exports = { bar: 2 };  // works (replaces exports)
> exports = { baz: 3 };         // does NOT export baz (link broken)
> ```

> [!success] Pros / Cons
> **Watch-out:** this subtlety is a classic bug/interview trap. Prefer being explicit with `module.exports`, or use ES Modules where `export` is clearer.

### 18. Uncaught errors

> [!question] Q18
> How do you handle uncaught exceptions and unhandled promise rejections?

Listen to `process.on('uncaughtException')` and `process.on('unhandledRejection')` to **log and alert**, but treat them as **fatal** — the process is in an unknown state, so exit and let a process manager (PM2, Kubernetes) restart it. Prefer preventing them with proper `try/catch` and `.catch`.

> [!success] Pros / Cons
> **Log + exit + restart (correct):** clean recovery, no corrupted state. **Con/anti-pattern:** using these handlers to "keep running" leaves the app in an undefined state and hides bugs. Related: [[00-javascript#P6 Missing .catch]].

### 19. EventEmitter

> [!question] Q19
> What is an `EventEmitter`? How would you build one?

Node's built-in publish/subscribe object: `.on(event, fn)` subscribes, `.emit(event, ...args)` calls all listeners **synchronously** in order, `.once` for one-time. It underpins streams, HTTP servers, and much of Node.

> [!example]
> ```js
> class Emitter {
>   constructor() { this.events = {}; }
>   on(e, fn) { (this.events[e] ||= []).push(fn); return this; }
>   emit(e, ...a) { (this.events[e] || []).forEach(fn => fn(...a)); }
> }
> ```

> [!success] Pros / Cons
> **Pros:** decouples producers from consumers, core to Node's design. **Con/watch-out:** listeners run synchronously (a slow listener blocks); forgetting to remove listeners leaks memory (see [[#20 Debugging a memory leak]]).

### 20. Debugging a memory leak

> [!question] Q20
> How do you debug a memory leak in a Node.js service? (relevant to your low-server-load work)

Reproduce under load and watch heap/RSS grow over time. Take **heap snapshots** (Chrome DevTools via `--inspect`, or `heapdump`) at intervals and **diff** them to find which object types keep growing. Common culprits: unbounded caches/Maps, un-removed listeners, closures holding large scopes, global arrays, and timers.

> [!tip] CV tie-in
> This is the investigation behind achieving **low server load** in your affiliate/ads work — find what grows, bound it (LRU/TTL), remove listeners, null references.

> [!success] Pros / Cons
> **Snapshot-diff approach:** pinpoints the exact leaking type. **Con:** leaks can be slow/intermittent and hard to reproduce; profiling under realistic load takes effort. Related JS cause: [[00-javascript#39 Closure and listener leaks]].

---

## Advanced

### 21. await and the event loop

> [!question] Q21
> Walk through exactly what happens when an `async` function hits `await` in terms of the event loop and microtasks.

An `async` function runs synchronously **until** the first `await`. At `await expr`, it wraps `expr` in `Promise.resolve`, schedules the rest of the function as a **microtask**, and returns control to the caller. When the awaited promise settles, that continuation runs from the microtask queue — after the current operation, before the next timer/macrotask.

> [!success] Pros / Cons
> **Pro:** predictable, non-blocking sequencing. **Con/watch-out:** code after `await` always defers to a microtask even for already-resolved values — a common ordering surprise. Deep dive: [[00-javascript#P16 await desugaring]].

### 22. Scaling a Node API

> [!question] Q22
> How would you scale a Node.js API to handle high throughput (clustering, load balancing, statelessness, caching)? (relevant to your 600+ daily orders / low server load)

Make the app **stateless** (store sessions/state in Redis/DB) so any instance can serve any request. Run **multiple instances** behind a load balancer (cluster module or containers + Nginx/K8s). Add **caching** (Redis), **connection pooling**, and offload heavy/async work to **queues** (RabbitMQ). Optimise hot paths and DB queries/indexes; monitor p95/p99.

> [!tip] CV tie-in
> This mirrors handling **600+ daily affiliate orders** with low load: cache aggressively, keep requests non-blocking, and queue background work. See [[06-system-design#23 Reliable order processing with queues]].

> [!success] Pros / Cons
> **Horizontal + stateless + queues:** near-linear scaling, resilience, absorbs spikes. **Cons:** more moving parts (LB, Redis, queue), eventual consistency for queued work, and infrastructure cost.

### 23. Profiling production Node

> [!question] Q23
> How do you profile and optimize CPU and memory in production Node (`--inspect`, heap snapshots, flame graphs)?

**CPU:** `node --inspect` + Chrome DevTools profiler, or `--prof`/`0x` for flame graphs to find hot functions. **Memory:** heap snapshots and allocation timelines to find leaks/retainers. Use `clinic.js` for end-to-end diagnosis and an APM (Datadog, New Relic) for continuous insight. Always measure → change one thing → re-measure.

> [!success] Pros / Cons
> **Pros:** data-driven optimisation, finds the real bottleneck. **Con:** profiling adds overhead (careful in prod); flame graphs take practice to read.

### 24. Graceful shutdown

> [!question] Q24
> How do you implement graceful shutdown of a Node service (drain connections, close DB, finish in-flight work)?

On `SIGTERM`/`SIGINT`: stop accepting new connections (`server.close()`), finish in-flight requests within a timeout, close DB/Redis/queue connections and flush logs, then exit. Add a hard-timeout fallback to force-exit if draining hangs. In Kubernetes, pair with readiness probes so traffic drains first.

> [!example]
> ```js
> process.on('SIGTERM', async () => {
>   server.close();                 // stop new requests
>   await db.close();               // clean up
>   process.exit(0);
> });
> ```

> [!success] Pros / Cons
> **Pros:** zero dropped requests and no corrupted state during deploys/scaling. **Con:** must handle the hang case (hard timeout) and coordinate with the orchestrator's timing. See [[14-docker-deployment#20 Graceful shutdown]].

### 25. Avoid blocking the loop

> [!question] Q25
> How do you prevent blocking the event loop, and how do you detect it?

Do not run heavy synchronous work (large JSON parse, crypto, big loops, sync `fs`) in the request path. Use async APIs, stream large data, offload CPU work to worker threads/services, and paginate. **Detect** blocking with event-loop lag monitors (`perf_hooks.monitorEventLoopDelay`, `blocked-at`, or APM).

> [!success] Pros / Cons
> **Pros:** responsive server, stable latency. **Con/watch-out:** it is easy to introduce a blocking call accidentally (a big `JSON.parse`); monitoring loop lag is essential to catch it. Root cause: [[#14 CPU-bound tasks]].

### 26. Connection pooling

> [!question] Q26
> Explain connection pooling for databases and why it matters under load.

Opening a DB connection per request is expensive (TCP + auth handshake). A **pool** keeps a set of reusable open connections that requests borrow and return. This bounds the number of connections (protecting the DB), cuts latency, and raises throughput.

> [!success] Pros / Cons
> **Pros:** faster queries, protects the DB from too many connections, higher throughput. **Con/watch-out:** wrong pool size hurts — too big overwhelms the DB, too small becomes a bottleneck; connections can go stale and need health checks. Related: [[05-databases#15 Connection pooling]].

### 27. Redis-backed rate limiter

> [!question] Q27
> How would you design a rate limiter in Node (token bucket) backed by Redis?

Give each key (user/IP) a **bucket** with a capacity and refill rate, stored in Redis. On each request, **atomically** (a Lua script, or `INCR`/`EXPIRE`) compute available tokens from elapsed time; allow and decrement if tokens remain, else reject with `429` + `Retry-After`. Redis centralises the state so the limit holds across all instances.

> [!success] Pros / Cons
> **Redis-backed:** consistent limits across a horizontally-scaled fleet, fast. **Con:** adds a Redis dependency/round trip; must be atomic to avoid race conditions. Design detail: [[06-system-design#12 Design a rate limiter]].

### 28. Worker thread communication

> [!question] Q28
> What are worker threads' communication mechanisms and their limits (structured clone, SharedArrayBuffer)?

Workers talk via **message passing** (`postMessage`/`on('message')`), which **copies** data using the structured clone algorithm — so large payloads are expensive to copy. To share **without** copying, use `SharedArrayBuffer` (with `Atomics` for safe coordination) or **transfer** ownership of an `ArrayBuffer`/`MessagePort` (transfer list) so it moves instead of cloning.

> [!example]
> ```js
> // transfer (move, no copy) a big buffer to a worker
> worker.postMessage(buffer, [buffer]); // buffer is now unusable on this side
> ```

> [!success] Pros / Cons
> **Message passing:** simple, isolated (no shared-memory bugs). Con: copying is slow for big data. **SharedArrayBuffer/transfer:** zero-copy, fast. Con: only raw/transferable data, and shared memory needs `Atomics` to avoid race conditions.

---

## Related notes
- [[00-javascript]] — event loop, promises, closures, memory leaks (browser side).
- [[04-nestjs]] — the framework you build Node backends with.
- [[06-system-design]] — scaling, queues, rate limiting, caching.
- [[14-docker-deployment]] — running Node in containers, graceful shutdown.

## References & Further Study
- [Node.js docs](https://nodejs.org/en/docs) — official API and guides.
- [Node.js: The event loop, timers, and process.nextTick](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick).
- [Node.js: Stream API](https://nodejs.org/api/stream.html) and [Backpressuring in Streams](https://nodejs.org/en/learn/modules/backpressuring-in-streams).
- [clinic.js](https://clinicjs.org/) — production profiling toolkit.
