---
title: Networking & HTTP — Answers
topic: networking
tags: [interview, fullstack, networking, http, dns]
related: ["[[11-web-performance]]", "[[06-system-design]]"]
---

# Networking & HTTP — Answers

> [!abstract] How to use this note
> Each item has the **question**, a plain-English **explanation**, an **example**, and a **Pros / Cons** box. Numbers match [[questions/10-networking-http|the questions file]].
> [!warning] CV-linked answers
> Items 17 and 22 tie to your WebChat WebSocket work. Items 20 and 23 connect to your Next.js e-commerce performance migration.

---

## Beginner

### 1. What Happens When You Type a URL

> [!question] Q1
> What happens when you type a URL into the browser and hit Enter? (extremely common — go end to end)

This is the classic "show me you understand the full stack" question. Walk through it in order:

1. **Parse the URL** — browser extracts protocol, host, path, query.
2. **DNS lookup** — resolve `example.com` to an IP (browser/OS/router caches first).
3. **TCP connection** — three-way handshake to the server (or CDN edge).
4. **TLS handshake** (HTTPS) — negotiate encryption, verify certificate.
5. **HTTP request** — browser sends `GET /path HTTP/1.1` with headers (`Host`, `User-Agent`, cookies).
6. **Server processing** — load balancer → app server → maybe DB/cache → build response.
7. **HTTP response** — status, headers, body (HTML).
8. **Browser rendering** — parse HTML → DOM, fetch CSS/JS/images, build CSSOM → render tree → layout → paint → composite.
9. **JavaScript** — executes, may fetch more data (SPA/Next.js hydration).

Mention caching (browser, CDN), HTTP/2 multiplexing, and the critical rendering path to show depth.

> [!example]
> `https://shop.com/products/42` → DNS → TCP+TLS → `GET /products/42` → 200 HTML → parse → fetch CSS/JS/images → paint product page → React hydrates → WebSocket connects for live chat.

> [!success] Pros / Cons
> **End-to-end answer:** impresses interviewers at Amazon/Google-style loops. **Con:** easy to ramble — structure with numbered steps and pause for follow-ups (DNS? TLS? rendering?).

> [!info] Further reading
> [MDN: How the Web works](https://developer.mozilla.org/en-US/docs/Learn_web_development/Getting_started/Web_standards/How_the_web_works)

### 2. DNS Resolution

> [!question] Q2
> What is DNS and how does DNS resolution work?

**DNS (Domain Name System)** translates human-readable domain names (`shop.com`) into IP addresses (`93.184.216.34`) so computers can connect.

Resolution flow (simplified):
1. Browser checks its **cache**, then the **OS cache**.
2. Query goes to a **recursive resolver** (usually your ISP or `8.8.8.8`).
3. On cache miss: resolver asks **root servers** → **TLD servers** (`.com`) → **authoritative name server** for the domain.
4. Authoritative server returns the **A/AAAA record** (IPv4/IPv6).
5. Result is **cached** according to its **TTL** (Time To Live).

Common record types: **A/AAAA** (IP), **CNAME** (alias), **MX** (mail), **TXT** (verification/SPF).

> [!example]
> First visit to `api.shop.com`: ~20–120 ms for DNS. Repeat visit within TTL: 0 ms (cached). TTL of 300 s means cache refreshes every 5 minutes.

> [!success] Pros / Cons
> **DNS caching:** fast repeat lookups. **Con:** TTL delays propagation when you change IPs. **CNAME chains:** flexible aliasing. **Con:** extra lookup hops add latency.

> [!info] Further reading
> [Cloudflare Learning: What is DNS?](https://www.cloudflare.com/learning/dns/what-is-dns/)

### 3. HTTP Methods

> [!question] Q3
> What are the main HTTP methods (GET, POST, PUT, PATCH, DELETE) and their semantics?

HTTP methods describe the **intent** of a request on a resource:

| Method | Purpose | Safe | Idempotent |
|--------|---------|------|------------|
| **GET** | Read a resource | Yes | Yes |
| **POST** | Create or submit data | No | No |
| **PUT** | Replace entire resource | No | Yes |
| **PATCH** | Partial update | No | No* |
| **DELETE** | Remove resource | No | Yes |

**Safe** = no side effects (should not change server state). **Idempotent** = repeating the request has the same effect as once. Never mutate data on GET.

> [!example]
> `GET /users/1` — fetch user. `POST /users` — create user. `PUT /users/1` — replace entire user object. `PATCH /users/1` — update email only. `DELETE /users/1` — remove user.

> [!success] Pros / Cons
> **RESTful semantics:** predictable APIs, cache-friendly GETs. **Con:** not all APIs follow them strictly; POST is often used as a catch-all "action" endpoint.

> [!info] Further reading
> [MDN: HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)

### 4. HTTP Status Code Categories

> [!question] Q4
> What are the main HTTP status code categories (1xx–5xx)? Give common examples.

Status codes tell the client what happened:

- **1xx Informational** — `100 Continue`, `103 Early Hints`
- **2xx Success** — `200 OK`, `201 Created`, `204 No Content`
- **3xx Redirection** — `301 Moved Permanently`, `302 Found`, `304 Not Modified`, `307 Temporary Redirect`
- **4xx Client Error** — `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `429 Too Many Requests`
- **5xx Server Error** — `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`, `504 Gateway Timeout`

Key distinctions: **401** = not authenticated; **403** = authenticated but not allowed. **301** = permanent redirect (SEO passes link equity); **302** = temporary.

> [!example]
> Cached asset revalidation: browser sends `If-None-Match: "abc"` → server returns **`304 Not Modified`** (no body, use cache). Login with wrong password → **`401`**. Valid user accessing admin panel → **`403`**.

> [!success] Pros / Cons
> **Precise status codes:** help clients handle errors correctly. **Con:** teams often overuse `200` with error bodies or `500` for everything — loses HTTP semantics.

> [!info] Further reading
> [MDN: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status)

### 5. HTTP vs HTTPS

> [!question] Q5
> What is the difference between HTTP and HTTPS?

**HTTP** sends data in **plaintext** — anyone on the network can read or modify it. **HTTPS** wraps HTTP in **TLS**: encrypted, authenticated (certificate), and tamper-evident.

HTTPS is required for: security (login, cookies), modern browser features (service workers, geolocation), SEO ranking signals, and HTTP/2/3 on most browsers.

> [!example]
> HTTP: `GET /login` body visible as `password=secret123`. HTTPS: same request appears as random encrypted bytes to an observer.

> [!success] Pros / Cons
> **HTTPS:** essential for production, nearly free with Let's Encrypt. **Con:** certificate renewal and slightly more handshake latency (TLS 1.3 minimizes this).

> [!info] Further reading
> [web.dev: Why HTTPS matters](https://web.dev/articles/why-https-matters)

### 6. Request and Response Headers

> [!question] Q6
> What are request/response headers? Name a few important ones.

Headers are **metadata key-value pairs** sent with every HTTP message. They control caching, content type, auth, compression, and security.

Important headers:
- **`Content-Type`** — body format (`application/json`, `text/html`)
- **`Authorization`** — credentials (`Bearer <token>`)
- **`Accept`** — formats the client accepts
- **`Cache-Control`** / **`ETag`** — caching behavior
- **`Set-Cookie`** / **`Cookie`** — session state
- **`Content-Encoding`** — compression (`gzip`, `br`)
- **`Access-Control-Allow-Origin`** — CORS
- **`Strict-Transport-Security`** — force HTTPS (HSTS)
- **`Content-Security-Policy`** — XSS mitigation

> [!example]
> Request: `Accept: application/json`, `Authorization: Bearer eyJ...`. Response: `Content-Type: application/json`, `Cache-Control: max-age=3600`, `ETag: "v1-abc"`.

> [!success] Pros / Cons
> **Headers:** powerful control without changing URL/body. **Con:** easy to misconfigure (wrong `Content-Type`, missing CORS, overly permissive CSP).

> [!info] Further reading
> [MDN: HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)

### 7. TCP vs UDP

> [!question] Q7
> What is the difference between TCP and UDP?

**TCP (Transmission Control Protocol)** is connection-oriented, **reliable**, and **ordered**. It uses a three-way handshake, acknowledges packets, and retransmits lost ones. Used for web (HTTP), email, most APIs.

**UDP (User Datagram Protocol)** is connectionless, **unreliable**, and **unordered** — but lower latency with no handshake overhead. Used for video/voice, gaming, DNS queries. **HTTP/3 runs over QUIC**, which is built on UDP but adds reliability at the application layer.

> [!example]
> Video call: a lost UDP packet = brief glitch (acceptable). Bank transfer: TCP ensures every byte arrives in order.

> [!success] Pros / Cons
> **TCP:** reliable, easy to reason about. **Con:** head-of-line blocking at transport layer (addressed by QUIC/HTTP/3). **UDP:** fast, low overhead. **Con:** app must handle loss/reorder if reliability is needed.

> [!info] Further reading
> [Cloudflare Learning: TCP vs UDP](https://www.cloudflare.com/learning/ddos/glossary/user-datagram-protocol-udp/)

---

## Intermediate

### 8. HTTP/1.1 vs HTTP/2 vs HTTP/3

> [!question] Q8
> What is the difference between HTTP/1.1, HTTP/2, and HTTP/3? (commonly asked)

**HTTP/1.1:**
- One request at a time per connection (head-of-line blocking at app layer).
- **Keep-alive** reuses connections; browsers open **6+ parallel connections** to compensate.
- Text headers (verbose, repeated every request).

**HTTP/2:**
- **Multiplexes** many streams over one TCP connection.
- **Header compression** (HPACK).
- **Server push** (rarely used in practice).
- Still suffers **TCP-level head-of-line blocking** — one lost packet stalls all streams.

**HTTP/3:**
- Runs over **QUIC (UDP)** with independent streams.
- Lost packet affects only its own stream, not others.
- **Faster connection setup** (0-RTT resumption).
- Better on mobile/lossy networks.

> [!example]
> A page with 50 assets: HTTP/1.1 opens 6 connections × multiple round trips. HTTP/2 sends all 50 over 1 connection in parallel. HTTP/3 avoids TCP stall when one packet is lost on a flaky mobile network.

> [!success] Pros / Cons
> **HTTP/2/3:** major latency wins for asset-heavy pages. **Con:** HTTP/3 needs QUIC support on server/CDN; debugging UDP firewalls can be tricky. **HTTP/1.1:** universal fallback.

> [!info] Further reading
> [web.dev: Introduction to HTTP/2](https://web.dev/articles/performance-http2) and [HTTP/3 explained](https://web.dev/articles/http3)

### 9. HTTP Caching Headers

> [!question] Q9
> What are HTTP caching headers (Cache-Control, ETag, Last-Modified, Expires) and how does caching work?

Caching avoids redundant network requests. Key headers:

- **`Cache-Control`** — primary directive: `max-age=3600` (fresh for 1 hour), `no-cache` (must revalidate), `no-store` (never cache), `immutable` (never revalidate — safe for hashed filenames), `public`/`private`.
- **`ETag`** — content fingerprint. Browser sends `If-None-Match: "abc"` → server returns **`304 Not Modified`** if unchanged (no body transfer).
- **`Last-Modified`** / **`If-Modified-Since`** — time-based equivalent of ETag.
- **`Expires`** — legacy absolute expiry date (prefer `Cache-Control`).

Flow: first request → store response + headers. Second request within `max-age` → serve from cache (no network). After expiry → revalidate with ETag → 304 (cheap) or 200 (new content).

> [!example]
> `Cache-Control: public, max-age=31536000, immutable` on `app.a1b2c3.js` — cache for 1 year; filename hash changes on deploy, busting cache safely.

> [!success] Pros / Cons
> **Strong caching:** huge performance win, less server load. **Con:** stale content if TTL too long — use `stale-while-revalidate` or on-demand invalidation for dynamic data.

> [!info] Further reading
> [MDN: HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Caching)

### 10. Safe vs Idempotent Methods

> [!question] Q10
> What is the difference between idempotent and safe methods? Which HTTP methods are which?

**Safe** methods should not change server state — they are read-only. **Idempotent** methods produce the same result if called multiple times.

| Method | Safe | Idempotent |
|--------|------|------------|
| GET, HEAD, OPTIONS | Yes | Yes |
| PUT, DELETE | No | Yes |
| POST | No | No |
| PATCH | No | Usually No |

POST is neither safe nor idempotent — repeating `POST /orders` can create duplicate orders. That is why **idempotency keys** matter for payment/retry logic.

> [!example]
> Retry `DELETE /users/1` three times → user deleted once, then 404. Retry `POST /orders` three times → three orders (bad). Fix: send `Idempotency-Key: uuid` header.

> [!success] Pros / Cons
> **Idempotent retries:** safe to retry on network failure (PUT, DELETE). **Con:** POST retries need explicit idempotency keys — not automatic.

> [!info] Further reading
> [MDN: Idempotency](https://developer.mozilla.org/en-US/docs/Glossary/Idempotent)

### 11. CDN and Caching

> [!question] Q11
> What is a CDN and how does it work with caching?

A **CDN (Content Delivery Network)** is a global network of **edge servers** that cache content close to users. When a user in Dhaka requests an image, they get it from a nearby edge node instead of your origin server in the US.

Flow:
1. User requests `cdn.shop.com/product.jpg`.
2. Edge checks cache — **cache hit** → instant response.
3. **Cache miss** → edge fetches from origin, caches per `Cache-Control`, serves to user.
4. Next user in that region gets a cache hit.

CDNs reduce latency, origin load, and bandwidth costs. They cache static assets, images, and cacheable HTML/API responses.

> [!example]
> Your e-commerce product images served via CloudFront: first request ~200 ms (origin fetch), subsequent requests in Asia ~20 ms (edge hit).

> [!success] Pros / Cons
> **Pros:** lower latency worldwide, DDoS absorption, automatic compression. **Cons:** cache invalidation complexity; dynamic/personalized content may not cache well.

> [!info] Further reading
> [Cloudflare Learning: What is a CDN?](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/)

### 12. TCP Three-Way Handshake and TLS Handshake

> [!question] Q12
> What is the TCP three-way handshake? What is the TLS handshake?

**TCP three-way handshake** establishes a reliable connection:
1. Client → Server: **SYN** (synchronize)
2. Server → Client: **SYN-ACK** (acknowledge + synchronize)
3. Client → Server: **ACK** (acknowledge)

Connection is now open for data transfer.

**TLS handshake** (after TCP, for HTTPS):
1. Client sends supported TLS versions and cipher suites.
2. Server picks cipher, sends **certificate**.
3. Client verifies certificate against trusted CAs.
4. Key exchange establishes a **shared session key**.
5. Encrypted HTTP communication begins.

**TLS 1.3** reduces this to **1 round trip** (sometimes **0-RTT** on resumed connections).

> [!example]
> New HTTPS connection: TCP (~1.5 RTT) + TLS 1.3 (~1 RTT) + HTTP request (~1 RTT) = ~3.5 round trips before first byte. Connection reuse (keep-alive) eliminates repeat handshakes.

> [!success] Pros / Cons
> **TLS 1.3:** faster, more secure defaults. **Con:** 0-RTT resumption has replay-attack considerations for non-idempotent requests. **Keep-alive:** avoids repeated handshakes. **Con:** long-lived connections consume server resources.

> [!info] Further reading
> [Cloudflare Learning: TLS handshake](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)

### 13. REST Principles and Richardson Maturity Model

> [!question] Q13
> Explain REST principles. What makes an API RESTful (Richardson Maturity Model)?

**REST (Representational State Transfer)** principles:
- **Stateless** — each request contains all context; no server session.
- **Resource-based URLs** — nouns, not verbs (`/users/1`, not `/getUser`).
- **Uniform interface** — standard HTTP methods, status codes, representations (JSON).
- **Cacheable** — responses should declare cacheability.
- **Client-server separation** — independent evolution.

**Richardson Maturity Model:**
- **Level 0** — single endpoint, RPC-style over HTTP (`POST /api { action: "getUser" }`).
- **Level 1** — multiple resource URLs.
- **Level 2** — HTTP verbs + status codes (where most "RESTful" APIs sit).
- **Level 3** — **HATEOAS** — responses include hypermedia links to related actions.

> [!example]
> Level 2: `GET /orders/42` → 200 JSON. Level 3 adds `"links": [{ "rel": "cancel", "href": "/orders/42/cancel" }]`.

> [!success] Pros / Cons
> **Level 2 REST:** practical, widely understood, HTTP-cacheable. **Con:** HATEOAS (Level 3) adds complexity few clients use. **GraphQL alternative:** flexible queries but harder to cache via HTTP.

> [!info] Further reading
> [Martin Fowler: Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html)

### 14. REST vs GraphQL

> [!question] Q14
> What is the difference between REST and GraphQL?

**REST** exposes multiple endpoints, each returning a **fixed structure**. Clients may **over-fetch** (get fields they don't need) or **under-fetch** (need multiple round trips).

**GraphQL** exposes **one endpoint** with a typed schema. Clients request **exactly the fields** they need in one query.

Trade-offs:
- **REST:** simpler, HTTP-cacheable, easy to rate-limit per endpoint.
- **GraphQL:** flexible, avoids over/under-fetching; but caching is harder (POST-only), N+1 query risk, and query complexity attacks need cost limits.

> [!example]
> REST: `GET /users/1` returns 20 fields; client needs 2 → over-fetch. GraphQL: `{ user(id: 1) { name email } }` → exactly 2 fields.

> [!success] Pros / Cons
> **GraphQL:** great for mobile/clients with varied data needs. **Con:** server complexity (resolvers, DataLoader, query cost analysis). **REST:** industry default, CDN-friendly. **Con:** multiple endpoints to maintain.

> [!info] Further reading
> [GraphQL official: REST vs GraphQL](https://graphql.org/learn/thinking-in-graphs/)

### 15. Content Negotiation and Accept Header

> [!question] Q15
> What is content negotiation and what does the Accept header do?

**Content negotiation** lets client and server agree on the best **representation** of a resource. The client sends preferences; the server picks the best match.

- **`Accept`** — desired response format (`application/json`, `text/html`, `image/webp`).
- **`Accept-Language`** — preferred language (`en-US`, `bn`).
- **`Accept-Encoding`** — compression support (`gzip, br`).

Server responds with matching **`Content-Type`**, **`Content-Language`**, **`Content-Encoding`**.

> [!example]
> Browser: `Accept: text/html`. API client: `Accept: application/json`. Same URL `/users/1` returns HTML page or JSON depending on client.

> [!success] Pros / Cons
> **Pros:** one URL, multiple representations — clean API design. **Con:** adds server complexity; most modern APIs skip HTML negotiation and use separate routes or subdomains.

> [!info] Further reading
> [MDN: Content negotiation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Content_negotiation)

### 16. Cookies vs Sessions vs Tokens

> [!question] Q16
> What are cookies vs sessions vs tokens for maintaining state over stateless HTTP?

HTTP is **stateless** — each request is independent. Three common patterns to maintain user state:

- **Cookies** — small key-value pairs stored by the browser, auto-sent with requests. Used to carry session IDs or tokens.
- **Server sessions** — session ID in a cookie maps to **server-side state** (in memory or Redis). Easy to revoke; requires shared store at scale.
- **Tokens (JWT)** — self-contained, signed payload. Server validates signature without DB lookup. Scales horizontally; revocation is harder (needs short TTL + refresh strategy).

> [!example]
> Session: cookie `sid=abc123` → server looks up `abc123` in Redis → `{ userId: 1, role: 'admin' }`. JWT: cookie/header `Bearer eyJ...` → server verifies signature → reads `{ sub: 1, role: 'admin' }` from payload.

> [!success] Pros / Cons
> **Server sessions:** instant revocation, small cookie. **Con:** Redis/memory needed at scale. **JWTs:** stateless, fast. **Con:** revocation complexity — see [[09-security#10 JWT Mechanics]].

---

## Advanced

### 17. WebSocket Handshake (WebChat)

> [!question] Q17
> How does the WebSocket handshake work and how does it upgrade from HTTP? (your CV: WebChat)

WebSockets start as a normal **HTTP request** and **upgrade** to a persistent, full-duplex channel:

1. Client sends HTTP GET with special headers:
   - `Upgrade: websocket`
   - `Connection: Upgrade`
   - `Sec-WebSocket-Key: <random base64>`
   - `Sec-WebSocket-Version: 13`
2. Server validates and responds **`101 Switching Protocols`** with:
   - `Sec-WebSocket-Accept: <hash of key + magic GUID>`
3. TCP connection stays open — both sides send **WebSocket frames** (text/binary) without new HTTP requests.

Ideal for **low-latency, bidirectional** communication: chat, live notifications, collaborative editing.

> [!example]
> Your WebChat NestJS gateway: client connects → HTTP upgrade → authenticated WebSocket session → messages flow both ways in real time. Redis pub/sub scales messages across server instances.

> [!tip] CV tie-in
> Describe your WebChat architecture: NestJS WebSocket gateway, Redis pub/sub for multi-instance messaging, RabbitMQ for async bot replies. See [[07-cv-deep-dive#24 WebChat architecture]].

> [!success] Pros / Cons
> **WebSockets:** lowest latency, bidirectional, single connection. **Con:** harder to cache/load-balance than HTTP; need heartbeat/reconnect logic. **SSE/polling:** simpler infra. **Con:** higher latency or one-directional.

> [!info] Further reading
> [MDN: WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)

### 18. Head-of-Line Blocking

> [!question] Q18
> What is head-of-line blocking and how do HTTP/2 and HTTP/3 address it?

**Head-of-line (HOL) blocking** means one slow or lost item blocks everything behind it in the queue.

**HTTP/1.1:** one response at a time per connection — a slow CSS file blocks JS behind it. Fix: open multiple parallel connections (workaround, not a cure).

**HTTP/2:** multiplexes many streams over one TCP connection — app-layer HOL solved. But TCP is a **single byte stream** — one lost packet stalls **all** HTTP/2 streams until retransmitted (**TCP-level HOL blocking**).

**HTTP/3 (QUIC):** each stream is **independent** over UDP. A lost packet on stream A does not block stream B.

> [!example]
> On a mobile network with 2% packet loss: HTTP/2 page load stalls visibly. HTTP/3 continues loading other assets on unaffected streams.

> [!success] Pros / Cons
> **HTTP/3/QUIC:** best for lossy networks and multiplexed apps. **Con:** not yet universal; some corporate firewalls block UDP. **HTTP/2:** good enough on stable networks with CDN.

> [!info] Further reading
> [web.dev: HTTP/3](https://web.dev/articles/http3)

### 19. Keep-Alive and Connection Reuse

> [!question] Q19
> What is keep-alive / connection reuse and why does it matter for performance?

**Keep-alive** (HTTP persistent connections) reuses a single **TCP + TLS** connection for multiple HTTP requests instead of opening a new one each time.

Without keep-alive, every request pays:
- TCP three-way handshake (~1.5 RTT)
- TLS handshake (~1–2 RTT)
- Slow start (TCP congestion window ramps up)

HTTP/1.1 enables keep-alive by default (`Connection: keep-alive`). HTTP/2 and HTTP/3 multiplex many requests over one connection natively.

> [!example]
> A page with 30 assets: without keep-alive = 30 × (TCP + TLS) handshakes. With keep-alive = 1 handshake, 30 requests — saves hundreds of milliseconds.

> [!success] Pros / Cons
> **Pros:** major latency and CPU savings on asset-heavy pages. **Con:** long-lived connections consume server memory; need sensible timeout/idle limits.

> [!info] Further reading
> [MDN: Connection management in HTTP/1.x](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Connection_management_in_HTTP_1.x)

### 20. Browser Cache + CDN + Next.js

> [!question] Q20
> How does browser caching interact with a CDN and a Next.js app (stale-while-revalidate, immutable assets)?

Caching happens in **layers** — each with different rules:

1. **Browser cache** — stores responses per `Cache-Control`. Hashed static assets: `max-age=31536000, immutable` (never revalidate — filename change busts cache).
2. **CDN edge** — caches same headers at PoPs worldwide. Respects `s-maxage`, `stale-while-revalidate`.
3. **Next.js** — **Static files** (`/_next/static/`) get immutable headers automatically. **ISR/Data Cache** on the server. **Router Cache** on the client for navigations.

**`stale-while-revalidate`:** serve stale cached content instantly while fetching fresh content in the background — great for HTML that changes occasionally.

> [!example]
> `product-abc123.js` → browser + CDN cache 1 year (immutable). Product page HTML → CDN `s-maxage=60, stale-while-revalidate=300` → user sees instant page, CDN refreshes in background. Next.js ISR `revalidate: 60` regenerates on the server.

> [!tip] CV tie-in
> Your 11s→3s migration relied heavily on this layered caching plus `next/image` and code splitting. See [[11-web-performance#16 11s → 3s Diagnosis]] and [[02-nextjs]].

> [!success] Pros / Cons
> **Layered caching:** massive speed gains with correct invalidation. **Con:** debugging "why is this stale?" requires checking browser → CDN → Next.js cache → origin — four layers.

> [!info] Further reading
> [Next.js: Caching](https://nextjs.org/docs/app/building-your-application/caching)

### 21. gzip and Brotli Compression

> [!question] Q21
> Explain gzip/brotli compression and its impact on performance.

**gzip** and **brotli** compress text-based responses (HTML, CSS, JS, JSON, SVG) before sending over the network. The browser advertises support via **`Accept-Encoding: gzip, br`**; the server responds with **`Content-Encoding: br`**.

**Brotli** (`br`) generally achieves **15–25% better compression** than gzip for static text, especially at higher quality levels. Pre-compress static assets at build time for best results.

Do **not** compress already-compressed formats (JPEG, PNG, WebP, video) — it wastes CPU with no size benefit.

> [!example]
> 500 KB JS bundle → brotli → ~120 KB over the wire. On 3G, that saves ~1 second of download time alone.

> [!success] Pros / Cons
> **Brotli:** best compression ratio for text. **Con:** slower to compress on the fly — pre-compress at build. **gzip:** faster compression, universal support. **Con:** slightly larger than brotli.

> [!info] Further reading
> [web.dev: Compress images and text](https://web.dev/articles/compress-images)

### 22. Polling vs Long Polling vs SSE vs WebSockets

> [!question] Q22
> What is the difference between polling, long polling, SSE, and WebSockets? When use each?

| Technique | Direction | How it works | Best for |
|-----------|-----------|--------------|----------|
| **Polling** | Client → Server | Client requests every N seconds | Simple, low-frequency updates |
| **Long polling** | Client → Server | Server holds request open until data ready | Near-real-time over plain HTTP |
| **SSE** | Server → Client | One-way HTTP stream (`text/event-stream`) | Live feeds, notifications |
| **WebSockets** | Bidirectional | Upgraded persistent TCP channel | Chat, collaboration, gaming |

> [!example]
> **WebChat (your CV):** WebSockets — agent and customer both send messages instantly. **Order status page:** SSE — server pushes status updates. **Analytics dashboard:** polling every 30 s is fine.

> [!tip] CV tie-in
> WebChat uses WebSockets for real-time chat; Crisp webhooks (HTTP POST) handle external events asynchronously — cite both patterns. See [[07-cv-deep-dive#24 WebChat architecture]].

> [!success] Pros / Cons
> **WebSockets:** lowest latency, bidirectional. **Con:** infra complexity (sticky sessions, reconnect). **SSE:** simpler (plain HTTP), auto-reconnect built in. **Con:** one-direction only, limited browser connections per domain. **Polling:** dead simple. **Con:** wasteful, high latency.

> [!info] Further reading
> [MDN: Server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)

### 23. Debugging a Slow API Request

> [!question] Q23
> How would you debug a slow API request across the network stack?

Use a **layer-by-layer** approach — find where time is spent:

1. **Browser DevTools Network tab** — waterfall shows DNS, connection, TLS, **TTFB**, download time. Large TTFB = server slow. Long download = big payload or no compression.
2. **Check caching** — is the response cacheable? CDN hit or miss?
3. **Server-side** — APM/tracing (OpenTelemetry, Datadog) for handler time, DB query time. Run `explain()` on slow queries.
4. **Payload** — response size, compression enabled? Over-fetching fields?
5. **Client** — blocking JS after response arrives?

Fix the **dominant contributor** first, then re-measure.

> [!example]
> Waterfall: DNS 5 ms, TLS 30 ms, **TTFB 2800 ms**, download 50 ms → problem is server-side. APM shows DB query 2.6 s → add index → TTFB drops to 120 ms.

> [!tip] CV tie-in
> Same methodology you used for the 37 s → 1 s MongoDB query and the 11 s → 3 s page load — measure, find the bottleneck, fix, verify.

> [!success] Pros / Cons
> **Waterfall + APM:** pinpoints exact layer quickly. **Con:** need both client and server visibility — client-only debugging misses backend issues.

> [!info] Further reading
> [web.dev: Optimizing TTFB](https://web.dev/articles/optimize-ttfb)

### 24. CORS Preflight

> [!question] Q24
> What is CORS preflight and when is it triggered?

A **CORS preflight** is an automatic **`OPTIONS`** request the browser sends **before** the real cross-origin request, asking the server: "Am I allowed to make this request?"

Triggered for **"non-simple"** requests:
- Methods other than GET, POST, HEAD (e.g., PUT, DELETE, PATCH).
- Custom headers (e.g., `Authorization`, `X-Custom-Header`).
- `Content-Type` other than `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`.

The server must respond to OPTIONS with `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`. If approved, the browser sends the real request. Simple GET/POST requests skip preflight.

> [!example]
> React app at `https://app.com` sends `DELETE https://api.com/users/1` with `Authorization: Bearer ...` → browser first sends `OPTIONS https://api.com/users/1` → server approves → browser sends `DELETE`.

> [!success] Pros / Cons
> **Preflight:** protects servers from unexpected cross-origin writes. **Con:** adds an extra round trip (~50–200 ms) — minimize custom headers on hot paths where possible.

> [!info] Further reading
> [MDN: Preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request)

---

## Related notes
- [[11-web-performance]] — TTFB, caching, compression, Core Web Vitals.
- [[06-system-design]] — designing APIs, CDNs, and real-time systems at scale.
- [[09-security#3 CORS]] — security side of cross-origin requests.
- [[02-nextjs]] — Next.js caching layers and ISR.
- [[07-cv-deep-dive#24 WebChat architecture]] — WebSocket + webhook patterns from your CV.

## References & Further Study
- [MDN: HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP) — comprehensive HTTP reference.
- [web.dev: Performance](https://web.dev/explore/performance) — caching, compression, HTTP/2/3.
- [Cloudflare Learning Center](https://www.cloudflare.com/learning/) — DNS, CDN, TCP/UDP, TLS explained plainly.
- [High Performance Browser Networking (free book)](https://hpbn.co/) — deep dive into HTTP, TCP, WebSockets.
- [RFC 9110 — HTTP Semantics](https://httpwg.org/specs/rfc9110.html) — the official HTTP spec (for advanced study).
