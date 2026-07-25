---
title: React — Answers
topic: react
tags: [interview, fullstack, react, frontend]
related: ["[[00-javascript]]", "[[02-nextjs]]", "[[11-web-performance]]"]
---

# React — Answers

> [!abstract] How to use this note
> Each item has the **question**, a plain-English **explanation**, an **example**, and a **Pros / Cons** box. Numbers match [[questions/01-react|the questions file]]. React is a core focus for your interview — practice explaining the "why", not just the definition.

---

## Beginner

### 1. Virtual DOM

> [!question] Q1
> What is the virtual DOM and how does React use it?

The virtual DOM is a lightweight copy of the real page structure, kept in memory as plain JavaScript objects. When your data changes, React builds a new virtual copy, **compares** it with the old one (called "diffing"), and then updates only the small parts of the real page that actually changed.

> [!example]
> ```jsx
> // You describe WHAT the UI should look like:
> function Count({ n }) { return <p>Count: {n}</p>; }
> // React figures out the minimal real-DOM change when n goes 1 -> 2
> ```

> [!success] Pros / Cons
> **Pros:** you write simple "declarative" UI (describe the result, not the steps); React batches and minimises expensive real-DOM edits. **Cons:** the diffing itself costs CPU/memory, so it is not automatically "faster" than hand-tuned direct DOM code — for extreme cases you still optimise (keys, memo, virtualization).

### 2. JSX

> [!question] Q2
> What is JSX and how does it get transformed?

JSX is the HTML-like syntax you write inside JavaScript. It is **not** understood by browsers directly — a compiler (Babel or SWC) turns it into function calls (`React.createElement(...)` or the modern `jsx()` runtime) that produce plain JavaScript objects describing the UI.

> [!example]
> ```jsx
> const el = <h1 className="title">Hi</h1>;
> // compiles to roughly:
> const el2 = React.createElement('h1', { className: 'title' }, 'Hi');
> ```

> [!success] Pros / Cons
> **Pros:** readable, keeps markup and logic together, full JavaScript power inside `{ }`. **Cons:** needs a build step; small gotchas (`className` not `class`, `htmlFor` not `for`, must return one root element or a Fragment).

### 3. Props vs state

> [!question] Q3
> What is the difference between props and state?

**Props** are inputs passed **into** a component by its parent — the child treats them as read-only. **State** is data a component **owns** and can change over time; changing state triggers a re-render.

> [!example]
> ```jsx
> function Child({ label }) {          // label is a prop (read-only)
>   const [open, setOpen] = useState(false); // open is state (owned here)
>   return <button onClick={() => setOpen(!open)}>{label}: {String(open)}</button>;
> }
> ```

> [!success] Pros / Cons
> **Props:** enable reusable, predictable components (data flows down). Con: a child cannot change its own props. **State:** enables interactivity. Con: too much state (or state placed too high) causes extra re-renders — keep it as local as possible ([[#20 Lifting state up]]).

### 4. Controlled vs uncontrolled

> [!question] Q4
> What is the difference between a controlled and an uncontrolled component? (commonly asked at Meta)

In a **controlled** component, React state is the single source of truth for a form input (`value` + `onChange`). In an **uncontrolled** component, the browser/DOM keeps the value and you read it when needed with a `ref`.

> [!example]
> ```jsx
> // controlled
> const [name, setName] = useState('');
> <input value={name} onChange={e => setName(e.target.value)} />
> // uncontrolled
> const ref = useRef();
> <input defaultValue="Ana" ref={ref} /> // read ref.current.value on submit
> ```

> [!success] Pros / Cons
> **Controlled:** instant validation/formatting, easy to read/reset, one source of truth. Con: re-renders on every keystroke. **Uncontrolled:** less code, no per-keystroke render (good for big/simple forms). Con: harder to validate live and keep in sync. Libraries like React Hook Form use uncontrolled inputs for performance (see [[#21 Forms in React]]).

### 5. key prop

> [!question] Q5
> Why do you need a `key` prop when rendering lists? What happens if you use the array index as a key?

A `key` gives each list item a **stable identity** so React can match old and new items during diffing and reuse the right DOM node and state. Using the **array index** as a key breaks this when the list reorders, inserts, or deletes — React then attaches the wrong state/DOM to the wrong item.

> [!example]
> ```jsx
> // good: stable unique id
> {todos.map(t => <Todo key={t.id} todo={t} />)}
> // risky: index key - breaks on reorder/insert/delete
> {todos.map((t, i) => <Todo key={i} todo={t} />)}
> ```

> [!success] Pros / Cons
> **Stable id key (prefer):** correct reuse, preserved input focus/state, efficient updates. **Index key:** only safe for static, never-reordered lists. Con: subtle bugs (inputs showing the wrong value, animation glitches) when the list changes.

### 6. Class vs function components

> [!question] Q6
> What is the difference between a class component and a function component?

**Class** components use `class`, `this.state`, and lifecycle methods (`componentDidMount`, etc.). **Function** components are plain functions that use **hooks** (`useState`, `useEffect`) for the same features. Function components are today's standard.

> [!example]
> ```jsx
> function Timer() {
>   const [s, setS] = useState(0);
>   useEffect(() => { const id = setInterval(() => setS(x => x+1), 1000);
>     return () => clearInterval(id); }, []);
>   return <p>{s}s</p>;
> }
> ```

> [!success] Pros / Cons
> **Function + hooks (prefer):** less boilerplate, no confusing `this`, easy logic reuse via custom hooks. **Class:** still valid and needed for error boundaries ([[#25 Error boundaries]]). Con: verbose, `this` binding issues, harder to share logic.

### 7. Rules of hooks

> [!question] Q7
> What are the rules of hooks? Why can't you call hooks conditionally or in loops?

Two rules: (1) call hooks only at the **top level** of a component/custom hook — never inside `if`, loops, or nested functions; (2) call them only from React functions. React tracks each hook's state by its **call order**, so calling them conditionally would shift the order and mix up which state belongs to which hook.

> [!example]
> ```jsx
> // WRONG - conditional hook changes call order
> if (loggedIn) { const [x] = useState(0); }
> // RIGHT - always call, branch inside
> const [x, setX] = useState(0);
> if (loggedIn) { /* use x */ }
> ```

> [!success] Pros / Cons
> **Following the rules:** stable, predictable state; the `eslint-plugin-react-hooks` linter catches violations. **Con:** you sometimes must restructure code (compute conditionally *after* the hook) rather than skipping the hook.

### 8. useState

> [!question] Q8
> What does `useState` return and how do you update state based on the previous state?

`useState` returns a pair: the current value and a setter function. To update based on the previous value (safe with batching/async), pass a **function** to the setter.

> [!example]
> ```jsx
> const [count, setCount] = useState(0);
> setCount(prev => prev + 1); // functional update - always uses latest value
> // setCount(count + 1) can be stale if called multiple times in one tick
> ```

> [!success] Pros / Cons
> **Functional updater (prefer for derived updates):** avoids stale-value bugs when several updates batch together. **Direct value:** fine for simple sets. Con: using `count + 1` inside async code or multiple calls can use an outdated `count`.

### 9. useEffect vs useLayoutEffect

> [!question] Q9
> What is the difference between `useEffect` and `useLayoutEffect`?

Both run side effects after render. **`useEffect`** runs **after** the browser paints (asynchronous) — best for most effects (data fetching, subscriptions). **`useLayoutEffect`** runs **before** paint (synchronous) — use it when you must measure or change the DOM before the user sees it, to avoid a visible flicker.

> [!example]
> ```jsx
> useLayoutEffect(() => {
>   const { height } = ref.current.getBoundingClientRect();
>   setTooltipTop(height); // measured and applied before paint (no flicker)
> }, []);
> ```

> [!success] Pros / Cons
> **`useEffect`:** does not block painting → smoother UI. **`useLayoutEffect`:** prevents flicker for layout-dependent work. Con: it blocks painting, so heavy work there hurts performance — use only when necessary.

### 10. Conditional rendering

> [!question] Q10
> How do you conditionally render in React? List a few patterns.

Common patterns: logical `&&`, ternary, early `return null`, and lookup maps.

> [!example]
> ```jsx
> {isLoading && <Spinner />}
> {user ? <Profile user={user} /> : <Login />}
> if (!data) return null;
> const views = { list: <List/>, grid: <Grid/> }; return views[mode];
> ```

> [!success] Pros / Cons
> **`&&` / ternary:** concise. **Watch-out:** `{count && <X/>}` renders the number `0` when count is 0 — use `count > 0 && <X/>`. **Lookup maps:** clean for many cases; con: all branches are created unless you guard them.

### 11. Prop drilling

> [!question] Q11
> What is prop drilling and how can you avoid it?

Prop drilling is passing a prop down through many intermediate components that do not use it, just to reach a deep child. Avoid it with the Context API, component composition (passing `children`), or a state library.

> [!example]
> ```jsx
> // instead of threading `theme` through 5 levels:
> const ThemeCtx = createContext('light');
> <ThemeCtx.Provider value="dark"><App /></ThemeCtx.Provider>;
> const theme = useContext(ThemeCtx); // read anywhere below
> ```

> [!success] Pros / Cons
> **Context/composition:** removes noise, decouples middle components. **Con:** Context can cause extra re-renders if misused ([[#35 Preventing context re-renders]]); do not reach for global state for everything.

### 12. Fragments

> [!question] Q12
> What are fragments and why use them?

A Fragment (`<>...</>` or `<React.Fragment>`) lets a component return multiple elements **without** adding an extra wrapper `<div>` to the DOM.

> [!example]
> ```jsx
> return (
>   <>
>     <Header />
>     <Main />
>   </>
> );
> ```

> [!success] Pros / Cons
> **Pros:** cleaner DOM, avoids layout/CSS problems from wrapper divs (flex/grid). **Con:** the short `<>` form cannot take a `key` — use `<React.Fragment key={...}>` inside lists.

---

## Intermediate

### 13. useEffect dependency arrays

> [!question] Q13
> Explain `useEffect` dependency arrays. What are common mistakes (stale closures, missing/extra dependencies, infinite loops)? (commonly asked at Meta, Amazon)

The dependency array tells React **when** to re-run the effect: it re-runs whenever a listed value changes. `[]` = run once on mount; no array = run after every render.

> [!example]
> ```jsx
> useEffect(() => {
>   const id = setTimeout(() => setValue(query), 300);
>   return () => clearTimeout(id);
> }, [query]); // re-runs only when query changes
> ```

> [!success] Pros / Cons
> **Correct deps:** predictable effects, no leaks. **Common bugs/cons:** *missing* deps → stale closures (effect sees old values — see [[#32 Stale closure problem]]); *unstable* deps (new object/function each render) → runs every render; updating state that is in your own deps → **infinite loop**. Use the exhaustive-deps lint rule and `useCallback`/`useMemo` for stable deps.

### 14. useMemo and useCallback

> [!question] Q14
> When and why would you use `useMemo` and `useCallback`? What is the cost of overusing them?

`useMemo` caches a **computed value**; `useCallback` caches a **function reference** between renders. Use them to skip expensive recalculation, or to keep a stable reference so memoised children / effect deps do not change needlessly.

> [!example]
> ```jsx
> const sorted = useMemo(() => bigList.sort(cmp), [bigList]);   // cache value
> const onClick = useCallback(() => doThing(id), [id]);        // stable fn
> ```

> [!success] Pros / Cons
> **Pros:** avoid heavy recompute, prevent needless child re-renders, stable effect deps. **Cons:** they cost memory + a comparison every render; premature use adds noise for no gain. Only reach for them when profiling shows a real problem or a stable reference is required for correctness.

### 15. React.memo

> [!question] Q15
> What is `React.memo` and when does it actually help? When does it not?

`React.memo` wraps a component so it **skips re-rendering** when its props are shallow-equal to last time. It helps for pure components that render often with the **same** props.

> [!example]
> ```jsx
> const Row = React.memo(function Row({ item }) { return <li>{item.name}</li>; });
> ```

> [!success] Pros / Cons
> **Helps when:** the component is expensive and its props rarely change (paired with `useCallback`/`useMemo` on the props). **Does not help (and adds overhead) when:** props change every render (inline objects/functions/`children`), or the component is cheap. Con: false sense of optimisation if props are unstable.

### 16. Reconciliation and diffing

> [!question] Q16
> Explain reconciliation and the diffing algorithm.

Reconciliation is how React decides what changed. It compares the new element tree with the old one using fast heuristics: different element **type** → replace that subtree; same type → keep the node, update changed props; lists → match children by **key**.

> [!example]
> ```jsx
> // type change -> React throws away <p> and builds <div>
> cond ? <p>Hi</p> : <div>Hi</div>;
> ```

> [!success] Pros / Cons
> **Pro:** heuristics make diffing about O(n) instead of the O(n^3) of a full tree comparison. **Con:** the heuristics assume good keys and stable types — bad keys or flip-flopping types cause unnecessary unmount/remount and lost state.

### 17. Context API

> [!question] Q17
> How does the Context API work? What are its performance pitfalls?

Context shares a value (theme, current user, locale) with any descendant via a `Provider` and `useContext`, without prop drilling. The pitfall: **every** consumer re-renders whenever the context value changes, and passing a new object literal as `value` each render triggers all of them.

> [!example]
> ```jsx
> // pitfall: new object every render -> all consumers re-render
> <Ctx.Provider value={{ user, setUser }}>...
> // fix: memoise
> const value = useMemo(() => ({ user, setUser }), [user]);
> <Ctx.Provider value={value}>...
> ```

> [!success] Pros / Cons
> **Pros:** built-in, simple, perfect for low-frequency global values. **Cons:** poor for high-frequency updates (re-render storms). Mitigate by splitting contexts, memoising the value, or using a selector-based store ([[#34 State management trade-offs]]).

### 18. useRef beyond DOM

> [!question] Q18
> What is `useRef` used for beyond DOM access?

`useRef` gives you a stable, mutable box (`.current`) that persists across renders **without** causing a re-render when it changes. Beyond grabbing DOM nodes, use it to store timers, previous values, or the latest callback (to avoid stale closures).

> [!example]
> ```jsx
> const renders = useRef(0);
> renders.current++;          // survives renders, does NOT trigger one
> const latest = useRef(value);
> useEffect(() => { latest.current = value; }); // read latest in a callback
> ```

> [!success] Pros / Cons
> **Pros:** persist data without re-rendering; escape stale closures; hold imperative handles. **Con/watch-out:** changing a ref does **not** update the UI — never store rendered data in a ref expecting it to show.

### 19. Custom hook

> [!question] Q19
> What is a custom hook? Write one (e.g., `useDebounce` or `useFetch`).

A custom hook is a function whose name starts with `use` and which **composes** built-in hooks to reuse stateful logic across components.

> [!example]
> ```jsx
> function useDebounce(value, delay = 300) {
>   const [debounced, setDebounced] = useState(value);
>   useEffect(() => {
>     const id = setTimeout(() => setDebounced(value), delay);
>     return () => clearTimeout(id);
>   }, [value, delay]);
>   return debounced;
> }
> ```

> [!success] Pros / Cons
> **Pros:** DRY, testable, shareable logic; cleaner components. **Con:** each component using the hook gets its **own** state (hooks share logic, not state — use Context/a store to share state).

### 20. Lifting state up

> [!question] Q20
> What is lifting state up? When would you do it vs colocating state?

Lifting state up = moving shared state to the closest common **parent** when multiple components need the same data. Otherwise, **colocate** state as close as possible to where it is used.

> [!example]
> ```jsx
> function Parent() {
>   const [q, setQ] = useState('');       // lifted: both children need q
>   return <><Search q={q} onChange={setQ} /><Results q={q} /></>;
> }
> ```

> [!success] Pros / Cons
> **Lifting:** enables sharing/syncing between siblings. Con: placing state too high re-renders a big subtree. **Colocating:** minimal re-render scope, simpler. **Rule:** lift only as high as truly needed.

### 21. Forms in React

> [!question] Q21
> How do you handle forms in React? Controlled inputs vs libraries like React Hook Form.

Small forms: **controlled inputs** (state per field). Larger/complex forms: a library like **React Hook Form**, which uses uncontrolled inputs + refs to avoid re-rendering on every keystroke and adds validation.

> [!example]
> ```jsx
> // React Hook Form
> const { register, handleSubmit } = useForm();
> <form onSubmit={handleSubmit(save)}>
>   <input {...register('email', { required: true })} />
> </form>
> ```

> [!success] Pros / Cons
> **Controlled:** full control, easy live validation. Con: re-renders per keystroke, boilerplate for many fields. **React Hook Form:** better performance, built-in validation, less code. Con: extra dependency and a different mental model.

### 22. useReducer vs useState

> [!question] Q22
> What is the difference between `useReducer` and `useState`? When prefer `useReducer`?

`useState` is best for simple, independent values. `useReducer` centralises **complex** state transitions in a pure `reducer(state, action)` function — prefer it when state has many sub-values, the next state depends on the previous in non-trivial ways, or many actions update related state.

> [!example]
> ```jsx
> function reducer(state, action) {
>   switch (action.type) {
>     case 'inc': return { count: state.count + 1 };
>     case 'reset': return { count: 0 };
>   }
> }
> const [state, dispatch] = useReducer(reducer, { count: 0 });
> ```

> [!success] Pros / Cons
> **`useReducer`:** predictable, testable transitions; great for complex/related state; `dispatch` is stable. Con: more boilerplate. **`useState`:** simplest for independent primitives. Con: messy when many values change together.

### 23. Batching

> [!question] Q23
> How does batching work in React (especially React 18 automatic batching)?

Batching means React groups multiple state updates into a **single** re-render for performance. React 18 extended this "automatic batching" to updates inside promises, timeouts, and native event handlers (previously only React event handlers were batched).

> [!example]
> ```jsx
> setTimeout(() => {
>   setA(1); setB(2); // React 18: one re-render (batched); React 17: two
> });
> // opt out when you need a synchronous DOM update:
> flushSync(() => setA(1));
> ```

> [!success] Pros / Cons
> **Pros:** fewer renders → faster UI, automatic in React 18. **Con/watch-out:** occasionally you need the DOM updated immediately (measure between updates) — use `flushSync`, sparingly.

### 24. Unnecessary re-renders

> [!question] Q24
> What causes unnecessary re-renders and how do you diagnose them (React DevTools Profiler)?

Causes: a parent re-rendering (children re-render by default), new object/function/array props each render, context value changes, and unstable keys. Diagnose with the **React DevTools Profiler** ("why did this render", highlight updates).

> [!example]
> ```jsx
> // new array each render makes memoised <List> re-render
> <List items={data.filter(x => x.active)} />  // move filter into useMemo
> ```

> [!success] Pros / Cons
> **Fixing (memo, useMemo/useCallback, colocation, context splitting):** measurable performance gains. **Con:** do not optimise blindly — profile first; premature memoisation adds complexity with no benefit.

### 25. Error boundaries

> [!question] Q25
> What are error boundaries? Can they catch errors in event handlers or async code?

An error boundary is a component (currently must be a class, or use a library like `react-error-boundary`) that catches JavaScript errors in its child tree **during render** and shows a fallback UI instead of a blank crash.

> [!example]
> ```jsx
> class Boundary extends React.Component {
>   state = { hasError: false };
>   static getDerivedStateFromError() { return { hasError: true }; }
>   componentDidCatch(err, info) { logError(err, info); }
>   render() { return this.state.hasError ? <Fallback/> : this.props.children; }
> }
> ```

> [!success] Pros / Cons
> **Pros:** isolate failures so one broken widget does not crash the whole app; central error logging. **Cons/limits:** they do **not** catch errors in event handlers, async code (`setTimeout`/promises), SSR, or the boundary itself — handle those with `try/catch`.

### 26. Data fetching

> [!question] Q26
> How do you fetch data in React? Compare `useEffect` fetching vs React Query/SWR.

You can fetch in `useEffect`, but you must hand-roll loading/error/caching/refetch/race handling. Libraries like **React Query** or **SWR** provide caching, deduplication, background refetch, stale-while-revalidate, and cancellation out of the box.

> [!example]
> ```jsx
> // React Query - caching, loading & error handled for you
> const { data, isLoading, error } = useQuery({
>   queryKey: ['user', id], queryFn: () => fetchUser(id),
> });
> ```

> [!success] Pros / Cons
> **`useEffect` fetch:** no dependency, fine for one-off simple loads. Con: you reinvent caching, race handling, retries — easy to get wrong. **React Query/SWR:** robust server-state management, far less code. Con: extra dependency and concepts. For anything beyond trivial, prefer a data library.

---

## Advanced

### 27. Fiber architecture

> [!question] Q27
> Explain the React Fiber architecture. What problem does it solve over the old stack reconciler? (commonly asked at senior/staff level)

Fiber (React 16+) is React's rewritten rendering engine. The **old** reconciler processed the whole tree in one synchronous, uninterruptible pass, so big updates froze the main thread. Fiber breaks work into small **units** that can be **paused, resumed, reprioritised, or aborted**, enabling smooth, interruptible rendering.

> [!example]
> Two phases: **render/reconcile** (interruptible — builds a "work-in-progress" tree) and **commit** (synchronous — applies DOM changes). Urgent updates (typing) can interrupt non-urgent ones (rendering a huge list).

> [!success] Pros / Cons
> **Pros:** foundation for concurrent features ([[#28 Concurrent features]]), responsive UI under heavy work, prioritised updates. **Con:** internal complexity (mostly hidden from you); understanding it matters mainly for senior interviews and advanced debugging.

### 28. Concurrent features (React 18)

> [!question] Q28
> What are concurrent features in React 18 (`useTransition`, `useDeferredValue`, Suspense)? When would you use them?

They let React keep the UI responsive by treating some updates as **low priority**.

- **`useTransition`** — mark a state update as non-urgent (e.g. filtering a huge list) so typing stays smooth; gives an `isPending` flag.
- **`useDeferredValue`** — a value that "lags behind" during heavy renders.
- **`Suspense`** — show a fallback while something (data/code) loads.

> [!example]
> ```jsx
> const [isPending, startTransition] = useTransition();
> startTransition(() => setQuery(input)); // keeps input responsive
> ```

> [!success] Pros / Cons
> **Pros:** smooth typing/interactions even during expensive renders; better perceived performance. **Con:** adds mental overhead; overusing transitions can delay updates unexpectedly. Related: [[11-web-performance#24 Hydration & partial/progressive hydration]].

### 29. Suspense

> [!question] Q29
> Explain how Suspense works for data fetching and code splitting.

`Suspense` wraps children and shows a `fallback` while a child "suspends" (throws a promise while waiting). For **code splitting**, `React.lazy` suspends until a component's JS chunk loads. For **data**, frameworks/libraries integrate so a component suspends until its data resolves (enabling streaming SSR).

> [!example]
> ```jsx
> const Chart = React.lazy(() => import('./Chart'));
> <Suspense fallback={<Spinner/>}><Chart/></Suspense>
> ```

> [!success] Pros / Cons
> **Pros:** declarative loading states, cleaner than manual `isLoading`, enables streaming. **Con:** data-Suspense needs framework/library support (Next.js, React Query with suspense); error handling pairs it with an error boundary.

### 30. Server vs client components

> [!question] Q30
> What is the difference between server components and client components? (React 19 / Next.js App Router)

**Server Components** render on the server, can read the database/filesystem directly, and ship **zero JavaScript** to the browser — but cannot use state, effects, or browser APIs. **Client Components** (`'use client'`) run in the browser and support interactivity.

> [!example]
> ```jsx
> // server component (default in App Router) - fetches directly
> async function Page() { const posts = await db.getPosts(); return <List posts={posts}/>; }
> // client component - interactive
> 'use client'; function Like() { const [n,setN]=useState(0); /* ... */ }
> ```

> [!success] Pros / Cons
> **Server components:** smaller bundles, direct data access, better first load/SEO. Con: no interactivity/state. **Client components:** interactive. Con: ship JS, run on the client. **Pattern:** keep the tree mostly server components; push interactivity to small leaf client components. See [[02-nextjs#10 Server vs Client Components]].

### 31. Large lists (virtualization)

> [!question] Q31
> How would you optimize a large list of thousands of items (virtualization)?

Use **windowing/virtualization** (react-window, TanStack Virtual): render only the items visible in the viewport plus a small buffer, recycling DOM nodes as the user scrolls, so the number of DOM nodes stays constant regardless of list size.

> [!example]
> ```jsx
> import { FixedSizeList } from 'react-window';
> <FixedSizeList height={600} itemCount={10000} itemSize={40} width={400}>
>   {({ index, style }) => <div style={style}>Row {index}</div>}
> </FixedSizeList>
> ```

> [!success] Pros / Cons
> **Pros:** huge memory/render savings, smooth scrolling for massive lists. **Cons:** more complex; variable-height rows need measuring; find-on-page (Ctrl+F) and accessibility need extra care. Combine with stable keys, memoised rows, and pagination.

### 32. Stale closure problem

> [!question] Q32
> Explain the stale closure problem in depth and multiple ways to fix it.

A closure captures the values from the render it was created in. If a callback/effect references state but is not recreated (empty deps, or stored in a ref/timer), it keeps seeing the **old** value.

> [!example]
> ```jsx
> useEffect(() => {
>   const id = setInterval(() => console.log(count), 1000); // always logs the first count
>   return () => clearInterval(id);
> }, []); // fix options below
> ```

> [!success] Pros / Cons
> **Fixes (each with trade-offs):** (1) add the value to deps → recreates the closure (may re-run more often); (2) functional updater `setX(prev => ...)` → no dependency needed; (3) store latest value in a ref and read `ref.current` → avoids re-subscribing but is more manual; (4) `useReducer` → dispatch reads current state. **Con of ignoring it:** silent, confusing bugs. Related JS concept: [[00-javascript#15 Closure]].

### 33. useSyncExternalStore

> [!question] Q33
> How does `useSyncExternalStore` work and why was it introduced?

It is a hook for subscribing to an **external** (non-React) store safely in concurrent rendering. You give it `subscribe`, `getSnapshot`, and optionally `getServerSnapshot`. It prevents **tearing** — different parts of the UI showing inconsistent store values during a concurrent render.

> [!example]
> ```jsx
> const width = useSyncExternalStore(
>   cb => { window.addEventListener('resize', cb); return () => window.removeEventListener('resize', cb); },
>   () => window.innerWidth
> );
> ```

> [!success] Pros / Cons
> **Pros:** consistent snapshots, SSR-safe; libraries (Redux, Zustand) use it internally. **Con:** low-level — most app code uses a state library rather than calling it directly.

### 34. State management trade-offs

> [!question] Q34
> What are the trade-offs of different state management approaches (Context, Redux, Zustand, Jotai, React Query)? (relevant to your migration project)

- **Context** — built-in; good for low-frequency global values; poor for high-frequency updates.
- **Redux (Toolkit)** — predictable, great devtools/middleware for large apps; more boilerplate.
- **Zustand/Jotai** — minimal hook-based stores with selector subscriptions that avoid extra re-renders.
- **React Query/SWR** — for **server** state (caching/sync), not general client state.

> [!tip] CV tie-in
> For the Rokomari migration, separate **server state** (React Query) from **client/UI state** (Context or a light store) — this avoids reinventing caching and keeps global state small.

> [!success] Pros / Cons
> **Context:** zero deps; con: re-render storms. **Redux:** structure + tooling; con: verbose. **Zustand/Jotai:** simple + performant; con: smaller ecosystem. **React Query:** solves server-state brilliantly; con: not for pure UI state. Often you combine React Query + a small client store.

### 35. Preventing context re-renders

> [!question] Q35
> How do you prevent a Context value from causing all consumers to re-render?

Techniques: (1) **memoise** the provider `value` with `useMemo`; (2) **split** into multiple contexts so unrelated updates do not cross-trigger; (3) separate **state** and **dispatch** into two contexts (dispatch is stable); (4) use a **selector-based** store so components subscribe only to the slice they read.

> [!example]
> ```jsx
> const value = useMemo(() => ({ user, settings }), [user, settings]);
> <UserCtx.Provider value={value}>{children}</UserCtx.Provider>
> ```

> [!success] Pros / Cons
> **Pros:** big re-render savings for widely-consumed context. **Con:** more contexts/structure to manage; selector stores add a dependency. Foundation: [[#17 Context API]].

### 36. Architecting a large React/Next.js frontend

> [!question] Q36
> Explain how you would architect a large-scale React/Next.js frontend (folder structure, data layer, shared components) — tie this to leading the Rokomari migration.

Use a **feature-based** folder structure (`features/<domain>/{components,hooks,api,types}`) over type-based, a shared `components/ui` design-system layer, a centralised **data layer** (API client + React Query hooks) so components do not call `fetch` directly, clear server/client component boundaries, and consistent error/loading conventions enforced by lint rules and review.

> [!tip] CV tie-in
> During the Rokomari migration I would migrate incrementally page-by-page (strangler pattern), keep a shared component library for visual consistency, set bundle/performance budgets, and instrument analytics to prove parity. See [[02-nextjs#27 Incremental migration to Next.js]].

> [!success] Pros / Cons
> **Feature-based + central data layer:** scales with team size, easy to find/change a feature, testable. **Con:** more upfront structure and conventions to agree on; requires discipline to keep boundaries clean.

### 37. Hydration and mismatches

> [!question] Q37
> What is hydration and what causes hydration mismatches?

Hydration is React attaching event listeners and reconciling its virtual tree with the server-rendered HTML on the client. A **mismatch** happens when the server HTML differs from the client's first render.

> [!example]
> ```jsx
> // mismatch: server and client render different text
> <p>{new Date().toLocaleTimeString()}</p>          // differs server vs client
> // fix: render client-only value after mount
> const [time, setTime] = useState(null);
> useEffect(() => setTime(new Date().toLocaleTimeString()), []);
> ```

> [!success] Pros / Cons
> **Correct hydration:** fast first paint (SSR) + full interactivity. **Common causes/cons:** `Date`/`Math.random`, reading `window`/`localStorage` during render, locale/timezone diffs, invalid HTML nesting. Fixes: defer to `useEffect`, a mounted flag, or `suppressHydrationWarning` for known-safe cases. See [[02-nextjs#22 Hydration mismatches]].

### 38. Synthetic events

> [!question] Q38
> How does React handle event delegation under the hood (synthetic events)?

React wraps native browser events in a cross-browser `SyntheticEvent`. Instead of attaching a listener to every node, React attaches listeners at the **root container** (since React 17; previously `document`) and uses **delegation** to dispatch events to your handlers.

> [!example]
> ```jsx
> <button onClick={handleClick}>Save</button>
> // React does not attach onClick directly to the button DOM node;
> // it routes the event through its root listener + synthetic event system.
> ```

> [!success] Pros / Cons
> **Pros:** fewer real listeners (memory), consistent behaviour across browsers, easy pooling/normalisation. **Con/watch-out:** mixing React and raw `addEventListener` can behave unexpectedly (order, `stopPropagation` scope). Base concept: [[00-javascript#25 Event delegation]].

---

## Related notes
- [[00-javascript]] — closures, `this`, event loop that underpin React behaviour.
- [[02-nextjs]] — server/client components, hydration, rendering strategies.
- [[11-web-performance]] — re-render cost, virtualization, Core Web Vitals.

## References & Further Study
- [React docs (react.dev)](https://react.dev/) — the official, example-rich reference; start with "Learn React".
- [react.dev: Escape Hatches](https://react.dev/learn/escaping-the-rabbit-hole-with-refs-and-effects) — refs, effects, stale closures.
- [React DevTools Profiler guide](https://react.dev/learn/react-developer-tools) — diagnosing re-renders.
- [TanStack Query (React Query)](https://tanstack.com/query/latest) — server-state management.
- [react-window](https://react-window.vercel.app/) / [TanStack Virtual](https://tanstack.com/virtual/latest) — list virtualization.
