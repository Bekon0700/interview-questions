---
title: Web Performance & Browser Internals — Answers
topic: web-performance
tags: [interview, fullstack, performance, core-web-vitals, browser]
related: ["[[02-nextjs]]", "[[01-react]]", "[[07-cv-deep-dive]]"]
---

# Web Performance & Browser Internals — Answers

> [!abstract] How to use this note
> Each item has the **question**, a plain-English **explanation**, an **example**, and a **Pros / Cons** box. Numbers match [[questions/11-web-performance|the questions file]].
> [!warning] CV-linked answers
> Items 16 and 23 are tied to your Rokomari Next.js migration (11s→3s page load, performance budgets). Be ready to explain what you measured and what you changed.

---

## Beginner

### 1. Core Web Vitals (LCP, CLS, INP)

> [!question] Q1
> What are the Core Web Vitals (LCP, CLS, INP) and what do they measure? (commonly asked)

**Core Web Vitals** are Google's key user-experience metrics for real-world page quality:

- **LCP (Largest Contentful Paint)** — **Loading.** Time until the largest visible content element (hero image, heading block) renders. **Good: ≤ 2.5 s.**
- **CLS (Cumulative Layout Shift)** — **Visual stability.** How much visible content shifts unexpectedly during load. **Good: ≤ 0.1.**
- **INP (Interaction to Next Paint)** — **Responsiveness.** Worst latency from user interaction (click, tap, keypress) to the next visual update. **Good: ≤ 200 ms.** (Replaced FID in 2024.)

They measure what **real users** experience in the field (CrUX data), not just lab tests.

> [!example]
> E-commerce product page: LCP = hero product image at 1.8 s (good). CLS = 0.05 because image has width/height set (good). INP = 180 ms after clicking "Add to cart" (good).

> [!success] Pros / Cons
> **Pros:** objective, user-centric metrics; affect SEO ranking. **Cons:** field data lags deployments by ~28 days; lab scores (Lighthouse) can differ from real-user CrUX.

> [!info] Further reading
> [web.dev: Core Web Vitals](https://web.dev/articles/vitals)

### 2. Critical Rendering Path

> [!question] Q2
> What is the critical rendering path?

The **critical rendering path** is the minimum sequence of steps the browser must complete to render the **first pixels** on screen:

1. Parse HTML → build **DOM**
2. Parse CSS → build **CSSOM**
3. Combine DOM + CSSOM → **render tree** (visible nodes only)
4. **Layout** — compute geometry (positions, sizes)
5. **Paint** — fill in pixels
6. **Composite** — assemble layers on GPU

Optimizing it means: minimize render-blocking resources, prioritize above-the-fold content, inline critical CSS, defer non-critical JS/CSS.

> [!example]
> A page with one blocking CSS file (200 KB) and one blocking JS file (500 KB) cannot paint until both download and parse — even if the HTML arrived in 50 ms.

> [!success] Pros / Cons
> **Inlining critical CSS + deferring JS:** faster first paint. **Con:** inline CSS increases HTML size and isn't cacheable separately; must extract "critical" CSS carefully.

> [!info] Further reading
> [web.dev: Critical rendering path](https://web.dev/articles/critical-rendering-path)

### 3. defer vs async on Script Tags

> [!question] Q3
> What is the difference between `defer` and `async` on a script tag?

Both download scripts **without blocking HTML parsing** (unlike a plain `<script>` tag).

- **`async`** — download in parallel, **execute immediately** when ready. Order is **not guaranteed**. Use for independent scripts (analytics, ads).
- **`defer`** — download in parallel, **execute after HTML parsing completes**, in **document order**. Use for scripts that depend on the DOM or on each other.

Plain `<script>` (no attribute) blocks HTML parsing during download and execution.

> [!example]
> ```html
> <script defer src="app.js"></script>      <!-- runs after DOM ready, in order -->
> <script async src="analytics.js"></script> <!-- runs whenever ready, any order -->
> ```

> [!success] Pros / Cons
> **`defer`:** safe for app code, preserves order. **Con:** executes slightly later than `async`. **`async`:** earliest execution. **Con:** can run before DOM is ready; order unpredictable.

> [!info] Further reading
> [MDN: script element](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script)

### 4. Common Causes of Slow Initial Page Load

> [!question] Q4
> What causes a slow initial page load? List common culprits.

The usual suspects, roughly by impact on e-commerce sites:

1. **Large/unoptimized images** — biggest bytes, often the LCP element.
2. **Too much JavaScript** — large bundles, long parse/execute time.
3. **Render-blocking CSS/JS** — delays first paint.
4. **No caching/CDN** — every visit hits origin from far away.
5. **Slow server / high TTFB** — unoptimized DB queries, cold starts.
6. **Too many network requests** — HTTP overhead adds up.
7. **Unoptimized fonts** — extra connections, FOIT/FOUT, CLS.
8. **No compression** — gzip/brotli not enabled.
9. **Third-party scripts** — analytics, chat widgets, ad tags.

> [!example]
> Your pre-migration page: 2 MB of JS, unoptimized hero JPEG (400 KB), no CDN, render-blocking jQuery plugins → 11 s load on 3G. See [[#16 11s → 3s Diagnosis]].

> [!success] Pros / Cons
> **Measuring first (Lighthouse/DevTools):** ranks culprits by actual impact. **Con:** fixing the wrong thing (e.g., minifying CSS when images are 80% of bytes) wastes effort.

> [!info] Further reading
> [web.dev: Fast load times](https://web.dev/explore/fast)

### 5. Lazy Loading

> [!question] Q5
> What is lazy loading and how do you lazy-load images and components?

**Lazy loading** defers loading offscreen or non-critical resources until they are needed (about to enter the viewport or on user action). This reduces initial payload and speeds first paint.

**Images:**
- Native: `<img loading="lazy" src="..." />`
- Next.js: `<Image />` lazy-loads by default (except `priority` images).

**Components:**
- `React.lazy(() => import('./HeavyChart'))` + `<Suspense>`
- Next.js: `dynamic(() => import('./Modal'), { ssr: false })`

> [!example]
> Product page: hero image loads immediately (`priority`). 20 product thumbnails below the fold lazy-load as user scrolls. Checkout modal JS loads only when user clicks "Buy."

> [!success] Pros / Cons
> **Pros:** smaller initial download, faster LCP for above-the-fold content. **Cons:** below-the-fold content may "pop in" on scroll — use placeholders/skeletons for UX.

> [!info] Further reading
> [web.dev: Lazy loading images](https://web.dev/articles/lazy-loading-images)

### 6. Image Optimization

> [!question] Q6
> Why does image optimization matter and how do you do it?

Images are usually the **largest bytes** on a page and often the **LCP element**. Unoptimized images directly hurt load time and Core Web Vitals.

Techniques:
- **Modern formats** — WebP, AVIF (30–50% smaller than JPEG).
- **Correct sizing** — `srcset`/`sizes` or `next/image` to serve right dimensions per device.
- **Compression** — quality 75–85 for photos is usually invisible.
- **Lazy load** offscreen images.
- **Set width/height** (or `aspect-ratio`) to prevent CLS.
- **CDN** for fast global delivery.

> [!example]
> ```jsx
> import Image from 'next/image';
> <Image src="/hero.jpg" width={1200} height={600} alt="Hero" priority />
> ```
> Next.js serves WebP/AVIF, correct size, and reserves space automatically.

> [!tip] CV tie-in
> `next/image` was likely your single biggest LCP win in the 11s→3s migration on an image-heavy e-commerce site.

> [!success] Pros / Cons
> **`next/image`:** automates format, sizing, lazy load, CLS prevention. **Con:** requires config for external domains; optimization step adds build/deploy complexity.

> [!info] Further reading
> [web.dev: Choose the right image format](https://web.dev/articles/choose-the-right-image-format)

### 7. Performance Measurement Tools

> [!question] Q7
> What tools do you use to measure frontend performance?

| Tool | What it measures | Best for |
|------|-----------------|----------|
| **Lighthouse** | Lab audit (CWV, opportunities) | CI checks, quick audits |
| **Chrome DevTools** | Network waterfall, Performance flame chart, Coverage | Deep debugging |
| **WebPageTest** | Multi-location, filmstrip, connection profiles | Before/after comparisons |
| **PageSpeed Insights** | Lab + field (CrUX) data | Real-user vs lab gap |
| **web-vitals library** | Field CWV in your app | Production RUM |
| **`@next/bundle-analyzer`** | Bundle composition | Finding large dependencies |

Always test with **throttling** (Slow 4G, 4× CPU slowdown) to emulate real devices.

> [!example]
> Workflow: Lighthouse baseline → DevTools Network (find largest resources) → Coverage tab (find unused JS) → bundle analyzer (find fat dependencies) → fix → re-run Lighthouse.

> [!success] Pros / Cons
> **Lab tools (Lighthouse):** reproducible, great for CI. **Con:** don't reflect real-user conditions (extensions, device diversity). **Field data (CrUX):** ground truth. **Con:** 28-day rolling window, slow feedback loop.

> [!info] Further reading
> [web.dev: Performance tooling](https://web.dev/articles/vitals-tools)

---

## Intermediate

### 8. Repaint vs Reflow (Layout)

> [!question] Q8
> What is the difference between repaint (repaint) and reflow (layout)? What triggers each?

After initial render, DOM/CSS changes trigger updates:

- **Reflow (layout)** — recalculates element **geometry** (size, position). Triggered by: changing width/height/margin, adding/removing DOM nodes, reading layout properties (`offsetHeight`, `getBoundingClientRect`), font load, viewport resize. **Expensive** — may affect many elements.
- **Repaint** — redraws **pixels** without geometry changes. Triggered by: color, visibility, background, outline changes. Cheaper than reflow, but still costly at scale.

**Composite-only** changes (`transform`, `opacity`) skip layout and paint — handled on the GPU compositor thread. Best for animations.

> [!example]
> `element.style.width = '200px'` → reflow + repaint. `element.style.color = 'red'` → repaint only. `element.style.transform = 'translateX(100px)'` → composite only (fastest).

> [!success] Pros / Cons
> **Composite animations:** smooth 60 fps without layout cost. **Con:** `transform` doesn't affect document flow — use for visual movement, not layout changes.

> [!info] Further reading
> [web.dev: Avoid large, complex layouts and layout thrashing](https://web.dev/articles/avoid-large-complex-layouts-and-layout-thrashing)

### 9. Code Splitting and Tree Shaking

> [!question] Q9
> What is code splitting and tree shaking? How do they reduce bundle size?

**Code splitting** breaks one large JS bundle into **smaller chunks** loaded on demand — per route, per dynamic import. Users download only what they need for the current page.

**Tree shaking** removes **unused exports** (dead code) during the build. It relies on ES modules' static `import`/`export` structure so the bundler can determine what's used.

Together they shrink the initial JS payload and speed up parse/execute time.

> [!example]
> ```js
> // Route-based split (Next.js automatic)
> // app/dashboard/page.tsx → separate chunk from app/settings/page.tsx

> // Dynamic import
> const Chart = dynamic(() => import('./Chart')); // Chart.js loads only on dashboard
> ```
> Tree shaking removes `lodash` functions you never imported.

> [!success] Pros / Cons
> **Pros:** major load-time win on multi-page apps. **Cons:** too many small chunks increase HTTP overhead — balance chunk size (~20–50 KB gzip per chunk is a good target).

> [!info] Further reading
> [web.dev: Reduce JavaScript payloads with code splitting](https://web.dev/articles/reduce-javascript-payloads-with-code-splitting)

### 10. prefetch, preload, preconnect, dns-prefetch

> [!question] Q10
> What is the difference between prefetch, preload, preconnect, and dns-prefetch?

Resource hints tell the browser to prepare connections or fetch resources early:

| Hint | Priority | Purpose |
|------|----------|---------|
| **`dns-prefetch`** | Lowest | Resolve DNS for a domain early |
| **`preconnect`** | High | DNS + TCP + TLS handshake to an origin |
| **`preload`** | High | Fetch a critical resource for **this page** now (font, hero image, critical JS) |
| **`prefetch`** | Low | Fetch a resource likely needed for a **future navigation** |

> [!example]
> ```html
> <link rel="preconnect" href="https://fonts.googleapis.com" />
> <link rel="preload" href="/fonts/inter.woff2" as="font" crossorigin />
> <link rel="prefetch" href="/dashboard.js" /> <!-- user likely navigates here next -->
> ```

> [!success] Pros / Cons
> **`preload`:** eliminates critical-path latency for LCP resources. **Con:** over-preloading wastes bandwidth on resources not used. **`prefetch`:** free performance for predicted navigations. **Con:** wrong predictions waste bandwidth.

> [!info] Further reading
> [web.dev: Preconnect and dns-prefetch](https://web.dev/articles/preconnect-and-dns-prefetch)

### 11. Browser Rendering Pipeline

> [!question] Q11
> How does the browser render pixels: DOM → CSSOM → render tree → layout → paint → composite?

The full rendering pipeline:

1. **Parse HTML** → **DOM tree** (document structure).
2. **Parse CSS** → **CSSOM tree** (style rules).
3. **Combine** DOM + CSSOM → **render tree** (only visible nodes with computed styles).
4. **Layout (reflow)** — calculate exact position and size of every node.
5. **Paint** — fill in pixels for each node into **layers**.
6. **Composite** — GPU assembles layers into the final frame on screen.

JavaScript can modify DOM/CSSOM at any point, triggering partial re-runs of layout → paint → composite. Minimize these re-runs for performance.

> [!example]
> Adding a CSS class that changes `display: none` → removes node from render tree → layout recalculated. Changing `opacity: 0.5` → composite only (no layout).

> [!success] Pros / Cons
> **Understanding the pipeline:** lets you choose the cheapest type of DOM change. **Con:** browser engines optimize internally — micro-optimizing without profiling is premature.

> [!info] Further reading
> [web.dev: Inside look at modern web browser (Part 3)](https://developer.chrome.com/blog/inside-browser-part3)

### 12. Render-Blocking CSS and JS

> [!question] Q12
> What is render-blocking CSS/JS and how do you eliminate it?

**Render-blocking** resources prevent the browser from painting until they are downloaded and processed:

- **CSS** blocks rendering — the browser needs the CSSOM before it can build the render tree.
- **Synchronous JS** blocks HTML parsing — the parser stops until the script downloads and executes.

Elimination strategies:
- **Inline critical CSS** in `<head>`, load rest asynchronously.
- Use **`defer`** or **`async`** on scripts.
- **Code split** — load route-specific JS only.
- Remove unused CSS/JS (PurgeCSS, Coverage tab).
- Move non-critical scripts below the fold or to `requestIdleCallback`.

> [!example]
> Before: 3 render-blocking CSS files (600 KB total) + jQuery (blocking). After: 2 KB critical CSS inlined, rest deferred, jQuery replaced with modern bundle loaded with `defer` → first paint 4 s faster.

> [!success] Pros / Cons
> **Critical CSS extraction:** fast first paint. **Con:** tooling complexity (critical package, build step). **`defer` everything:** simple. **Con:** scripts that must run before paint can't be deferred.

> [!info] Further reading
> [web.dev: Render-blocking resources](https://web.dev/articles/render-blocking-resources)

### 13. Web Font Optimization

> [!question] Q13
> How do you optimize web fonts to avoid layout shift and FOUT?

Web fonts cause two problems: **FOUT** (Flash of Unstyled Text — fallback font shows first) and **CLS** (layout shift when the web font loads with different metrics).

Optimizations:
- **`font-display: swap`** — show fallback immediately, swap when font loads (avoids invisible text/FOIT).
- **`preload`** critical font files.
- **Subset fonts** to only needed characters/glyphs.
- **Self-host** via `next/font` — eliminates extra DNS/TLS connection to Google Fonts.
- **Match fallback metrics** — adjust `size-adjust`, `ascent-override` so fallback and web font occupy similar space (reduces CLS).

> [!example]
> ```js
> // next/font — self-hosted, zero layout shift, no external request
> import { Inter } from 'next/font/google';
> const inter = Inter({ subsets: ['latin'], display: 'swap' });
> ```

> [!success] Pros / Cons
> **`next/font`:** best DX, automatic optimization, no CLS. **Con:** Next.js-specific. **`font-display: swap`:** simple. **Con:** visible text swap (FOUT) unless fallback metrics are tuned.

> [!info] Further reading
> [web.dev: Best practices for fonts](https://web.dev/articles/font-best-practices)

### 14. Debouncing and Throttling

> [!question] Q14
> What is debouncing/throttling in the context of scroll/resize performance?

Scroll, resize, and mousemove events fire **very frequently** (60+ times per second). Running expensive handlers on every event causes **jank** (dropped frames).

- **Throttle** — run the handler at most once every N ms during continuous activity. Good for scroll-position updates, progress bars.
- **Debounce** — run the handler once after activity **stops** for N ms. Good for search-as-you-type, resize-end layout recalculation.

> [!example]
> ```js
> // Throttle: update sticky header at most every 100 ms during scroll
> const onScroll = throttle(() => updateHeader(), 100);

> // Debounce: search API call 300 ms after user stops typing
> const onInput = debounce((q) => fetchResults(q), 300);
> ```

> [!success] Pros / Cons
> **Throttle:** smooth continuous feedback with bounded work. **Con:** may miss the very latest event in the window. **Debounce:** avoids wasted work during rapid input. **Con:** feels slightly delayed for real-time UI.

> [!info] Further reading
> [web.dev: Debounce your input handlers](https://web.dev/articles/debounce-your-input-handlers)

### 15. Improving Time to First Byte (TTFB)

> [!question] Q15
> How do you improve Time to First Byte (TTFB)?

**TTFB** is the time from request start until the first byte of the response arrives. It reflects server processing + network latency — everything **before** the browser receives content.

Improve TTFB by:
- **Optimize server processing** — faster DB queries (indexes), caching hot data in Redis.
- **Use a CDN/edge** — serve from a node closer to the user.
- **Enable keep-alive / HTTP/2** — reduce connection overhead.
- **Cache rendered pages** — SSG/ISR in Next.js serves pre-built HTML instantly.
- **Enable compression** — gzip/brotli (helps total time, though TTFB itself is first byte).
- **Reduce redirects** — each redirect adds a full round trip.

> [!example]
> Product page TTFB: SSR with uncached DB query = 800 ms. Same page with ISR (pre-rendered, CDN-cached) = 45 ms TTFB from edge.

> [!success] Pros / Cons
> **SSG/ISR at the edge:** near-zero TTFB for static content. **Con:** content may be briefly stale. **Server-side caching (Redis):** fresh + fast. **Con:** cache invalidation complexity.

> [!info] Further reading
> [web.dev: Optimize TTFB](https://web.dev/articles/optimize-ttfb)

---

## Advanced

### 16. 11s → 3s Diagnosis (Your CV)

> [!question] Q16
> Walk me through how you diagnose and fix a page that loads in 11s to get it to 3s. (your CV)

Use a **measure → prioritize → fix → verify** loop:

**Step 1 — Measure baseline**
- Lighthouse + DevTools Network waterfall + WebPageTest on throttled 4G.
- Record: LCP element, total bytes, JS execution time, TTFB, number of requests.

**Step 2 — Rank problems by impact**
Typical findings on a legacy e-commerce page:
1. **Images** — unoptimized hero/product images (often 40–60% of bytes).
2. **JavaScript** — large legacy bundles (jQuery + plugins), no code splitting.
3. **Rendering strategy** — everything client-rendered or slow SSR with no caching.
4. **No CDN / weak cache headers** — every asset hits origin.
5. **Render-blocking resources** — synchronous CSS/JS in `<head>`.

**Step 3 — Fix biggest-first**
1. **`next/image`** for all product/hero images (WebP/AVIF, correct sizes, lazy load).
2. **Code splitting** — route-based chunks, dynamic imports for heavy components.
3. **SSG/ISR + streaming** — static shell paints fast; dynamic bits stream in.
4. **Enhanced lazy loading** — below-the-fold components and images deferred.
5. **Aggressive caching** — CDN, immutable hashed assets, ISR revalidation.
6. **`next/font`** — self-hosted fonts, no CLS.

**Step 4 — Verify after each change**
Re-run Lighthouse; attribute gains to specific changes. Present as data-driven decisions, not guesswork.

> [!example]
> Interview answer structure: "Lighthouse showed LCP at 8.2 s — the hero JPEG was 420 KB. After `next/image`, LCP dropped to 2.1 s. Then we split the 1.8 MB JS bundle → 3.8 s total. ISR + CDN took TTFB from 600 ms to 80 ms → final 3 s."

> [!tip] CV tie-in
> This is your strongest measurable win. Pair with [[07-cv-deep-dive#21 Page load 11s → 3s]] and [[02-nextjs]]. Know exact numbers and which change had the biggest impact.

> [!success] Pros / Cons
> **Data-driven approach:** credible, repeatable story. **Con:** without specific numbers interviewers may think it's vague — always cite before/after metrics.

> [!info] Further reading
> [web.dev: Performance audit workflow](https://web.dev/articles/discover-performance-opportunities-with-lighthouse)

### 17. CWV Impact of SSR vs CSR

> [!question] Q17
> How do LCP, CLS, and INP get affected by SSR vs CSR, and how do you optimize each?

| Metric | SSR/SSG | CSR | Optimization |
|--------|---------|-----|--------------|
| **LCP** | Better — content in initial HTML | Worse — blank until JS runs | SSR/SSG/streaming; prioritize LCP image |
| **CLS** | Same risk if no dimensions set | Same risk | Set image dimensions, font metrics, reserve ad space |
| **INP** | Risk from hydration blocking main thread | Often worse — all JS must run first | Less client JS (RSC), defer non-critical hydration |

**SSR/SSG** sends HTML with content already in it — LCP element can render before JS executes. **CSR** shows a blank/skeleton until JS downloads, parses, fetches data, and renders — LCP is delayed.

For INP, both modes suffer if the main thread is busy with hydration or heavy JS. **React Server Components** send zero JS for server-only components, reducing hydration cost.

> [!example]
> CSR product page: LCP at 4.5 s (JS must load + fetch + render). SSR same page: LCP at 1.8 s (HTML includes product image tag immediately).

> [!success] Pros / Cons
> **SSR/SSG:** best LCP and SEO. **Con:** server cost, hydration complexity. **CSR:** simpler hosting. **Con:** poor LCP/SEO unless mitigated with skeletons and code splitting.

> [!info] Further reading
> [web.dev: Rendering on the Web](https://web.dev/articles/rendering-on-the-web)

### 18. Main Thread vs Compositor Thread; Jank

> [!question] Q18
> What is the difference between the main thread and compositor thread? What is jank?

The browser uses multiple threads:

- **Main thread** — runs JavaScript, style calculation, layout, and paint. Only one main thread per tab — long tasks here block everything.
- **Compositor thread** — assembles pre-painted **layers** into frames on the GPU. Can run animations (`transform`, `opacity`) independently of the main thread.

**Jank** = dropped frames — the browser misses the ~16 ms budget for 60 fps. Usually caused by long main-thread tasks (heavy JS, forced layout/reflow). Users perceive this as stuttering or laggy scrolling.

> [!example]
> Animating `left: 100px` (main thread — layout every frame) = jank. Animating `transform: translateX(100px)` (compositor thread) = smooth 60 fps.

> [!success] Pros / Cons
> **Compositor-only animations:** buttery smooth. **Con:** limited to transform/opacity — can't animate layout properties on compositor. **Web Workers:** offload heavy computation. **Con:** no DOM access from workers.

> [!info] Further reading
> [web.dev: Stick to compositor-only properties](https://web.dev/articles/stick-to-compositor-only-properties-and-manage-layer-count)

### 19. Reducing JavaScript Execution Time

> [!question] Q19
> How do you reduce and optimize JavaScript execution time on the main thread?

Strategies ordered by impact:

1. **Ship less JS** — code splitting, tree shaking, replace heavy libraries, React Server Components.
2. **Defer non-critical JS** — `defer`, dynamic imports, `requestIdleCallback`.
3. **Break up long tasks** — chunk work into < 50 ms pieces; yield to the event loop.
4. **Memoize** expensive computations (`useMemo`, `React.memo`).
5. **Virtualize** large lists (react-window, TanStack Virtual).
6. **Web Workers** for heavy computation (parsing, image processing).

Measure with DevTools Performance panel — look for **long tasks** (yellow blocks > 50 ms).

> [!example]
> Rendering 10,000 table rows: 800 ms main-thread block → virtualize to 20 visible rows → 12 ms render. INP drops from 800 ms to 50 ms.

> [!success] Pros / Cons
> **Less JS (RSC, splitting):** root-cause fix. **Con:** architectural change. **Virtualization:** huge win for long lists. **Con:** adds complexity for variable-height items.

> [!info] Further reading
> [web.dev: Optimize long tasks](https://web.dev/articles/optimize-long-tasks)

### 20. Virtualization / Windowing

> [!question] Q20
> How does virtualization/windowing improve rendering of large lists?

**Virtualization** (windowing) renders only the items **currently visible** in the viewport (plus a small overscan buffer), recycling DOM nodes as the user scrolls. Instead of 10,000 DOM nodes, you have ~20.

This keeps DOM count, layout cost, and memory **constant** regardless of list size.

> [!example]
> ```jsx
> import { FixedSizeList } from 'react-window';
> <FixedSizeList height={600} itemCount={10000} itemSize={50} width="100%">
>   {({ index, style }) => <Row index={index} style={style} />}
> </FixedSizeList>
> ```

> [!success] Pros / Cons
> **Pros:** handles thousands of rows smoothly; low memory. **Cons:** variable row heights need extra logic (`VariableSizeList`); accessibility (screen readers) needs careful handling.

> [!info] Further reading
> [web.dev: Virtualize long lists](https://web.dev/articles/virtualize-long-lists-react-window)

### 21. PRPL Pattern and RAIL Model

> [!question] Q21
> What is the PRPL pattern and the RAIL performance model?

**PRPL** — a loading pattern for fast progressive web apps:
- **P**ush/preload critical resources for the initial route.
- **R**ender the initial route as quickly as possible.
- **P**re-cache remaining routes/assets for future navigations.
- **L**azy-load everything else on demand.

**RAIL** — a user-centric performance model with concrete targets:
- **R**esponse — respond to input in **< 100 ms** (user feels instant).
- **A**nimation — produce a frame every **< 16 ms** (60 fps).
- **I**dle — maximize idle time; defer non-critical work to idle periods.
- **L**oad — deliver meaningful content in **< 1 s** (perceived fast); interactive in **< 5 s**.

> [!example]
> PRPL in Next.js: `<link rel="preload">` hero image → SSR first paint → `<link rel="prefetch">` dashboard route → `dynamic()` import for modal.

> [!success] Pros / Cons
> **PRPL/RAIL:** clear framework for prioritizing work. **Con:** targets are guidelines — e-commerce on 3G may need adjusted budgets.

> [!info] Further reading
> [web.dev: RAIL model](https://web.dev/articles/rail) and [PRPL pattern](https://web.dev/articles/apply-instant-loading-with-prpl)

### 22. Service Workers and Caching Strategies

> [!question] Q22
> How do service workers and caching strategies improve performance and offline support?

A **service worker** is a background script that **intercepts network requests**. It enables:

- **Cache-first** — serve from cache, fall back to network (static assets).
- **Network-first** — try network, fall back to cache (API data).
- **Stale-while-revalidate** — serve cache immediately, update cache in background.
- **Offline support** — app works without network for cached routes.

They power **PWAs** — instant repeat visits and resilience on poor networks. Must handle **cache versioning** — bump cache name on deploy to invalidate old assets.

> [!example]
> Repeat visit to a PWA: service worker serves cached shell + CSS/JS instantly (< 200 ms). API call uses network-first with cached fallback if offline.

> [!success] Pros / Cons
> **Pros:** near-instant repeat loads, offline capability. **Cons:** debugging cache issues is hard; stale cache bugs if versioning is wrong; adds complexity for content that must always be fresh.

> [!info] Further reading
> [web.dev: Service worker caching strategies](https://web.dev/articles/service-worker-caching-and-http-caching)

### 23. Performance Budget in a Team

> [!question] Q23
> How do you set and enforce a performance budget in a team? (your CV: led migration)

A **performance budget** is a set of measurable limits the team agrees not to exceed:

**Define budgets:**
- Max JS bundle size (e.g., 200 KB gzip initial).
- Max LCP (2.5 s), CLS (0.1), INP (200 ms).
- Max total page weight (e.g., 1 MB).
- Max number of third-party scripts.

**Enforce in CI:**
- **Lighthouse CI** — fail PR if scores drop below threshold.
- **Bundle size checks** — `@next/bundle-analyzer` + size-limit in GitHub Actions.
- **CrUX/RUM monitoring** — track field metrics over time.

**Team process:**
- Review performance in PRs (like code review).
- Block merges that exceed budget without explicit approval.
- Re-measure after each release.

> [!example]
> As migration lead: set budget "initial JS < 180 KB gzip, LCP < 2.5 s on 4G." Added Lighthouse CI to pipeline — PR adding a 90 KB library failed the check → team found a lighter alternative.

> [!tip] CV tie-in
> Leading the Next.js migration means you owned not just the rewrite but keeping it fast. Mention budgets + CI enforcement as proof of sustainable performance culture.

> [!success] Pros / Cons
> **CI enforcement:** prevents gradual regression ("performance death by a thousand PRs"). **Con:** requires initial setup and team buy-in; budgets need periodic review as features grow.

> [!info] Further reading
> [web.dev: Performance budgets 101](https://web.dev/articles/performance-budgets-101)

### 24. Hydration and Partial/Progressive Hydration

> [!question] Q24
> How does hydration affect performance, and what are partial/progressive hydration and RSC's role?

**Hydration** is when React attaches event handlers and state to server-rendered HTML. It runs on the **main thread** and can block interactivity — hurting **INP**.

Problems with full hydration:
- All component JS must download and execute before the page is interactive.
- Large hydration work = long tasks = jank.

Solutions:
- **Progressive hydration** — hydrate components as they become visible or idle (React 18 `Suspense` boundaries).
- **Partial hydration / islands** — only hydrate interactive "islands"; static HTML stays plain (Astro model).
- **React Server Components (RSC)** — server-only components ship **zero client JS**. Only `"use client"` components hydrate. Dramatically reduces bundle and hydration cost.

> [!example]
> Next.js App Router page: product description = Server Component (0 JS). "Add to cart" button = Client Component (small hydration). vs Pages Router: entire page hydrates including static description.

> [!success] Pros / Cons
> **RSC + selective hydration:** best modern approach — fast LCP + low INP. **Con:** new mental model (`"use client"` boundaries), debugging harder, ecosystem still adapting.

> [!info] Further reading
> [React docs: Server Components](https://react.dev/reference/rsc/server-components) and [web.dev: Hydration strategies](https://web.dev/articles/hydration-and-partial-hydration)

---

## Related notes
- [[02-nextjs]] — rendering modes, `next/image`, caching, App Router performance patterns.
- [[01-react]] — hydration, Server Components, memoization, virtualization libraries.
- [[07-cv-deep-dive#21 Page load 11s → 3s]] — your migration story with STAR structure.
- [[10-networking-http]] — TTFB, caching headers, CDN, compression.
- [[14-docker-deployment]] — smaller Docker images helped deploy the performance wins faster.

## References & Further Study
- [web.dev: Performance](https://web.dev/explore/performance) — the primary learning resource for web performance.
- [web.dev: Core Web Vitals](https://web.dev/articles/vitals) — LCP, CLS, INP definitions and thresholds.
- [Chrome DevTools: Performance features reference](https://developer.chrome.com/docs/devtools/performance/reference) — how to read flame charts and long tasks.
- [MDN: Performance](https://developer.mozilla.org/en-US/docs/Web/Performance) — browser rendering, critical path, resource hints.
- [Next.js: Optimizing](https://nextjs.org/docs/app/building-your-application/optimizing) — official guide for images, fonts, scripts, and caching.
- [High Performance Browser Networking (hpbn.co)](https://hpbn.co/) — free book covering the full stack from TCP to browser rendering.
