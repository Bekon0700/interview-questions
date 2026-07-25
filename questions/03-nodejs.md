# Node.js — Questions

---

## Beginner

1. What is Node.js and what is it good/bad at? Why is it suitable for I/O-heavy apps?
2. What is `npm` vs `npx`? What is `package.json` vs `package-lock.json`?
3. What is the difference between CommonJS (`require`) and ES Modules (`import`)?
4. What are `process.env`, `process.argv`, and `__dirname`?
5. What is the difference between synchronous and asynchronous APIs in Node (e.g., `fs.readFileSync` vs `fs.readFile`)?
6. What is `package.json`'s `dependencies` vs `devDependencies` vs `peerDependencies`?
7. What is middleware in Express? Give the signature and an example.
8. How do you handle errors in an Express app?

## Intermediate

9. Explain the Node.js event loop phases (timers, pending callbacks, poll, check, close). (commonly asked at Amazon, PayPal)
10. What is the difference between `process.nextTick`, `setImmediate`, and `setTimeout`?
11. What are streams? What are the four types, and why use them? (commonly asked at Netflix)
12. What is backpressure and how do streams handle it?
13. What is the difference between the `cluster` module and `worker_threads`? When use each?
14. How does Node handle CPU-bound tasks and why can they be problematic?
15. What are `Buffer`s and when do you need them?
16. How do you manage environment configuration and secrets in a Node app?
17. What is the difference between `exports` and `module.exports`?
18. How do you handle uncaught exceptions and unhandled promise rejections?
19. What is an `EventEmitter`? How would you build one?
20. How do you debug a memory leak in a Node.js service? (relevant to your low-server-load work)

## Advanced

21. Walk through exactly what happens when an `async` function hits `await` in terms of the event loop and microtasks.
22. How would you scale a Node.js API to handle high throughput (clustering, load balancing, statelessness, caching)? (relevant to your 600+ daily orders / low server load)
23. How do you profile and optimize CPU and memory in production Node (`--inspect`, heap snapshots, flame graphs)?
24. How do you implement graceful shutdown of a Node service (drain connections, close DB, finish in-flight work)?
25. How do you prevent blocking the event loop, and how do you detect it?
26. Explain connection pooling for databases and why it matters under load.
27. How would you design a rate limiter in Node (token bucket) backed by Redis?
28. What are worker threads' communication mechanisms and their limits (structured clone, SharedArrayBuffer)?
