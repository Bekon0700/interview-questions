# System Design — Questions

Tailored to your CV projects. For each design question, practice: clarify requirements → estimate scale → high-level design → data model → deep dive → bottlenecks/trade-offs.

---

## Beginner / Fundamentals

1. What are the steps you follow in a system design interview? What questions do you ask first?
2. What is horizontal vs vertical scaling? Trade-offs of each?
3. What is a load balancer and what algorithms does it use (round robin, least connections, IP hash)?
4. What is the difference between stateless and stateful services? Why do we prefer stateless for scaling?
5. What is a CDN and when does it help?
6. What is the difference between synchronous (request/response) and asynchronous (message queue) communication?
7. What is a message queue and what problems does it solve? (your CV: RabbitMQ)
8. What is caching and where can you add caches in a system (client, CDN, app, DB)?
9. What is database replication and sharding at a high level?
10. What are the differences between REST, WebSocket, and long polling?

## Intermediate

11. Design a URL shortener (encoding, storage, redirects, scale). (classic warm-up)
12. Design a rate limiter (token bucket vs sliding window, distributed with Redis). (your CV: request queueing)
13. Design an authentication system with JWT access/refresh tokens. How do you handle refresh, rotation, and revocation? (your CV)
14. Design a notification system (email/push, fan-out, retries, dedup).
15. How would you design pagination for a large dataset API (offset vs cursor)?
16. Design a webhook delivery/consumption system with retries and idempotency. (your CV: webhooks)
17. How do you design an idempotent payment/order endpoint?
18. How would you add a caching layer to a read-heavy API and keep it consistent? (your CV: Redis)

## Advanced (mapped to your CV projects)

19. Design an affiliate/referral system: link generation, click tracking, attribution, commission calculation, fraud prevention, at 100k+ affiliates and 600+ daily orders. (your CV: Rokomari Affiliate)
20. Design a real-time chat / customer support system (WebSocket connections, presence, message delivery guarantees, scaling across servers with Redis pub/sub, persistence, bot replies). (your CV: WebChat)
21. Design an ad server: ad selection, budget-based prioritization, pacing, impression/click tracking, and analytics at scale. (your CV: Rokomari Ads)
22. Design the backend for a high-traffic e-commerce site (catalog, cart, checkout, inventory, order processing, handling flash sales). (your CV: Rokomari)
23. How would you design a system to process 600+ daily orders reliably with eventual reporting, using queues? (your CV)
24. How would you scale WebSocket connections to millions of concurrent users?
25. How do you ensure exactly-once / at-least-once processing with a message queue? How do you handle duplicate messages?
26. Design a click/impression tracking + analytics pipeline (high write throughput, aggregation, near-real-time dashboards). (your CV: ads analytics, GA)
27. How would you design commission calculation to be accurate, auditable, and idempotent?
28. How do you handle the dual-write problem (DB + message queue) and ensure consistency (outbox pattern)?
