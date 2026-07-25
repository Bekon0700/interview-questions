# Networking & HTTP — Questions

Classic interview opener at Amazon/Google. Ties into your WebChat and Next.js performance work.

---

## Beginner

1. What happens when you type a URL into the browser and hit Enter? (extremely common — go end to end)
2. What is DNS and how does DNS resolution work?
3. What are the main HTTP methods (GET, POST, PUT, PATCH, DELETE) and their semantics?
4. What are the main HTTP status code categories (1xx-5xx)? Give common examples.
5. What is the difference between HTTP and HTTPS?
6. What are request/response headers? Name a few important ones.
7. What is the difference between TCP and UDP?

## Intermediate

8. What is the difference between HTTP/1.1, HTTP/2, and HTTP/3? (commonly asked)
9. What are HTTP caching headers (Cache-Control, ETag, Last-Modified, Expires) and how does caching work?
10. What is the difference between idempotent and safe methods? Which HTTP methods are which?
11. What is a CDN and how does it work with caching?
12. What is the TCP three-way handshake? What is the TLS handshake?
13. Explain REST principles. What makes an API RESTful (Richardson Maturity Model)?
14. What is the difference between REST and GraphQL?
15. What is content negotiation and what does the Accept header do?
16. What are cookies vs sessions vs tokens for maintaining state over stateless HTTP?

## Advanced

17. How does the WebSocket handshake work and how does it upgrade from HTTP? (your CV: WebChat)
18. What is head-of-line blocking and how do HTTP/2 and HTTP/3 address it?
19. What is keep-alive / connection reuse and why does it matter for performance?
20. How does browser caching interact with a CDN and a Next.js app (stale-while-revalidate, immutable assets)?
21. Explain gzip/brotli compression and its impact on performance.
22. What is the difference between polling, long polling, SSE, and WebSockets? When use each?
23. How would you debug a slow API request across the network stack?
24. What is CORS preflight and when is it triggered?
