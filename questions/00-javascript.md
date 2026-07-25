# JavaScript & TypeScript — Questions

Core JavaScript is the most heavily tested area at top companies for full stack roles. Practice answering these out loud before checking the answers file.

---

## Beginner

1. What is the difference between `var`, `let`, and `const`? (commonly asked at Amazon, Microsoft)
2. What is the difference between `==` and `===`?
3. What are the primitive types in JavaScript? How do they differ from objects (reference types)?
4. What does `typeof null` return, and why is it considered a bug?
5. What is hoisting? How does it apply to `var`, `let`, `const`, and function declarations?
6. What is the difference between `null` and `undefined`?
7. What is the difference between function declarations and function expressions?
8. What are truthy and falsy values? List all falsy values in JavaScript.
9. What is the difference between `slice`, `splice`, and `split`?
10. What does `this` refer to in the global scope, inside a regular function, and inside an arrow function?
11. What is the difference between `map`, `forEach`, `filter`, and `reduce`?
12. What is the spread operator and the rest parameter? Give an example of each.
13. What is destructuring? Show object and array destructuring.
14. What is the difference between shallow copy and deep copy?

## Intermediate

15. What is a closure? Give a practical use case. (commonly asked at Meta, Amazon, Google)
16. Explain the event loop. What is the difference between the call stack, task queue (macrotask), and microtask queue? (commonly asked at Meta, Netflix)
17. What will be the output order of `console.log` with `setTimeout`, `Promise.then`, and synchronous code mixed together? Explain why.
18. What is the difference between `Promise`, `async/await`, and callbacks? How do you handle errors in each?
19. What is the difference between `Promise.all`, `Promise.allSettled`, `Promise.race`, and `Promise.any`?
20. How does prototypal inheritance work? What is the prototype chain?
21. What is the difference between `call`, `apply`, and `bind`?
22. Explain `debounce` and `throttle`. When would you use each? (commonly asked at Uber, Meta)
23. What is currying? Why is it useful?
24. What is the difference between arrow functions and regular functions (beyond syntax)?
25. What is event delegation and why is it useful?
26. Explain the difference between deep equality and reference equality.
27. What is the temporal dead zone (TDZ)?
28. What are `Map` and `Set`? How do they differ from plain objects and arrays?
29. What is the difference between synchronous and asynchronous code? What is the single-threaded nature of JavaScript?
30. How does garbage collection work in JavaScript? What causes memory leaks?

## Advanced

31. Implement `debounce` from scratch. Then add a `leading`/`trailing` option. (commonly asked at Uber, Stripe)
32. Implement a deep clone function. What edge cases must you handle (circular references, `Date`, `Map`, `Set`)?
33. Implement your own `Promise.all`.
34. Implement a promise-based concurrency limiter (promise pool) that runs at most N tasks at a time. (directly relevant to your affiliate request-queueing work)
35. Explain the microtask vs macrotask ordering with `async/await` desugaring. What does an `await` actually compile to?
36. What is a generator function? How can generators be used for async control flow?
37. Explain `WeakMap` and `WeakSet`. When would you use them over `Map`/`Set`?
38. How does `this` binding work with `new`? Explain the four rules of `this` binding precedence.
39. What is a memory leak in a closure or event listener, and how do you prevent it?
40. Implement `memoize` with a custom cache key resolver.

## TypeScript essentials

41. What is the difference between `interface` and `type`? When would you use each?
42. What are generics? Write a generic function and a generic constraint (`extends`).
43. Explain the utility types `Partial`, `Pick`, `Omit`, `Record`, `Readonly`, and `ReturnType`.
44. What is type narrowing? Explain type guards (`typeof`, `instanceof`, `in`, custom predicates).
45. What is the difference between `unknown`, `any`, and `never`?
46. What are discriminated unions and why are they powerful?
47. What is the difference between `enum` and `const enum`? What are the downsides of enums?
48. What are mapped types and conditional types? Give a simple example.

---

## Promises — dedicated track (Beginner → Advanced)

A focused progression on Promises and async, since these come up in almost every JS/full-stack interview and are central to your CV (Promise-based request queueing). Uses `P#` numbering; answers are in the matching `answers/00-javascript.md` file.

### Promises — Beginner

P1. What is a Promise? What are its three states (pending, fulfilled, rejected)?
P2. How do you create a Promise and consume it with `.then`, `.catch`, and `.finally`?
P3. What do `Promise.resolve()` and `Promise.reject()` do?
P4. What is promise chaining, and why is it better than nested callbacks ("callback hell")?
P5. What is the difference between returning a value vs returning a promise inside a `.then`?
P6. What happens if you forget to add a `.catch` (unhandled rejection)?
P7. What is `async`/`await` and how does it relate to promises? How do you handle errors with it?

### Promises — Intermediate

P8. What is the difference between `Promise.all`, `Promise.allSettled`, `Promise.race`, and `Promise.any`? (also Q19)
P9. How do you run promises in sequence vs in parallel? Show both.
P10. How do you convert a callback-based function into a promise-based one (promisify)?
P11. How do you add a timeout to a promise (reject if it takes too long)?
P12. Why is `await` inside a `for` loop often a performance problem, and how do you fix it?
P13. What is the difference between `for...of` with `await` and `array.forEach` with `await`?
P14. How does error propagation work through a chain of `.then`s?

### Promises — Advanced

P15. Explain the microtask queue and the exact output order when mixing `setTimeout`, `Promise.then`, and synchronous code. (also Q17)
P16. What does `await` actually desugar to, and why does code after `await` always run in a microtask? (also Q35)
P17. Implement your own `Promise.all`. (also Q33)
P18. Implement `Promise.allSettled` from scratch.
P19. Implement a promise-based concurrency limiter / promise pool (at most N in flight). Tie it to your affiliate request-queueing. (also Q34)
P20. Implement `retry` with exponential backoff for a flaky async function.
P21. How would you implement a promise queue that guarantees sequential execution of async tasks?
P22. What are common promise anti-patterns (the explicit-construction/`new Promise` anti-pattern, forgetting to return, nesting instead of chaining, swallowing errors)?
