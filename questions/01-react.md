# React — Questions

Focus area for your interview. Practice explaining the "why" behind each, not just the definition.

---

## Beginner

1. What is the virtual DOM and how does React use it?
2. What is JSX and how does it get transformed?
3. What is the difference between props and state?
4. What is the difference between a controlled and an uncontrolled component? (commonly asked at Meta)
5. Why do you need a `key` prop when rendering lists? What happens if you use the array index as a key?
6. What is the difference between a class component and a function component?
7. What are the rules of hooks? Why can't you call hooks conditionally or in loops?
8. What does `useState` return and how do you update state based on the previous state?
9. What is the difference between `useEffect`, `useLayoutEffect`?
10. How do you conditionally render in React? List a few patterns.
11. What is prop drilling and how can you avoid it?
12. What are fragments and why use them?

## Intermediate

13. Explain `useEffect` dependency arrays. What are common mistakes (stale closures, missing/extra dependencies, infinite loops)? (commonly asked at Meta, Amazon)
14. When and why would you use `useMemo` and `useCallback`? What is the cost of overusing them?
15. What is `React.memo` and when does it actually help? When does it not?
16. Explain reconciliation and the diffing algorithm.
17. How does the Context API work? What are its performance pitfalls?
18. What is `useRef` used for beyond DOM access?
19. What is a custom hook? Write one (e.g., `useDebounce` or `useFetch`).
20. What is lifting state up? When would you do it vs colocating state?
21. How do you handle forms in React? Controlled inputs vs libraries like React Hook Form.
22. What is the difference between `useReducer` and `useState`? When prefer `useReducer`?
23. How does batching work in React (especially React 18 automatic batching)?
24. What causes unnecessary re-renders and how do you diagnose them (React DevTools Profiler)?
25. What are error boundaries? Can they catch errors in event handlers or async code?
26. How do you fetch data in React? Compare `useEffect` fetching vs React Query/SWR.

## Advanced

27. Explain the React Fiber architecture. What problem does it solve over the old stack reconciler? (commonly asked at senior/staff level)
28. What are concurrent features in React 18 (`useTransition`, `useDeferredValue`, Suspense)? When would you use them?
29. Explain how Suspense works for data fetching and code splitting.
30. What is the difference between server components and client components? (React 19 / Next.js App Router)
31. How would you optimize a large list of thousands of items (virtualization)?
32. Explain the stale closure problem in depth and multiple ways to fix it.
33. How does `useSyncExternalStore` work and why was it introduced?
34. What are the trade-offs of different state management approaches (Context, Redux, Zustand, Jotai, React Query)? (relevant to your migration project)
35. How do you prevent a Context value from causing all consumers to re-render?
36. Explain how you would architect a large-scale React/Next.js frontend (folder structure, data layer, shared components) — tie this to leading the Rokomari migration.
37. What is hydration and what causes hydration mismatches?
38. How does React handle event delegation under the hood (synthetic events)?
