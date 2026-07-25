# Web Security — Questions

Your CV mentions JWT auth, so security follow-ups are very likely.

---

## Beginner

1. What is XSS (Cross-Site Scripting)? What are the types and how do you prevent it? (commonly asked)
2. What is CSRF (Cross-Site Request Forgery) and how do you prevent it?
3. What is CORS? What problem does it solve, and how do you configure it correctly?
4. What is SQL injection / NoSQL injection and how do you prevent it?
5. Why should you never store passwords in plain text? How should you store them?
6. What is HTTPS and why is it important? What does TLS do?
7. What is the difference between authentication and authorization?

## Intermediate

8. Where should you store a JWT on the client — localStorage vs cookies? What are the trade-offs? (your CV: JWT)
9. What are httpOnly, Secure, and SameSite cookie attributes?
10. How do JWTs work, and what are their security pitfalls (algorithm confusion, no revocation, long expiry)?
11. What is the principle of least privilege and why does it matter?
12. How do you securely handle secrets and API keys in an app?
13. What is rate limiting and how does it protect against brute force and abuse?
14. How do you prevent sensitive data exposure in APIs (over-fetching, error leakage)?
15. What is clickjacking and how do you prevent it (X-Frame-Options, CSP frame-ancestors)?

## Advanced

16. Walk through the OWASP Top 10 at a high level. (commonly asked)
17. What is a Content Security Policy (CSP) and how does it mitigate XSS?
18. How do you design a secure authentication + session system end to end?
19. How do you prevent injection in a MongoDB/Mongoose app specifically? (your CV: MongoDB)
20. How do you handle refresh token theft and detect token reuse? (your CV: token invalidation)
21. What are common webhook security concerns and how do you secure them? (your CV: webhooks)
22. How do you protect against IDOR (Insecure Direct Object Reference)?
23. How would you secure a file upload feature?
