# Databases (General Concepts) — Questions

General database and caching concepts. MongoDB-specific questions live in `16-mongodb.md`.

---

## Beginner

1. What is the difference between SQL and NoSQL databases? When would you choose each? (commonly asked at Amazon, Microsoft)
2. What are the main types of NoSQL databases (document, key-value, column-family, graph)?
3. What is an index and why does it speed up reads? What is the cost of an index?
4. What is a primary key vs a foreign key?
5. What is normalization and denormalization? Give a trade-off.
6. What is a transaction? What does ACID stand for?
7. What is a database schema? What does schema-less mean for document databases?

## Intermediate

8. Explain the CAP theorem. Where do MongoDB and PostgreSQL sit? (commonly asked at Amazon)
9. What is the difference between ACID and BASE?
10. What are database isolation levels (read uncommitted, read committed, repeatable read, serializable) and the anomalies they prevent?
11. What is optimistic vs pessimistic locking?
12. What is the N+1 query problem and how do you solve it?
13. What is database replication (primary/replica) and what problems does it solve?
14. What is sharding / horizontal partitioning and when do you need it?
15. What is connection pooling and why does it matter?
16. When would you use a relational DB vs a document DB for a specific feature (e.g., orders vs product catalog vs chat messages)?

## Advanced (Caching & Redis)

17. What is caching and what are the main caching strategies (cache-aside, read-through, write-through, write-behind)? (your CV: Redis caching)
18. Explain the cache-aside pattern step by step. How do you handle invalidation?
19. What is a cache stampede / thundering herd and how do you prevent it?
20. What is Redis and what data structures does it provide? Give a use case for each (strings, hashes, lists, sets, sorted sets, streams).
21. How do you use Redis for rate limiting, sessions, leaderboards, and pub/sub?
22. What is the difference between Redis as a cache vs as a primary datastore? How does persistence work (RDB, AOF)?
23. How do you keep a cache consistent with the database? What are common invalidation pitfalls?
24. How would you choose TTLs and eviction policies (LRU, LFU) for a cache?
25. How do you design for high read throughput vs high write throughput?
