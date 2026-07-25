---
title: Next.js — Answers
topic: nextjs
tags: [interview, fullstack, nextjs, react, ssr]
related: ["[[01-react]]", "[[11-web-performance]]", "[[14-docker-deployment]]", "[[07-cv-deep-dive]]"]
---

# Next.js — Answers

> [!abstract] How to use this note
> Each item has the **question**, a plain-English **explanation**, an **example**, and a **Pros / Cons** box. Numbers match [[questions/02-nextjs|the questions file]].
> [!warning] CV-linked answers
> Items 23-27 are tied to your Rokomari migration (Docker 2.1GB→170MB, page load 11s→3s). They are strong templates — replace specifics with what you actually did.

---

## Beginner

### 1. What Next.js solves

> [!question] Q1
> What is Next.js and what problems does it solve over plain React (CRA)?

Plain React (Create React App) ships a browser-only app: the first HTML is nearly empty until JavaScript loads, which hurts SEO and first paint, and gives you no routing, data-fetching, or image tools. Next.js is a **full-stack React framework** that adds server-side rendering, static generation, file-based routing, API routes, automatic code splitting, and image/font optimization.

> [!success] Pros / Cons
> **Pros:** better SEO and first paint (server rendering), batteries included (routing, data, images), production-ready build. **Cons:** more concepts to learn (rendering modes, caching), a server/build step to operate, and some "magic" that can surprise you (caching defaults).

### 2. Pages vs App Router

> [!question] Q2
> What is the difference between the Pages Router and the App Router?

**Pages Router** (`pages/` folder) is the original model: file-based routes with `getServerSideProps`/`getStaticProps` for data; components are client components. **App Router** (`app/` folder, Next 13+) uses React **Server Components** by default, nested layouts, `loading`/`error` files, streaming, Server Actions, and a new caching model.

> [!success] Pros / Cons
> **App Router (recommended):** smaller bundles (server components), layouts, streaming, modern data model. Con: newer, steeper learning curve, evolving. **Pages Router:** simpler mental model, tons of tutorials. Con: no server components, older data APIs.

### 3. Rendering strategies (CSR/SSR/SSG/ISR)

> [!question] Q3
> What are the different rendering strategies: CSR, SSR, SSG, and ISR? When would you use each? (commonly asked at Vercel, Shopify)

- **CSR** (client render) — the browser builds the page. Good for private dashboards where SEO does not matter.
- **SSR** (server render per request) — fresh HTML each request. Good for personalised or fast-changing content that needs SEO.
- **SSG** (static, at build) — HTML built once, served from CDN. Good for rarely-changing pages (marketing, docs).
- **ISR** (incremental static regeneration) — SSG that refreshes in the background. Good for big catalogs that change occasionally.

> [!example]
> ```jsx
> export const revalidate = 60; // ISR: rebuild this page at most every 60s
> ```

> [!success] Pros / Cons
> **CSR:** interactive, cheap server; con: bad SEO, blank first paint. **SSR:** always fresh + SEO; con: server load, slower TTFB at scale. **SSG:** fastest, cacheable; con: can go stale, needs rebuild. **ISR:** static speed + freshness; con: eventual consistency (a short stale window).

### 4. File-based routing

> [!question] Q4
> How does file-based routing work in Next.js?

Your folder/file structure **is** your URL map. A `page.tsx` inside `app/blog/[slug]/` becomes the route `/blog/:slug`. Special files add behaviour per route segment: `layout`, `loading`, `error`, `route`.

> [!example]
> ```
> app/
>   page.tsx            -> /
>   blog/[slug]/page.tsx -> /blog/:slug
> ```

> [!success] Pros / Cons
> **Pros:** no manual route config, structure mirrors URLs, conventions for loading/error. **Con:** deep nesting can get confusing; special-file conventions must be learned.

### 5. Link vs a

> [!question] Q5
> What is the difference between `Link` and a normal `<a>` tag?

`next/link` does **client-side navigation** — no full page reload, and it **prefetches** the linked route's code/data when the link is in view. A raw `<a>` does a full document reload, losing app state and speed.

> [!example]
> ```jsx
> import Link from 'next/link';
> <Link href="/about">About</Link> // fast SPA navigation + prefetch
> ```

> [!success] Pros / Cons
> **`Link`:** instant navigation, prefetching, preserved state. Con: only for internal routes. **`<a>`:** needed for external links / full reloads. Con: slow, drops SPA benefits for internal routes.

### 6. next/image

> [!question] Q6
> How does the `next/image` component optimize images?

It automatically serves **correctly sized**, modern formats (WebP/AVIF), **lazy-loads** offscreen images, and **reserves space** (width/height or `fill`) to prevent layout shift. This improves the LCP and CLS Core Web Vitals.

> [!example]
> ```jsx
> import Image from 'next/image';
> <Image src="/hero.jpg" width={1200} height={600} alt="Hero" priority />
> ```

> [!success] Pros / Cons
> **Pros:** big performance win (smaller, right-sized, lazy images), no layout shift. **Cons:** needs configuration for external image domains, and the optimization server/step adds some setup. See [[11-web-performance#6 Image optimization]].

### 7. Environment variables

> [!question] Q7
> What are environment variables in Next.js and what does the `NEXT_PUBLIC_` prefix do?

Server env vars are **private** by default. Prefixing with `NEXT_PUBLIC_` **inlines** the value into the browser bundle at build time, exposing it to the client — so never put secrets there.

> [!example]
> ```js
> process.env.DATABASE_URL          // server only (safe for secrets)
> process.env.NEXT_PUBLIC_API_URL   // shipped to the browser (no secrets!)
> ```

> [!success] Pros / Cons
> **Pros:** clear separation of server vs client config. **Con/watch-out:** it is easy to accidentally leak a secret by adding `NEXT_PUBLIC_`; public vars are baked at build time (changing them needs a rebuild). See [[09-security#12 Secrets handling]].

### 8. getStaticProps / getServerSideProps / getStaticPaths

> [!question] Q8
> What is the difference between `getStaticProps`, `getServerSideProps`, and `getStaticPaths` (Pages Router)?

These are the Pages Router data functions. `getStaticProps` runs at **build time** (SSG). `getServerSideProps` runs on **every request** (SSR). `getStaticPaths` lists which dynamic pages to pre-build (with `fallback` controlling on-demand generation).

> [!success] Pros / Cons
> **`getStaticProps`:** fast/cacheable; con: can go stale. **`getServerSideProps`:** always fresh; con: per-request server cost. **Note:** the App Router replaces all three with `async` server components + `fetch` caching ([[#11 Data fetching & caching]]).

### 9. API route / route handler

> [!question] Q9
> How do you create an API route / route handler in Next.js?

Pages Router: a file in `pages/api/` exporting a `(req, res)` handler. App Router: a `route.ts` exporting functions named by HTTP method (`GET`, `POST`, ...) that return a `Response`. Both run server-side — useful for backend logic, webhooks, and proxying.

> [!example]
> ```ts
> // app/api/health/route.ts
> export async function GET() { return Response.json({ ok: true }); }
> ```

> [!success] Pros / Cons
> **Pros:** backend endpoints without a separate server; colocated with the app. **Cons:** for heavy backend logic a dedicated service (like your NestJS APIs) is better; serverless routes have cold starts and time limits.

---

## Intermediate

### 10. Server vs Client Components

> [!question] Q10
> What are Server Components and Client Components in the App Router? When do you add `'use client'`? (commonly asked at Vercel)

In the App Router everything is a **Server Component** by default: it renders on the server, can fetch data directly, and ships **no JavaScript**. Add `'use client'` at the top of a file only when you need interactivity — state, effects, event handlers, or browser APIs.

> [!example]
> ```jsx
> // server component (default): fetch directly
> async function Products() { const list = await db.products(); return <Grid list={list}/>; }
> // client component: interactive
> 'use client';
> function AddToCart() { const [n,setN] = useState(0); /* ... */ }
> ```

> [!success] Pros / Cons
> **Server components:** smaller bundles, direct DB access, better first load. Con: no state/effects/events. **Client components:** interactive. Con: ship JS. **Best practice:** keep client components small and at the leaves; pass server data down as props. See [[01-react#30 Server vs client components]].

### 11. Data fetching & caching

> [!question] Q11
> How does data fetching and caching work in the App Router (`fetch` caching, `revalidate`, `cache` options)?

You can `await fetch()` directly in server components. By default responses are cached (the **Data Cache**). Control freshness with options.

> [!example]
> ```js
> fetch(url);                                   // cached by default
> fetch(url, { cache: 'no-store' });            // always fresh
> fetch(url, { next: { revalidate: 60 } });     // refresh at most every 60s
> fetch(url, { next: { tags: ['product'] } });  // invalidate later with revalidateTag
> ```

> [!success] Pros / Cons
> **Pros:** powerful built-in caching (fast + fewer backend calls), fine-grained control. **Con/watch-out:** the aggressive default caching surprises people ("my data won't update!"); you must choose the right `cache`/`revalidate`/`tags`.

### 12. ISR

> [!question] Q12
> What is ISR (Incremental Static Regeneration) and how does on-demand revalidation work?

ISR lets static pages update **after** deployment without a full rebuild. With `revalidate: N`, the first request after N seconds triggers a background regeneration while users keep getting the cached page (stale-while-revalidate). **On-demand** ISR uses `revalidatePath`/`revalidateTag` (e.g. triggered by a webhook) to refresh specific pages immediately when data changes.

> [!success] Pros / Cons
> **Pros:** static speed + SEO with fresh-enough data, scales to huge catalogs. **Cons:** a short stale window (time-based), and on-demand needs wiring a webhook. See CV tie-in [[#32 Caching/revalidation on data change]].

### 13. Streaming

> [!question] Q13
> What is streaming and how do `loading.tsx` and Suspense boundaries enable it?

Streaming sends the page in **chunks**: the shell (header, layout) appears immediately with a fallback, and slow content streams in when ready. A `loading.tsx` (or a `<Suspense fallback>`) marks the boundary.

> [!example]
> ```jsx
> // app/dashboard/loading.tsx shows instantly while the page's data loads
> export default function Loading() { return <Skeleton />; }
> ```

> [!success] Pros / Cons
> **Pros:** faster perceived load (users see something immediately), better TTFB. **Con:** more boundaries to design; layout shift if skeletons do not match final content.

### 14. Middleware

> [!question] Q14
> What is Next.js middleware and what are good use cases (auth, redirects, A/B testing, geolocation)?

Middleware runs on the **Edge** before a request finishes, for matched routes. Common uses: auth redirects, rewrites, A/B testing, feature flags, geolocation routing, and setting headers/cookies.

> [!example]
> ```ts
> export function middleware(req) {
>   if (!req.cookies.get('session')) return NextResponse.redirect(new URL('/login', req.url));
> }
> export const config = { matcher: ['/dashboard/:path*'] };
> ```

> [!success] Pros / Cons
> **Pros:** fast, global request handling at the edge (close to users). **Cons:** Edge runtime limits (no Node-only APIs), must stay lightweight, and complex logic there is hard to debug.

### 15. Layouts, nested layouts, templates

> [!question] Q15
> How do layouts, nested layouts, and templates work in the App Router?

A `layout.tsx` wraps a segment and all its children and **persists** across navigations (state kept, no re-render). Nested layouts compose down the tree. A `template.tsx` is like a layout but **re-mounts** on each navigation (fresh state) — good for enter animations.

> [!success] Pros / Cons
> **Layouts:** shared UI without re-rendering, preserved state (e.g. a sidebar). Con: persistence can be surprising if you expected a reset. **Templates:** fresh state per navigation. Con: re-mounting costs more.

### 16. next/font

> [!question] Q16
> How does `next/font` optimize fonts and prevent layout shift?

It **self-hosts** fonts (Google or local) at build time, removing external requests, and sizes the fallback font to match the real one so text does not jump when the web font loads (reduces CLS).

> [!success] Pros / Cons
> **Pros:** faster fonts (no external round trip), no layout shift, privacy (no Google request). **Con:** build-time setup; large font families still add bytes (subset them).

### 17. SEO / metadata

> [!question] Q17
> How do you handle SEO and metadata in Next.js?

App Router: export a static `metadata` object or a dynamic `generateMetadata` function per route (title, description, Open Graph, canonical). Combined with SSR/SSG so crawlers get full HTML. Add JSON-LD structured data, sitemaps, and robots config.

> [!example]
> ```ts
> export const metadata = { title: 'Product', description: '...' };
> ```

> [!success] Pros / Cons
> **Pros:** first-class SEO (server-rendered meta, per-route control). **Con:** dynamic metadata means an extra data fetch; you must keep canonical/OG correct to avoid duplicate-content issues.

### 18. router.push vs redirect

> [!question] Q18
> What is the difference between `router.push` and `redirect`? How does navigation work client vs server side?

`router.push` (from `useRouter`, **client**) navigates in response to user actions, SPA-style. `redirect()` (from `next/navigation`, **server**) is called during server rendering/Server Actions to send the user elsewhere **before** the response is sent.

> [!success] Pros / Cons
> **`router.push`:** interactive client navigation, no reload. Con: client-only. **`redirect`:** enforce access/flow on the server before render. Con: happens server-side, not for mid-interaction UI changes.

### 19. Route Handlers vs Server Actions

> [!question] Q19
> How do Route Handlers differ from Server Actions? When would you use Server Actions?

**Route Handlers** (`route.ts`) are HTTP endpoints callable by anything (public APIs, webhooks). **Server Actions** (`'use server'`) are async server functions called **directly** from components/forms without creating an endpoint — ideal for form submissions/mutations with automatic revalidation.

> [!example]
> ```tsx
> 'use server';
> async function createTodo(formData) { await db.insert(...); revalidatePath('/todos'); }
> <form action={createTodo}><input name="text" /></form>
> ```

> [!success] Pros / Cons
> **Server Actions:** less boilerplate for internal mutations, progressive enhancement (work without JS), auto revalidation. Con: newer, not for external callers. **Route Handlers:** standard, callable by anyone (webhooks/mobile). Con: more boilerplate for simple form mutations.

### 20. Dynamic / catch-all / route groups

> [!question] Q20
> What are dynamic routes, catch-all routes, and route groups?

`[id]` = one dynamic segment; `[...slug]` = catch-all (many segments); `[[...slug]]` = optional catch-all; `(group)` = a route group that organises folders **without** changing the URL (useful for applying different layouts).

> [!success] Pros / Cons
> **Pros:** flexible URL modelling, grouping without URL noise. **Con:** the bracket conventions are easy to mix up; catch-all routes need careful handling of the params array.

---

## Advanced

### 21. App Router caching layers

> [!question] Q21
> Explain the full Next.js caching model in the App Router: Request Memoization, Data Cache, Full Route Cache, and Router Cache. (advanced Vercel-style question)

Four layers work together:
- **Request Memoization** — within one render, identical `fetch` calls are deduped.
- **Data Cache** — persistent server cache of `fetch` results across requests (controlled by `revalidate`/`tags`/`cache`).
- **Full Route Cache** — cached rendered HTML/RSC of static routes at build/revalidation.
- **Router Cache** — client-side cache of visited segments for instant back/forward.

> [!success] Pros / Cons
> **Pros:** excellent performance out of the box, fewer backend hits. **Con:** the biggest source of confusion in modern Next.js — "why isn't my data updating?" usually means you must invalidate the right layer (revalidateTag/Path, `cache: 'no-store'`).

> [!info] Further study
> - [Next.js: Caching](https://nextjs.org/docs/app/building-your-application/caching)

### 22. Hydration mismatches

> [!question] Q22
> How does hydration work in Next.js and what commonly causes hydration mismatches?

Hydration is React attaching interactivity to the server-rendered HTML on the client. A **mismatch** happens when the server HTML differs from the client's first render — from `Date`/`Math.random`, reading `window`/`localStorage` during render, locale/timezone diffs, or invalid HTML nesting.

> [!example]
> ```jsx
> // fix: render client-only value after mount
> const [now, setNow] = useState(null);
> useEffect(() => setNow(Date.now()), []);
> ```

> [!success] Pros / Cons
> **Correct hydration:** SSR speed + full interactivity. **Con:** mismatches cause warnings/visual glitches. Fixes: defer to `useEffect`, mounted flag, or `suppressHydrationWarning`. Same concept in [[01-react#37 Hydration and mismatches]].

### 23. Docker image 2.1GB → 170MB

> [!question] Q23
> How would you reduce a Next.js Docker image from ~2GB to under 200MB? (your CV: 2.1GB → 170MB)

Combine several techniques:
- **Multi-stage build** — a `deps` stage installs packages, a `builder` stage runs `next build`, a tiny `runner` stage copies only the output.
- **`output: 'standalone'`** — Next traces and copies only the exact files actually used, not the whole `node_modules`.
- **Alpine base** (`node:20-alpine`) instead of full Debian.
- Copy only `.next/standalone`, `.next/static`, `public`; use `.dockerignore`; copy `package.json` before source for layer caching.

> [!example]
> ```dockerfile
> FROM node:20-alpine AS deps
> WORKDIR /app
> COPY package.json package-lock.json ./
> RUN npm ci
> FROM node:20-alpine AS builder
> WORKDIR /app
> COPY --from=deps /app/node_modules ./node_modules
> COPY . .
> RUN npm run build
> FROM node:20-alpine AS runner
> WORKDIR /app
> ENV NODE_ENV=production
> COPY --from=builder /app/public ./public
> COPY --from=builder /app/.next/standalone ./
> COPY --from=builder /app/.next/static ./.next/static
> EXPOSE 3000
> CMD ["node", "server.js"]
> ```

> [!success] Pros / Cons
> **Pros:** ~12x smaller image → faster deploys/pulls, lower cost, smaller attack surface. **Cons:** multi-stage Dockerfiles are more complex; `standalone` output occasionally needs manual copying of extra files. Full Docker detail: [[14-docker-deployment#15 Next.js Dockerfile walkthrough]].

### 24. Page load 11s → 3s

> [!question] Q24
> How did you reduce page load time from 11s to 3s? Walk through your methodology. (your CV)

**Measure first**, then fix the biggest costs, then re-measure:
1. Profile (Lighthouse, WebPageTest, DevTools, bundle analysis).
2. Reduce JS: dynamic imports/code splitting, drop heavy/duplicate libraries, tree-shaking.
3. Optimize images with `next/image` (usually the biggest LCP win on e-commerce).
4. Improve rendering: move eligible pages to SSG/ISR, stream the rest.
5. Enhance lazy loading of below-the-fold content; cache aggressively; use `next/font`.

> [!tip] CV framing
> Present it as data-driven: "I profiled, prioritised by impact, applied the fix, and validated with before/after metrics." That reasoning impresses more than the number alone.

> [!success] Pros / Cons
> **Pros of this approach:** measurable, defensible gains; avoids guesswork. **Con/watch-out:** be ready for "how did you measure each step?" — know your tools and the single biggest contributor. See [[11-web-performance#16 11s to 3s diagnosis]].

### 25. Load testing & performance

> [!question] Q25
> How do you approach load testing and performance optimization for a production Next.js e-commerce app? (your CV)

Use load-testing tools (k6, Artillery, Locust) to simulate concurrent/peak traffic, measure p95/p99 latency, throughput, and error rate, and find the breaking point. Pair with APM/observability. Then optimise caching, CDN, DB queries/indexes, connection pooling, and horizontal scaling; set performance budgets and run tests in CI before big sales events.

> [!success] Pros / Cons
> **Pros:** confidence the site survives peak load, catches regressions early. **Cons:** realistic load tests take effort to set up; test data/environment must resemble production or results mislead.

### 26. output: 'standalone'

> [!question] Q26
> What is the `output: 'standalone'` build option and why does it matter for deployment?

It makes Next output a **minimal, self-contained** folder with a `server.js` and only the runtime dependencies actually traced as needed — not the full `node_modules`. This is the core of the Docker size reduction.

> [!example]
> ```js
> // next.config.js
> module.exports = { output: 'standalone' };
> ```

> [!success] Pros / Cons
> **Pros:** tiny deployable artifact, simpler containerisation, faster deploys. **Con:** occasionally you must manually copy extra runtime files (e.g. some `public`/config) it did not trace.

### 27. Incremental migration to Next.js

> [!question] Q27
> How would you migrate a large legacy jQuery/SSR app to Next.js incrementally without a big-bang rewrite? (your CV: led the Rokomari migration)

Use the **strangler-fig** pattern: stand up Next.js and route traffic **page-by-page** (via rewrites/proxy), migrating high-value pages first while the legacy app keeps serving the rest. Build a shared component library for visual consistency, validate SEO/analytics/performance parity per slice, preserve URLs and add redirects, then move to critical funnels (product, cart, checkout).

> [!tip] CV tie-in
> Emphasise **leadership**: planning the strategy, coordinating with UI/UX and backend teams, setting performance budgets, and de-risking with flags. See [[07-cv-deep-dive#17 What leading involved]] and [[01-react#36 Architecting a large React/Next.js frontend]].

> [!success] Pros / Cons
> **Pros:** no big-bang risk, continuous delivery, easy rollback per page, early value. **Cons:** two stacks run in parallel for a while (added complexity), shared styling/state between old and new needs care.

### 28. Bundle size optimization

> [!question] Q28
> How do you optimize bundle size in Next.js (dynamic imports, code splitting, analyzing the bundle)?

Measure with `@next/bundle-analyzer`, then: `next/dynamic` to lazy-load heavy/below-the-fold components (`ssr: false` when appropriate), rely on automatic per-route code splitting, import only what you need (avoid barrel imports that break tree-shaking), swap heavy libraries for lighter ones (date-fns over moment), and move logic to server components so it never ships to the client.

> [!success] Pros / Cons
> **Pros:** faster load, better Core Web Vitals, less client CPU. **Con:** over-splitting causes many small requests and loading spinners; `ssr: false` hurts SEO for that content.

### 29. SSR + hydration for SEO

> [!question] Q29
> How does Next.js handle SSR + client hydration for SEO on an e-commerce site?

Product and category pages are server-rendered (SSR or ISR) so crawlers receive **fully populated HTML** (titles, descriptions, prices, structured data) without relying on client JS. React then hydrates for interactivity. Add canonical tags, JSON-LD product schema, and sitemaps.

> [!success] Pros / Cons
> **Pros:** great crawlability and first paint plus a dynamic UX. **Con:** SSR adds server cost; you must keep server and client output consistent to avoid hydration issues ([[#22 Hydration mismatches]]).

### 30. Auth in App Router

> [!question] Q30
> How would you implement authentication in the App Router (middleware, cookies, server components)?

Store a session token in an **httpOnly, Secure, SameSite** cookie. Use **middleware** to check it and redirect unauthenticated users at the edge for protected routes. In server components/Server Actions, read the cookie (`cookies()`) and validate server-side to fetch user-scoped data. Refresh tokens via a route handler.

> [!success] Pros / Cons
> **Pros:** secure (httpOnly resists XSS token theft), checks at the edge, no client trust. **Con:** cookie auth needs CSRF protection ([[09-security#2 CSRF]]); refresh/rotation logic adds complexity. Deep dive: [[06-system-design#13 JWT auth system]].

### 31. SSR vs SSG vs ISR for a product catalog

> [!question] Q31
> What are the trade-offs of SSR vs SSG vs ISR specifically for an e-commerce product catalog with frequently changing prices/stock?

Pure SSG risks stale prices/stock. Pure SSR is always fresh but adds server load/latency at scale. **ISR is usually best**: serve product pages statically for speed/SEO and revalidate on a short interval or on-demand when price/stock changes. For highly volatile data (live stock), render a static shell via ISR and fetch the volatile bit client-side.

> [!success] Pros / Cons
> **SSG:** fastest; con: stale prices (dangerous for commerce). **SSR:** correct; con: cost/latency at scale. **ISR (recommended):** fast + fresh-enough; con: brief stale window — mitigate with on-demand revalidation for critical changes.

### 32. Caching/revalidation on data change

> [!question] Q32
> How do you handle caching and revalidation when product data changes (webhooks / on-demand revalidation)?

**Tag** your fetches (`next: { tags: ['product-123'] }`), and when the product changes in the backend, call `revalidateTag('product-123')` (or `revalidatePath`) from a **webhook** route handler so only the affected pages regenerate.

> [!tip] CV tie-in
> This is the same **webhook-driven** approach behind your WebChat/AdServer integrations — an external change triggers a precise refresh. See [[04-nestjs#21 Reliable webhooks]].

> [!success] Pros / Cons
> **Pros:** static-fast delivery with near-real-time correctness, precise (only affected pages rebuild). **Con:** requires wiring webhooks + tag discipline; a missed tag means stale content lingers.

---

## Related notes
- [[01-react]] — server/client components, hydration, Suspense.
- [[11-web-performance]] — Core Web Vitals, the theory behind 11s→3s.
- [[14-docker-deployment]] — multi-stage builds behind the 2.1GB→170MB win.
- [[07-cv-deep-dive]] — telling the migration story in the interview.

## References & Further Study
- [Next.js docs](https://nextjs.org/docs) — official, covers App Router, rendering, caching.
- [Next.js: Rendering](https://nextjs.org/docs/app/building-your-application/rendering) and [Caching](https://nextjs.org/docs/app/building-your-application/caching).
- [web.dev: Rendering on the Web](https://web.dev/articles/rendering-on-the-web) — CSR/SSR/SSG/ISR trade-offs.
- [Vercel: Deploying Next.js with Docker](https://github.com/vercel/next.js/tree/canary/examples/with-docker) — the standalone Dockerfile pattern.
