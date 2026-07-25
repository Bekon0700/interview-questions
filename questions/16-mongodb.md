# MongoDB — Questions

Dedicated deep dive. This is a major strength on your CV (Affiliate, AdServer, WebChat, 37s → 1s query). Expect drill-down questions.

---

## Beginner

1. What is MongoDB and what is the document model (BSON)? How does it map to relational concepts?
2. What is a collection vs a document vs a field?
3. What is `_id` and what is an ObjectId? What does it encode?
4. What are the basic CRUD operations (`insertOne`, `find`, `updateOne`, `deleteOne`)?
5. What are common query operators (`$eq`, `$gt`, `$in`, `$and`, `$or`, `$regex`)?
6. What is the difference between `find()` and `findOne()`? What does a cursor do?
7. What is projection and why use it?

## Intermediate

8. What types of indexes does MongoDB support (single, compound, multikey, text, geospatial, hashed, TTL, partial, unique)? (commonly asked)
9. What is a compound index and the ESR (Equality, Sort, Range) rule for ordering its fields?
10. How does `explain()` work? What do `COLLSCAN` vs `IXSCAN` mean, and what fields do you look at? (your CV: 37s → 1s optimization)
11. Walk through how you reduced a query from 37 seconds to 1 second. (your CV)
12. What is the aggregation pipeline? Explain common stages (`$match`, `$group`, `$project`, `$sort`, `$lookup`, `$unwind`, `$facet`).
13. When should you embed documents vs reference them? What are the trade-offs?
14. What is `$lookup` and how does it compare to a SQL join? What are its performance implications?
15. What is the difference between `updateOne`, `updateMany`, `replaceOne`, and upserts?
16. What are atomic operators like `$set`, `$inc`, `$push`, `$pull`, `$addToSet`?
17. What are Mongoose schemas, models, middleware (hooks), and virtuals?
18. What does `.lean()` do in Mongoose and when should you use it?

## Advanced

19. What is the MongoDB replica set architecture? Explain primary, secondaries, elections, and oplog.
20. What are read concerns and write concerns? How do they affect consistency and durability?
21. What is read preference (primary, secondary, nearest) and when would you read from secondaries?
22. How does sharding work in MongoDB? How do you choose a shard key, and what makes a bad one?
23. How do transactions work in MongoDB (multi-document, sessions)? What are the limitations?
24. What are change streams and what problems do they solve? (real-time features)
25. What schema design patterns do you know (bucket, outlier, computed, subset, extended reference)?
26. How do you optimize aggregation pipelines (index usage, `$match` early, `allowDiskUse`, `$project` to reduce docs)?
27. How do you avoid the pitfalls of unbounded array growth in a document?
28. How does `populate` differ from `$lookup`, and when would you choose each? (your CV)
29. How would you design the MongoDB schema and indexes for the affiliate reporting queries that need to be fast? (your CV)
30. How do you handle pagination efficiently at scale (skip/limit vs range/keyset pagination)?
