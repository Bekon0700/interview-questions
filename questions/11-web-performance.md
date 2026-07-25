# Web Performance & Browser Internals — Questions

You claim an 11s → 3s improvement, so expect theory questions behind it.

---

## Beginner

1. What are the Core Web Vitals (LCP, CLS, INP) and what do they measure? (commonly asked)
2. What is the critical rendering path?
3. What is the difference between `defer` and `async` on a script tag?
4. What causes a slow initial page load? List common culprits.
5. What is lazy loading and how do you lazy-load images and components?
6. Why does image optimization matter and how do you do it?
7. What tools do you use to measure frontend performance?

## Intermediate

8. What is the difference between repaint (repaint) and reflow (layout)? What triggers each?
9. What is code splitting and tree shaking? How do they reduce bundle size?
10. What is the difference between prefetch, preload, preconnect, and dns-prefetch?
11. How does the browser render pixels: DOM → CSSOM → render tree → layout → paint → composite?
12. What is render-blocking CSS/JS and how do you eliminate it?
13. How do you optimize web fonts to avoid layout shift and FOUT?
14. What is debouncing/throttling in the context of scroll/resize performance?
15. How do you improve Time to First Byte (TTFB)?

## Advanced

16. Walk me through how you diagnose and fix a page that loads in 11s to get it to 3s. (your CV)
17. How do LCP, CLS, and INP get affected by SSR vs CSR, and how do you optimize each?
18. What is the difference between the main thread and compositor thread? What is jank?
19. How do you reduce and optimize JavaScript execution time on the main thread?
20. How does virtualization/windowing improve rendering of large lists?
21. What is the PRPL pattern and the RAIL performance model?
22. How do service workers and caching strategies improve performance and offline support?
23. How do you set and enforce a performance budget in a team? (your CV: led migration)
24. How does hydration affect performance, and what are partial/progressive hydration and RSC's role?
