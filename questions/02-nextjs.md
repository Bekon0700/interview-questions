# Next.js — Questions

Includes migration-specific questions tied directly to your Rokomari Next.js migration.

---

## Beginner

1. What is Next.js and what problems does it solve over plain React (CRA)?
2. What is the difference between the Pages Router and the App Router?
3. What are the different rendering strategies: CSR, SSR, SSG, and ISR? When would you use each? (commonly asked at Vercel, Shopify)
4. How does file-based routing work in Next.js?
5. What is the difference between `Link` and a normal `<a>` tag?
6. How does the `next/image` component optimize images?
7. What are environment variables in Next.js and what does the `NEXT_PUBLIC_` prefix do?
8. What is the difference between `getStaticProps`, `getServerSideProps`, and `getStaticPaths` (Pages Router)?
9. How do you create an API route / route handler in Next.js?

## Intermediate

10. What are Server Components and Client Components in the App Router? When do you add `'use client'`? (commonly asked at Vercel)
11. How does data fetching and caching work in the App Router (`fetch` caching, `revalidate`, `cache` options)?
12. What is ISR (Incremental Static Regeneration) and how does on-demand revalidation work?
13. What is streaming and how do `loading.tsx` and Suspense boundaries enable it?
14. What is Next.js middleware and what are good use cases (auth, redirects, A/B testing, geolocation)?
15. How do layouts, nested layouts, and templates work in the App Router?
16. How does `next/font` optimize fonts and prevent layout shift?
17. How do you handle SEO and metadata in Next.js?
18. What is the difference between `router.push` and `redirect`? How does navigation work client vs server side?
19. How do Route Handlers differ from Server Actions? When would you use Server Actions?
20. What are dynamic routes, catch-all routes, and route groups?

## Advanced

21. Explain the full Next.js caching model in the App Router: Request Memoization, Data Cache, Full Route Cache, and Router Cache. (advanced Vercel-style question)
22. How does hydration work in Next.js and what commonly causes hydration mismatches?
23. How would you reduce a Next.js Docker image from ~2GB to under 200MB? (your CV: 2.1GB → 170MB)
24. How did you reduce page load time from 11s to 3s? Walk through your methodology. (your CV)
25. How do you approach load testing and performance optimization for a production Next.js e-commerce app? (your CV)
26. What is the `output: 'standalone'` build option and why does it matter for deployment?
27. How would you migrate a large legacy jQuery/SSR app to Next.js incrementally without a big-bang rewrite? (your CV: led the Rokomari migration)
28. How do you optimize bundle size in Next.js (dynamic imports, code splitting, analyzing the bundle)?
29. How does Next.js handle SSR + client hydration for SEO on an e-commerce site?
30. How would you implement authentication in the App Router (middleware, cookies, server components)?
31. What are the trade-offs of SSR vs SSG vs ISR specifically for an e-commerce product catalog with frequently changing prices/stock?
32. How do you handle caching and revalidation when product data changes (webhooks / on-demand revalidation)?
