---
title: JavaScript & TypeScript — Answers
topic: javascript
tags: [interview, fullstack, javascript, typescript]
related: ["[[01-react]]", "[[03-nodejs]]", "[[12-dsa-practical-coding]]"]
---

# JavaScript & TypeScript — Answers

> [!abstract] How to use this note
> Each item embeds the **question** (as a callout), a plain-English **explanation**, a worked **example**, and a **Pros / Cons** box. Answer numbers match [[questions/00-javascript|the questions file]]. New to the format? See the callout legend in [[README]].

---

## Beginner

### 1. var vs let vs const

> [!question] Q1
> What is the difference between `var`, `let`, and `const`? (commonly asked at Amazon, Microsoft)

In plain terms: these are three ways to create a variable, and they differ in **where the variable is visible** (scope) and **whether you can change it**.

- **`var`** is *function-scoped*. It ignores `{ }` blocks, is "hoisted" (moved to the top of the function) and starts as `undefined`. You can redeclare it. This old behaviour causes many bugs.
- **`let`** is *block-scoped* (only visible inside the nearest `{ }`). You can reassign it but not redeclare it in the same block.
- **`const`** is also *block-scoped* but you **cannot reassign** it. Important: `const` does not freeze the value — a `const` object can still have its properties changed; only the variable binding is locked.

> [!example]
> ```js
> function demo() {
>   if (true) {
>     var a = 1;   // visible in the whole function
>     let b = 2;   // visible only inside this if-block
>   }
>   console.log(a); // 1
>   console.log(b); // ReferenceError: b is not defined
> }
> const user = { name: 'Ana' };
> user.name = 'Bob'; // OK - we mutate the object
> // user = {};       // Error - we cannot reassign a const
> ```

> [!success] Pros / Cons
> **`const` (prefer this):** safest, signals "this won't be reassigned". Cannot be used when you must reassign.
> **`let`:** flexible for counters/reassignment; slightly less safe than `const`.
> **`var`:** avoid. Pros: none in modern code. Cons: function-scope leaks, confusing hoisting, silent redeclaration bugs.
> **Rule of thumb:** default to `const`, use `let` only when you must reassign, never use `var`.

> [!info] Further study
> - [MDN: var, let, const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)

### 2. == vs ===

> [!question] Q2
> What is the difference between `==` and `===`?

`===` (**strict equality**) checks that both the **value and the type** are the same, with no conversion. `==` (**loose equality**) first *converts* the two sides to the same type, then compares — which produces surprising results.

> [!example]
> ```js
> 0 === '';        // false (number vs string)
> 0 == '';         // true  (both become falsy/0)
> null == undefined; // true
> [] == false;     // true  (weird coercion)
> 1 === 1;         // true
> ```

> [!success] Pros / Cons
> **`===` (prefer):** predictable, no hidden conversions. Con: you must convert types yourself when needed.
> **`==`:** shorter, and `x == null` is a handy way to check for both `null` and `undefined`. Con: unpredictable coercion causes bugs.
> **Rule:** always use `===`, except the idiomatic `x == null` check.

### 3. Primitives vs reference types

> [!question] Q3
> What are the primitive types in JavaScript? How do they differ from objects (reference types)?

**Primitives** are the simplest values: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`. They are **immutable** and copied **by value** (you get an independent copy). **Objects** (including arrays and functions) are copied **by reference** — the variable holds a pointer, so two variables can point at the *same* object and a change through one is visible through the other.

> [!example]
> ```js
> let a = 10; let b = a; b++;      // b=11, a still 10 (copied by value)
> let x = { n: 1 }; let y = x; y.n = 99;
> console.log(x.n);                // 99 (both point to the same object)
> ```

> [!success] Pros / Cons
> **By value (primitives):** safe, no accidental shared changes. Con: cannot represent complex/shared data.
> **By reference (objects):** efficient for large data, enables shared state. Con: accidental mutations and hard-to-track bugs; often need copies (see [[#14 Shallow vs deep copy]]).

### 4. typeof null

> [!question] Q4
> What does `typeof null` return, and why is it considered a bug?

`typeof null` returns `"object"`. This is a **historical bug** from the very first JavaScript engine: values were tagged by a small type code, objects had tag `0`, and `null` was represented as an all-zero pointer — so it accidentally matched the "object" tag. It can never be fixed because too much existing code depends on it.

> [!example]
> ```js
> typeof null;        // "object"  (the famous bug)
> null === null;      // true      (correct way to test for null)
> ```

> [!success] Pros / Cons
> **Watch-out:** never rely on `typeof x === 'object'` to mean "real object" — it is also true for `null`. Always add `x !== null`.

### 5. Hoisting

> [!question] Q5
> What is hoisting? How does it apply to `var`, `let`, `const`, and function declarations?

Hoisting means the engine "sees" all declarations in a scope before running the code, as if they were moved to the top. But *how* they are hoisted differs:

- **`var`** — hoisted and set to `undefined`, so using it early gives `undefined` (no error).
- **`let` / `const`** — hoisted but **not initialised**; using them before their line throws a `ReferenceError` (this gap is the *temporal dead zone*, see [[#27 Temporal dead zone]]).
- **Function declarations** — fully hoisted (name and body), so you can call them before they appear.
- **Function expressions / arrows** — follow the variable rules above.

> [!example]
> ```js
> console.log(a); // undefined (var hoisted)
> var a = 1;
> greet();        // "hi" - function declaration hoisted
> function greet() { console.log('hi'); }
> // console.log(b); // ReferenceError (let in TDZ)
> let b = 2;
> ```

> [!success] Pros / Cons
> **Pro:** function declarations can be organised after their use (readable top-down code).
> **Con:** `var` hoisting hides bugs (using a value before it is set). Prefer `let`/`const` whose TDZ turns those mistakes into clear errors.

### 6. null vs undefined

> [!question] Q6
> What is the difference between `null` and `undefined`?

`undefined` = "nothing is here yet" set **by JavaScript** (an unassigned variable, a missing property, a function with no `return`). `null` = "intentionally empty" set **by you** to say "no value on purpose".

> [!example]
> ```js
> let x;              // undefined (engine)
> const obj = {};
> obj.missing;        // undefined (no such key)
> let selected = null; // null (you chose "nothing selected")
> ```

> [!success] Pros / Cons
> **Use `null`** when you deliberately want "empty"; it documents intent. **Watch-out:** `typeof null` is `"object"` while `typeof undefined` is `"undefined"`; use `x == null` to catch both at once.

### 7. Function declaration vs expression

> [!question] Q7
> What is the difference between function declarations and function expressions?

A **declaration** (`function foo(){}`) is fully hoisted, so it can be called before its line. An **expression** (`const foo = function(){}` or `const foo = () => {}`) is created only when that line runs, so calling it earlier fails.

> [!example]
> ```js
> hey();                       // works - declaration hoisted
> function hey() {}
> // ho();                     // Error - expression not created yet
> const ho = () => {};
> ```

> [!success] Pros / Cons
> **Declarations:** callable anywhere in scope; good for top-level helpers. Con: can be hoisted "too early", hiding ordering.
> **Expressions:** can be assigned conditionally, passed around, and kept private; clearer lifecycle. Con: not available before their line.

### 8. Truthy and falsy

> [!question] Q8
> What are truthy and falsy values? List all falsy values in JavaScript.

When a value is used where a boolean is expected (like an `if`), JavaScript converts it. The **falsy** values (treated as `false`) are exactly: `false`, `0`, `-0`, `0n` (BigInt zero), `""` (empty string), `null`, `undefined`, and `NaN`. **Everything else is truthy** — including `"0"`, `"false"`, `[]`, and `{}`.

> [!example]
> ```js
> if ("") console.log('no');   // skipped (empty string is falsy)
> if ([]) console.log('yes');  // runs  (empty array is truthy!)
> ```

> [!success] Pros / Cons
> **Pro:** enables short checks like `if (user)`. **Con/watch-out:** `if (count)` wrongly skips when `count === 0`; and `[]`/`{}` are truthy, which surprises many. Be explicit (`if (count > 0)`) when `0` is valid.

### 9. slice vs splice vs split

> [!question] Q9
> What is the difference between `slice`, `splice`, and `split`?

- **`slice(start, end)`** — copies a part of an array/string. Does **not** change the original.
- **`splice(start, deleteCount, ...items)`** — **changes** the array in place (removes/inserts) and returns the removed items.
- **`split(separator)`** — a **string** method that breaks a string into an array.

> [!example]
> ```js
> [1,2,3,4].slice(1,3);        // [2,3]  (original unchanged)
> const a = [1,2,3];
> a.splice(1,1);               // removes 2 -> a is now [1,3]
> 'a,b,c'.split(',');          // ['a','b','c']
> ```

> [!success] Pros / Cons
> **`slice`/`split`:** safe, non-mutating (easier to reason about). **`splice`:** powerful in-place editing but mutates — a common source of bugs when the array is shared. Prefer non-mutating methods in React state and shared data.

### 10. this basics

> [!question] Q10
> What does `this` refer to in the global scope, inside a regular function, and inside an arrow function?

`this` is a special keyword whose value depends on **how a function is called**.

- **Global scope:** the global object (`window` in the browser, `globalThis`), or `undefined` in modules/strict mode.
- **Regular function called plainly:** `undefined` in strict mode, or the global object otherwise.
- **Arrow function:** has **no own `this`** — it uses the `this` of the surrounding code (lexical `this`).

> [!example]
> ```js
> const obj = {
>   name: 'A',
>   normal() { return this.name; },       // 'A' (called as obj.normal())
>   arrow: () => this.name,               // uses outer this, not obj
> };
> ```

> [!success] Pros / Cons
> **Arrow `this`:** great for callbacks (keeps the outer `this`, no more `const self = this`). Con: cannot be used as object methods that need their own `this`, or as constructors. See [[#38 this binding rules]].

### 11. map / forEach / filter / reduce

> [!question] Q11
> What is the difference between `map`, `forEach`, `filter`, and `reduce`?

All loop over an array, but return different things:

- **`map`** → a **new array**, each item transformed.
- **`forEach`** → `undefined`; used only for side effects.
- **`filter`** → a **new array** with items that pass a test.
- **`reduce`** → a **single value** built up from all items.

> [!example]
> ```js
> const n = [1,2,3,4];
> n.map(x => x*2);              // [2,4,6,8]
> n.filter(x => x%2===0);      // [2,4]
> n.reduce((sum,x)=>sum+x,0);  // 10
> n.forEach(x => console.log(x)); // logs, returns undefined
> ```

> [!success] Pros / Cons
> **`map`/`filter`/`reduce`:** non-mutating, chainable, declarative — ideal for React and data pipelines. Con: create new arrays (tiny memory cost).
> **`forEach`:** simple for side effects. Con: cannot be chained, cannot `break`, and does **not** wait for `await` (see [[#P13 for...of await vs forEach await]]).

### 12. Spread and rest

> [!question] Q12
> What is the spread operator and the rest parameter? Give an example of each.

They share the `...` syntax but do opposite things. **Spread** *expands* a collection into individual items. **Rest** *collects* many items into one array/object.

> [!example]
> ```js
> // spread (expand)
> const merged = [...[1,2], ...[3,4]]; // [1,2,3,4]
> const copy = { ...user, active: true };
> // rest (collect)
> function sum(...nums) { return nums.reduce((a,b)=>a+b,0); }
> const { id, ...others } = user;      // others = everything except id
> ```

> [!success] Pros / Cons
> **Pro:** clean copying/merging and flexible function arguments; core to immutable updates in React/Redux. **Con:** spread makes a **shallow** copy only (nested objects stay shared — see [[#14 Shallow vs deep copy]]).

### 13. Destructuring

> [!question] Q13
> What is destructuring? Show object and array destructuring.

Destructuring is a shorthand to **pull values out** of objects/arrays into variables, with optional defaults and renaming.

> [!example]
> ```js
> // object: rename, default, nested
> const { name, age = 18, address: { city } = {} } = user;
> // array: skip and collect the rest
> const [first, , third, ...rest] = [10, 20, 30, 40, 50];
> // first=10, third=30, rest=[40,50]
> ```

> [!success] Pros / Cons
> **Pro:** shorter, readable extraction; great for function parameters and React props/state. **Con:** deep destructuring with defaults can get hard to read; overuse hurts clarity.

### 14. Shallow vs deep copy

> [!question] Q14
> What is the difference between shallow copy and deep copy?

A **shallow copy** duplicates only the top level; nested objects are still **shared** (same reference). A **deep copy** duplicates **every** level, so nothing is shared.

> [!example]
> ```js
> const original = { a: 1, nested: { b: 2 } };
> const shallow = { ...original };
> shallow.nested.b = 99;
> console.log(original.nested.b); // 99  (nested was shared!)
>
> const deep = structuredClone(original); // true deep copy
> ```

> [!success] Pros / Cons
> **Shallow (`{...obj}`, `Object.assign`, `slice`):** fast, enough when there is no nesting. Con: nested mutations leak.
> **Deep (`structuredClone`, recursive clone):** fully independent. Con: slower/more memory.
> **`JSON.parse(JSON.stringify(x))`:** quick deep copy but only for JSON-safe data — it drops functions/`undefined`, turns `Date` into a string, and crashes on circular references.

---

## Intermediate

### 15. Closure

> [!question] Q15
> What is a closure? Give a practical use case. (commonly asked at Meta, Amazon, Google)

A closure is a function that **remembers the variables from where it was created**, even after that outer function has finished. The inner function keeps a live link to those variables.

> [!example]
> ```js
> function counter() {
>   let count = 0;          // private variable
>   return () => ++count;   // this inner function "closes over" count
> }
> const inc = counter();
> inc(); // 1
> inc(); // 2  (count survived between calls)
> ```

> [!success] Pros / Cons
> **Pros:** private state/encapsulation, function factories, memoisation, keeping state in callbacks. **Cons:** the captured variables stay in memory while the closure lives, so careless closures (timers, listeners) can cause memory leaks (see [[#39 Closure and listener leaks]]).

> [!info] Further study
> - [MDN: Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)

### 16. Event loop

> [!question] Q16
> Explain the event loop. What is the difference between the call stack, task queue (macrotask), and microtask queue? (commonly asked at Meta, Netflix)

JavaScript runs on **one thread** with a **call stack** (the list of functions currently running). Slow work (timers, network, file I/O) is handed to the host (browser/Node), which later puts a callback into a **queue**. The **event loop** keeps the app moving:

1. Run all synchronous code (empty the call stack).
2. Drain the **microtask queue** completely (promise `.then`/`catch`, `queueMicrotask`).
3. Take **one macrotask** (`setTimeout`, I/O, UI event) and run it.
4. Drain microtasks again, then repeat.

The key rule: **microtasks always run before the next macrotask**.

> [!example]
> ```js
> console.log('A');
> setTimeout(() => console.log('B'));       // macrotask
> Promise.resolve().then(() => console.log('C')); // microtask
> console.log('D');
> // Order: A, D, C, B
> ```

> [!success] Pros / Cons
> **Pro:** one thread + event loop = simple model, no data races, handles thousands of I/O operations efficiently. **Con:** a heavy synchronous task blocks *everything* (UI freeze); microtask loops can starve rendering/timers.

> [!info] Further study
> - [JavaScript.info: Event loop](https://javascript.info/event-loop)
> - [Jake Archibald: In the Loop (talk)](https://www.youtube.com/watch?v=cCOL7MC4Pl0)

### 17. Output ordering

> [!question] Q17
> What will be the output order of `console.log` with `setTimeout`, `Promise.then`, and synchronous code mixed together? Explain why.

Synchronous lines run first, top to bottom. Then all **microtasks** (promise callbacks) run. Then **macrotasks** (timers). See [[#16 Event loop]] for the mechanism.

> [!example]
> ```js
> console.log(1);
> setTimeout(() => console.log(2));
> Promise.resolve().then(() => console.log(3));
> console.log(4);
> // Output: 1, 4, 3, 2
> ```

> [!success] Pros / Cons
> **Interview watch-out:** a `setTimeout(..., 0)` does **not** run "immediately" — any pending promise callback goes first. Getting this ordering right is a very common trick question.

### 18. Callbacks vs promises vs async/await

> [!question] Q18
> What is the difference between `Promise`, `async/await`, and callbacks? How do you handle errors in each?

Three generations of handling async work:

- **Callbacks** — pass a function to run later. Error handling is manual (`if (err)`), and nesting many callbacks becomes "callback hell".
- **Promises** — an object for a future value; chain with `.then`, handle errors with `.catch`.
- **async/await** — sugar over promises that *reads* like normal code; handle errors with `try/catch`.

> [!example]
> ```js
> // callback
> readFile(path, (err, data) => { if (err) return handle(err); use(data); });
> // promise
> readFileP(path).then(use).catch(handle);
> // async/await
> try { const data = await readFileP(path); use(data); } catch (err) { handle(err); }
> ```

> [!success] Pros / Cons
> **Callbacks:** universal, no dependencies. Con: nesting, messy error handling.
> **Promises:** chainable, combinators (`Promise.all`). Con: still some boilerplate.
> **async/await:** cleanest to read/debug. Con: easy to accidentally serialise independent work (see [[#P12 await in a loop]]).

### 19. Promise combinators

> [!question] Q19
> What is the difference between `Promise.all`, `Promise.allSettled`, `Promise.race`, and `Promise.any`?

They run several promises together but differ in **when they finish** and **how they treat failures**. (Full detail with examples in [[#P8 Combinators]].)

- **`all`** — waits for all; **fails fast** on the first rejection.
- **`allSettled`** — waits for all; **never rejects**, gives each result's status.
- **`race`** — settles as soon as the **first** one settles (success or failure).
- **`any`** — resolves with the **first success**; rejects only if all fail.

> [!success] Pros / Cons
> **`all`:** fast, but one failure loses all results. **`allSettled`:** robust for "do all, report each", but you must inspect statuses. **`race`:** perfect for timeouts. **`any`:** good for "first working source wins" (e.g. mirror servers).

### 20. Prototypal inheritance

> [!question] Q20
> How does prototypal inheritance work? What is the prototype chain?

Every object has a hidden link (`[[Prototype]]`) to another object. When you read a property, JavaScript looks on the object itself, then follows the link up the **prototype chain** until it finds the property or reaches `null`. This is how objects "inherit" shared behaviour. `class` syntax is just a nicer way to set this up.

> [!example]
> ```js
> const animal = { eats: true };
> const dog = Object.create(animal); // dog's prototype is animal
> dog.barks = true;
> console.log(dog.eats);  // true (found on animal via the chain)
> ```

> [!success] Pros / Cons
> **Pros:** memory-efficient (shared methods live once on the prototype), flexible/dynamic. **Cons:** deep chains slow lookups and confuse debugging; mutating built-in prototypes (`Array.prototype`) is dangerous.

### 21. call / apply / bind

> [!question] Q21
> What is the difference between `call`, `apply`, and `bind`?

All three let you set what `this` means inside a function. **`call`** and **`apply`** invoke immediately (call takes comma-separated args, apply takes an array). **`bind`** does not call it — it returns a **new** function with `this` locked in for later.

> [!example]
> ```js
> function greet(greeting) { return greeting + ' ' + this.name; }
> const user = { name: 'Sam' };
> greet.call(user, 'Hi');       // "Hi Sam"
> greet.apply(user, ['Hello']); // "Hello Sam"
> const bound = greet.bind(user);
> bound('Hey');                 // "Hey Sam"
> ```

> [!success] Pros / Cons
> **`call`/`apply`:** one-off borrowing of a method. **`bind`:** great for callbacks/event handlers where the call happens later. Con: `bind` creates a new function each time (watch out in hot render paths — prefer arrow methods or memoisation).

### 22. Debounce vs throttle

> [!question] Q22
> Explain `debounce` and `throttle`. When would you use each? (commonly asked at Uber, Meta)

Both limit how often a function runs during rapid events.

- **Debounce** = "wait until the user stops". Run only after N ms of *no* new calls. Great for search-as-you-type, autosave, window resize.
- **Throttle** = "at most once per N ms". Run on a steady schedule during continuous activity. Great for scroll, mousemove, drag.

> [!example]
> ```js
> searchInput.addEventListener('input', debounce(fetchResults, 300)); // waits for a pause
> window.addEventListener('scroll', throttle(updateNav, 100));        // steady rate
> ```

> [!success] Pros / Cons
> **Debounce:** fewest calls, but the action is delayed until activity stops. **Throttle:** guarantees regular updates (smooth UI), but runs more often than debounce. Choose by whether you need the *final* value (debounce) or *continuous* updates (throttle). Implementations: [[#31 Implement debounce]], [[#22 Debounce vs throttle|throttle in 12-dsa]].

### 23. Currying

> [!question] Q23
> What is currying? Why is it useful?

Currying turns a function that takes many arguments into a chain of functions that each take one: `f(a, b, c)` becomes `f(a)(b)(c)`. It lets you "pre-fill" some arguments now and supply the rest later (partial application).

> [!example]
> ```js
> const add = a => b => a + b;
> const add5 = add(5);   // pre-filled
> add5(3);               // 8
> ```

> [!success] Pros / Cons
> **Pros:** build specialised functions from general ones, cleaner functional composition, reusable configuration. **Cons:** extra function calls, and it can feel unfamiliar/harder to read for teams not used to functional style.

### 24. Arrow vs regular functions

> [!question] Q24
> What is the difference between arrow functions and regular functions (beyond syntax)?

Arrow functions are not just shorter. They have **no own `this`** (they use the surrounding `this`), **no `arguments` object**, **cannot be constructors** (no `new`), have no `prototype`, and cannot be generators. Regular functions get their own `this` based on how they are called.

> [!example]
> ```js
> const obj = {
>   items: [1,2],
>   log() { this.items.forEach(i => console.log(this.items, i)); } // arrow keeps obj's this
> };
> ```

> [!success] Pros / Cons
> **Arrow:** perfect for callbacks and array methods (no `this` surprises), concise. Con: cannot be object methods needing their own `this`, constructors, or `arguments`.
> **Regular:** needed for methods, constructors, generators, and dynamic `this`. Con: `this` depends on call site (a common bug source).

### 25. Event delegation

> [!question] Q25
> What is event delegation and why is it useful?

Instead of adding a listener to every child element, add **one** listener to a shared parent and use `event.target` to find which child was clicked. This works because events "bubble" up from the target to its ancestors.

> [!example]
> ```js
> list.addEventListener('click', (e) => {
>   const item = e.target.closest('li');
>   if (item) console.log('clicked', item.dataset.id);
> });
> ```

> [!success] Pros / Cons
> **Pros:** far fewer listeners (less memory), and it automatically handles items added **later**. **Cons:** needs care with `event.target` vs `currentTarget`, and some events do not bubble (e.g. `focus`, `blur` — use capture or `focusin`).

### 26. Deep vs reference equality

> [!question] Q26
> Explain the difference between deep equality and reference equality.

**Reference equality** (`===` on objects) is true only if both variables point to the **same** object in memory. **Deep equality** compares the **contents** recursively. Two different objects with identical contents are reference-unequal but deep-equal.

> [!example]
> ```js
> {a:1} === {a:1};                 // false (different objects)
> const o = {a:1}; o === o;        // true  (same reference)
> JSON.stringify({a:1}) === JSON.stringify({a:1}); // true (naive deep check)
> ```

> [!success] Pros / Cons
> **Reference check:** instant (O(1)); React uses it for fast re-render decisions. Con: says "different" for equal-looking objects.
> **Deep check:** accurate by content. Con: slow for big objects and tricky with cycles/special types — use a tested library (lodash `isEqual`).

### 27. Temporal dead zone

> [!question] Q27
> What is the temporal dead zone (TDZ)?

The TDZ is the period from entering a scope until a `let`/`const` variable's declaration line. The variable **exists** but is not initialised, so touching it throws a `ReferenceError`. It exists to catch "used before declared" mistakes.

> [!example]
> ```js
> // console.log(x); // ReferenceError (x is in the TDZ)
> let x = 5;
> console.log(x);    // 5
> ```

> [!success] Pros / Cons
> **Pro:** turns silent `undefined` bugs (old `var` behaviour) into loud, early errors. **Con:** can confuse beginners who expect hoisting to give `undefined`.

### 28. Map and Set

> [!question] Q28
> What are `Map` and `Set`? How do they differ from plain objects and arrays?

**`Map`** is a key→value collection where keys can be **any type** (even objects), it keeps insertion order, and has a `.size`. **`Set`** stores **unique** values only.

> [!example]
> ```js
> const m = new Map(); m.set('a', 1); m.set(document.body, 'node');
> const s = new Set([1,1,2,3]); // {1,2,3}
> s.has(2); // true - O(1)
> ```

> [!success] Pros / Cons
> **Map vs object:** Map allows any key type, easy `.size`, no accidental prototype keys, faster for frequent add/remove. Con: not directly JSON-serialisable.
> **Set vs array:** O(1) membership test and automatic de-duplication vs array's O(n) `includes`. Con: no index access/order operations like arrays.

### 29. Synchronous vs asynchronous / single-threaded

> [!question] Q29
> What is the difference between synchronous and asynchronous code? What is the single-threaded nature of JavaScript?

**Synchronous** code runs line by line, each step blocking the next. **Asynchronous** code starts a task now and continues; the result arrives later via the event loop. JavaScript is **single-threaded** (one call stack), so it can only do one thing at a time — which is why long synchronous work freezes the page, and why async APIs exist.

> [!example]
> ```js
> console.log('start');
> setTimeout(() => console.log('later'), 0); // async, runs after sync code
> console.log('end');
> // start, end, later
> ```

> [!success] Pros / Cons
> **Pro:** single thread avoids complex locking/race conditions; async keeps the UI responsive. **Con:** CPU-heavy work blocks everything — offload it (Web Workers in the browser, `worker_threads` in Node — see [[03-nodejs#13 cluster vs worker_threads]]).

### 30. Garbage collection and leaks

> [!question] Q30
> How does garbage collection work in JavaScript? What causes memory leaks?

JavaScript frees memory automatically using **mark-and-sweep**: starting from "roots" (global variables, the current call stack), it marks everything still reachable and deletes the rest. A **memory leak** happens when you unintentionally keep references, so the collector cannot free objects you no longer need.

> [!example]
> ```js
> // leak: interval keeps `bigData` alive forever
> let bigData = loadHugeThing();
> setInterval(() => use(bigData), 1000); // never cleared
> // fix: clearInterval when done, or null out references
> ```

> [!success] Pros / Cons
> **Pro:** automatic GC — no manual `free()`, fewer crashes. **Con:** you still leak via forgotten timers, un-removed listeners, detached DOM nodes, accidental globals, and large closures. `WeakMap`/`WeakSet` (see [[#37 WeakMap and WeakSet]]) help by not blocking collection.

> [!info] Further study
> - [MDN: Memory management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management)

---

## Advanced

### 31. Implement debounce

> [!question] Q31
> Implement `debounce` from scratch. Then add a `leading`/`trailing` option. (commonly asked at Uber, Stripe)

Debounce returns a wrapped function that resets a timer on every call and only runs `fn` after the calls stop for `delay` ms. `leading` fires on the first call; `trailing` fires after the pause.

> [!example]
> ```js
> function debounce(fn, delay, { leading = false, trailing = true } = {}) {
>   let timer = null;
>   return function (...args) {
>     const callNow = leading && !timer;
>     clearTimeout(timer);
>     timer = setTimeout(() => {
>       timer = null;
>       if (trailing && !callNow) fn.apply(this, args);
>     }, delay);
>     if (callNow) fn.apply(this, args);
>   };
> }
> ```

> [!success] Pros / Cons
> **Pro:** cuts expensive work (API calls, re-renders) during rapid input. **Cons/watch-outs:** must preserve `this` and `args` (`apply`); guard against firing twice when both `leading` and `trailing` are on; the trailing call is delayed. Complexity: O(1) per call.

### 32. Deep clone

> [!question] Q32
> Implement a deep clone function. What edge cases must you handle (circular references, `Date`, `Map`, `Set`)?

Recursively copy every level. Track already-seen objects in a `WeakMap` to survive circular references, and special-case built-in types.

> [!example]
> ```js
> function deepClone(value, seen = new WeakMap()) {
>   if (value === null || typeof value !== 'object') return value;
>   if (value instanceof Date) return new Date(value);
>   if (value instanceof RegExp) return new RegExp(value);
>   if (seen.has(value)) return seen.get(value);      // circular ref
>   if (value instanceof Map) {
>     const m = new Map(); seen.set(value, m);
>     value.forEach((v, k) => m.set(k, deepClone(v, seen)));
>     return m;
>   }
>   if (value instanceof Set) {
>     const s = new Set(); seen.set(value, s);
>     value.forEach(v => s.add(deepClone(v, seen)));
>     return s;
>   }
>   const out = Array.isArray(value) ? [] : {};
>   seen.set(value, out);
>   for (const key of Reflect.ownKeys(value)) out[key] = deepClone(value[key], seen);
>   return out;
> }
> ```

> [!success] Pros / Cons
> **Pro:** correct independent copies including cycles and special types. **Con:** slower/more memory than shallow copy; still misses exotic cases (class instances, functions). In modern runtimes prefer built-in `structuredClone` unless you need custom behaviour.

### 33. Implement Promise.all

> [!question] Q33
> Implement your own `Promise.all`.

Return a new promise that resolves when **all** inputs resolve (preserving order) and rejects on the **first** failure. Full version and notes in [[#P17 Implement Promise.all]].

> [!example]
> ```js
> function promiseAll(promises) {
>   return new Promise((resolve, reject) => {
>     const results = []; let remaining = promises.length;
>     if (remaining === 0) return resolve(results);
>     promises.forEach((p, i) => {
>       Promise.resolve(p).then(v => {
>         results[i] = v;
>         if (--remaining === 0) resolve(results);
>       }, reject);
>     });
>   });
> }
> ```

> [!success] Pros / Cons
> **Pro:** parallel execution, ordered results. **Con:** "fail fast" — one rejection discards all other results (use `allSettled` if you need every outcome).

### 34. Promise pool / concurrency limiter

> [!question] Q34
> Implement a promise-based concurrency limiter (promise pool) that runs at most N tasks at a time. (directly relevant to your affiliate request-queueing work)

Start tasks up to a limit; whenever one finishes, start the next. Full explanation in [[#P19 Concurrency limiter / promise pool]].

> [!example]
> ```js
> async function promisePool(tasks, limit) {
>   const results = []; const executing = new Set();
>   for (let i = 0; i < tasks.length; i++) {
>     const p = Promise.resolve().then(() => tasks[i]());
>     results.push(p); executing.add(p);
>     p.finally(() => executing.delete(p));
>     if (executing.size >= limit) await Promise.race(executing);
>   }
>   return Promise.all(results);
> }
> ```

> [!tip] CV tie-in
> This is exactly your affiliate **"Promise-based request queueing"** — cap concurrent API calls so downstream services are not overwhelmed, and drain as slots free up.

> [!success] Pros / Cons
> **Pro:** protects downstream services, controls memory/CPU, faster than fully sequential. **Con:** more complex than `Promise.all`; picking the right limit needs testing.

### 35. await desugaring

> [!question] Q35
> Explain the microtask vs macrotask ordering with `async/await` desugaring. What does an `await` actually compile to?

An `async` function always returns a promise. `await expr` is roughly `Promise.resolve(expr).then(continuation)`, where "the rest of the function" becomes the continuation, scheduled as a **microtask**. So code after any `await` always runs asynchronously (in a microtask), even when the value is already ready. Detail in [[#P16 await desugaring]].

> [!example]
> ```js
> async function f() { console.log(1); await null; console.log(2); }
> f(); console.log(3);
> // 1, 3, 2  (line after await runs as a microtask)
> ```

> [!success] Pros / Cons
> **Pro:** synchronous-looking async code with correct ordering. **Con/watch-out:** the microtask deferral surprises people who expect the line after `await` to run immediately.

### 36. Generators

> [!question] Q36
> What is a generator function? How can generators be used for async control flow?

A generator (`function*`) can **pause** with `yield` and **resume** with `.next()`, producing values lazily. `.next(value)` can also send a value back in, allowing two-way communication. Before native `async/await`, libraries drove generators to sequence promises (the idea behind async/await).

> [!example]
> ```js
> function* ids() { let i = 1; while (true) yield i++; }
> const gen = ids();
> gen.next().value; // 1
> gen.next().value; // 2
> ```

> [!success] Pros / Cons
> **Pros:** lazy/infinite sequences, custom iteration, pause/resume logic, memory-efficient streaming. **Cons:** unfamiliar syntax; for async work `async/await` is simpler and preferred today.

### 37. WeakMap and WeakSet

> [!question] Q37
> Explain `WeakMap` and `WeakSet`. When would you use them over `Map`/`Set`?

They hold **weak** references to their object keys/values, meaning the garbage collector can still remove those objects. Use them to attach extra data to an object that should disappear automatically when the object does.

> [!example]
> ```js
> const meta = new WeakMap();
> function tag(node) { meta.set(node, { seen: Date.now() }); }
> // when `node` is removed from the DOM and unreferenced, its entry is GC'd automatically
> ```

> [!success] Pros / Cons
> **Pros:** automatic cleanup → prevents leaks for per-object caches/metadata. **Cons:** not iterable, no `.size`, keys must be objects — because entries can vanish at any time.

### 38. this binding rules

> [!question] Q38
> How does `this` binding work with `new`? Explain the four rules of `this` binding precedence.

`this` is decided by **how the function is called**, checked in this order (highest wins):

1. **`new` binding** — `new Fn()` → `this` is the brand-new object.
2. **Explicit** — `call`/`apply`/`bind` → `this` is the argument you pass.
3. **Implicit** — `obj.method()` → `this` is `obj`.
4. **Default** — plain `fn()` → `undefined` (strict) or global object.

Arrow functions ignore all four and use the surrounding (lexical) `this`.

> [!example]
> ```js
> function F() { this.x = 1; }
> const a = new F();          // this = new object -> a.x === 1
> function g() { return this; }
> g.call({y:2});              // this = {y:2}
> ```

> [!success] Pros / Cons
> **Pro:** flexible — one function can serve as method, constructor, or borrowed function. **Con:** the same rules are a frequent bug source; arrows remove the guesswork for callbacks. Related: [[#10 this basics]], [[#21 call / apply / bind]].

### 39. Closure and listener leaks

> [!question] Q39
> What is a memory leak in a closure or event listener, and how do you prevent it?

A closure keeps its captured variables alive while the closure is reachable. A classic leak: you add an event listener whose handler captures large state and never call `removeEventListener`, so the handler **and** everything it captured (often a detached DOM node) stays in memory.

> [!example]
> ```js
> // React: clean up on unmount to avoid leaks
> useEffect(() => {
>   const onScroll = () => doWork(bigData);
>   window.addEventListener('scroll', onScroll);
>   return () => window.removeEventListener('scroll', onScroll); // cleanup
> }, []);
> ```

> [!success] Pros / Cons
> **Prevention pros:** removing listeners, using `AbortController`, nulling references, and capturing only what you need keeps memory flat. **Con/watch-out:** easy to forget cleanup in long-lived pages/SPAs — this is a common production bug (ties to [[03-nodejs#20 Debugging a memory leak]]).

### 40. memoize with key resolver

> [!question] Q40
> Implement `memoize` with a custom cache key resolver.

Memoisation caches a function's result by its arguments, so repeated calls with the same inputs skip the work.

> [!example]
> ```js
> function memoize(fn, resolver = (...args) => JSON.stringify(args)) {
>   const cache = new Map();
>   return function (...args) {
>     const key = resolver(...args);
>     if (cache.has(key)) return cache.get(key);
>     const result = fn.apply(this, args);
>     cache.set(key, result);
>     return result;
>   };
> }
> ```

> [!success] Pros / Cons
> **Pro:** big speed-up for pure, expensive, repeated calls. **Cons:** uses memory (cache can grow unbounded — add size/TTL limits); only safe for **pure** functions; the key resolver must uniquely represent the arguments. For object arguments, a `WeakMap` key avoids leaks.

---

## TypeScript essentials

### 41. interface vs type

> [!question] Q41
> What is the difference between `interface` and `type`? When would you use each?

Both describe the shape of data. **`interface`** can be re-opened and extended (declaration merging) and reads naturally for object/class contracts. **`type`** is more flexible — it can also describe unions, intersections, tuples, primitives, and computed (mapped/conditional) types.

> [!example]
> ```ts
> interface User { id: number; name: string; }
> interface User { email: string; } // merged: User now has all three
>
> type Status = 'active' | 'inactive';   // union - only type can do this
> type WithId<T> = T & { id: number };    // intersection
> ```

> [!success] Pros / Cons
> **`interface`:** extendable/mergeable, great for public APIs and class contracts. Con: cannot express unions directly.
> **`type`:** can express anything (unions, tuples, mapped types). Con: cannot be merged/re-opened. **Rule:** interfaces for object/class shapes, `type` for unions and computed types.

### 42. Generics

> [!question] Q42
> What are generics? Write a generic function and a generic constraint (`extends`).

Generics are **type variables** — they let one function/type work with many types while keeping the relationship between input and output. `extends` constrains what types are allowed.

> [!example]
> ```ts
> function identity<T>(value: T): T { return value; }
> identity<number>(5);      // T = number
>
> function len<T extends { length: number }>(x: T): number { return x.length; }
> len('abc'); len([1,2]);   // ok; len(5) -> error (no .length)
> ```

> [!success] Pros / Cons
> **Pros:** reusable **and** type-safe code (no `any`), preserves types through the call. **Cons:** overusing generics makes signatures hard to read; sometimes a simple union is clearer.

### 43. Utility types

> [!question] Q43
> Explain the utility types `Partial`, `Pick`, `Omit`, `Record`, `Readonly`, and `ReturnType`.

Built-in helpers that transform existing types so you do not repeat yourself.

> [!example]
> ```ts
> interface User { id: number; name: string; age: number; }
> type Draft = Partial<User>;              // all optional
> type NameOnly = Pick<User, 'name'>;      // { name: string }
> type NoId = Omit<User, 'id'>;            // without id
> type Book = Record<string, number>;      // { [key: string]: number }
> type Frozen = Readonly<User>;            // all read-only
> type R = ReturnType<() => User>;         // User
> ```

> [!success] Pros / Cons
> **Pro:** derive many types from one source of truth (less duplication, fewer mismatches). **Con:** stacking many utilities becomes cryptic; name the intermediate types for clarity.

> [!info] Further study
> - [TypeScript: Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)

### 44. Type narrowing and guards

> [!question] Q44
> What is type narrowing? Explain type guards (`typeof`, `instanceof`, `in`, custom predicates).

Narrowing is when TypeScript **refines** a broad type to a more specific one inside a branch, based on a check. Those checks are "type guards".

> [!example]
> ```ts
> function print(x: string | number) {
>   if (typeof x === 'string') x.toUpperCase(); // x narrowed to string
>   else x.toFixed(2);                           // x narrowed to number
> }
> // custom predicate
> function isCat(a: Animal): a is Cat { return 'meow' in a; }
> ```

> [!success] Pros / Cons
> **Pro:** safe access to type-specific members without casting; compiler-checked branches. **Con:** custom predicates (`a is Cat`) are trusted by the compiler — if you write the check wrong, you get a silent unsafe cast.

### 45. unknown vs any vs never

> [!question] Q45
> What is the difference between `unknown`, `any`, and `never`?

- **`any`** — turns **off** type checking (anything goes). Unsafe.
- **`unknown`** — can hold anything, but you **must narrow** it before use (the safe version of `any`).
- **`never`** — a value that can **never** happen (a function that always throws, or an impossible branch).

> [!example]
> ```ts
> let a: any = 5; a.foo.bar;        // compiles, may crash at runtime
> let u: unknown = 5;
> // u.toFixed();                    // error - must narrow first
> if (typeof u === 'number') u.toFixed();
> function fail(): never { throw new Error(); }
> ```

> [!success] Pros / Cons
> **`unknown` (prefer over `any`):** keeps safety at boundaries (e.g. `JSON.parse`, API data). **`any`:** convenient escape hatch but disables the whole point of TypeScript. **`never`:** enables exhaustiveness checks in unions (see [[#46 Discriminated unions]]).

### 46. Discriminated unions

> [!question] Q46
> What are discriminated unions and why are they powerful?

A union of object types that all share one **literal "tag"** field. Switching on that tag lets TypeScript know exactly which shape you have and can enforce that you handle every case.

> [!example]
> ```ts
> type Shape =
>   | { kind: 'circle'; r: number }
>   | { kind: 'square'; side: number };
> function area(s: Shape) {
>   switch (s.kind) {
>     case 'circle': return Math.PI * s.r ** 2; // s is the circle shape
>     case 'square': return s.side ** 2;
>   }
> }
> ```

> [!success] Pros / Cons
> **Pros:** compile-time safety for state modelling (loading/success/error), exhaustiveness checks with `never`, self-documenting. **Cons:** requires a discriminant field on every member and a little boilerplate.

### 47. enum vs const enum

> [!question] Q47
> What is the difference between `enum` and `const enum`? What are the downsides of enums?

A normal **`enum`** compiles to a real object at runtime (extra JS, supports reverse lookup for numeric enums). A **`const enum`** is **inlined** at compile time (no runtime object, smaller output) but has caveats with certain bundlers/`isolatedModules`.

> [!example]
> ```ts
> enum Color { Red, Green }        // produces a runtime object
> const enum Size { S, M }         // inlined: Size.S becomes 0 directly
> // Modern alternative many teams prefer:
> const Role = { Admin: 'admin', User: 'user' } as const;
> type Role = typeof Role[keyof typeof Role];
> ```

> [!success] Pros / Cons
> **`enum`:** familiar, groups related constants. Con: adds runtime code, non-idiomatic JS output.
> **`const enum`:** zero runtime cost. Con: bundler/tooling caveats.
> **Union of string literals / `as const`:** no runtime cost, tree-shakeable, idiomatic — often the best choice.

### 48. Mapped and conditional types

> [!question] Q48
> What are mapped types and conditional types? Give a simple example.

A **mapped type** transforms every property of a type. A **conditional type** picks a type based on a condition (`T extends U ? X : Y`), and `infer` extracts a type from inside another.

> [!example]
> ```ts
> type Optional<T> = { [K in keyof T]?: T[K] };        // mapped
> type Flatten<T> = T extends Array<infer U> ? U : T;   // conditional + infer
> type A = Flatten<string[]>; // string
> ```

> [!success] Pros / Cons
> **Pros:** powerful, DRY type transformations (the engine behind utility types). **Cons:** advanced and hard to read/debug; keep them shallow and well-named, or they become write-only code.

---

## Promises — dedicated track (Beginner → Advanced)

> [!abstract]
> A focused progression on Promises and async — the most common JS interview area and central to your CV (Promise-based request queueing). Numbers use `P#` to match [[questions/00-javascript|the questions file]].

### Promises — Beginner

#### P1. What is a Promise & its states

> [!question] P1
> What is a Promise? What are its three states (pending, fulfilled, rejected)?

A Promise is an object that represents a value that will be ready **later** (the result of an async operation). It is always in one of three states: **pending** (still working), **fulfilled** (finished with a value), or **rejected** (failed with a reason). Once it settles (fulfilled or rejected) it **cannot change again**.

> [!example]
> ```js
> const p = fetch('/api'); // pending now...
> p.then(res => /* fulfilled with res */ res.json())
>  .catch(err => /* rejected with err */ console.error(err));
> ```

> [!success] Pros / Cons
> **Pros:** a clean object you can pass around, chain, and combine; one-time settled result. **Cons:** cannot be cancelled natively (need `AbortController`), and a settled promise never re-runs.

> [!info] Further study
> - [MDN: Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)

#### P2. Creating and consuming

> [!question] P2
> How do you create a Promise and consume it with `.then`, `.catch`, and `.finally`?

You create one with `new Promise((resolve, reject) => {...})`. Call `resolve(value)` on success or `reject(error)` on failure. You consume it with `.then` (success), `.catch` (failure), and `.finally` (always).

> [!example]
> ```js
> const p = new Promise((resolve, reject) => {
>   setTimeout(() => resolve('done'), 100); // or reject(new Error('failed'))
> });
> p.then(v => console.log(v))
>  .catch(err => console.error(err))
>  .finally(() => console.log('cleanup'));
> ```

> [!success] Pros / Cons
> **Pro:** clear success/error/cleanup separation. **Con/watch-out:** the executor runs **immediately** and synchronously; only wrap in `new Promise` when bridging non-promise APIs (see the anti-pattern in [[#P22 Anti-patterns]]).

#### P3. Promise.resolve / Promise.reject

> [!question] P3
> What do `Promise.resolve()` and `Promise.reject()` do?

Shortcuts to create an already-settled promise. `Promise.resolve(value)` is already fulfilled (and adopts the state of a promise you pass it); `Promise.reject(reason)` is already rejected.

> [!example]
> ```js
> Promise.resolve(42).then(v => console.log(v)); // 42
> Promise.reject(new Error('no')).catch(e => console.log(e.message)); // no
> ```

> [!success] Pros / Cons
> **Pro:** handy to start a chain, return a cached value from an async-signature function, or normalise "value or promise" into a promise. **Con:** `Promise.reject` with no `.catch` becomes an unhandled rejection (see [[#P6 Missing .catch]]).

#### P4. Promise chaining

> [!question] P4
> What is promise chaining, and why is it better than nested callbacks ("callback hell")?

Each `.then` returns a **new** promise, so you can line up async steps **flatly** and handle all errors with **one** `.catch`. Nested callbacks instead grow rightwards ("pyramid of doom") with error handling repeated everywhere.

> [!example]
> ```js
> // chained (flat, one error handler)
> fetchUser()
>   .then(user => fetchOrders(user.id))
>   .then(orders => render(orders))
>   .catch(handleError);
> ```

> [!success] Pros / Cons
> **Pros:** flat, readable, unified error handling, easy to insert steps. **Cons:** still more verbose than `async/await`; forgetting to `return` inside `.then` breaks the chain (see [[#P5 Returning a value vs a promise]]).

#### P5. Returning a value vs a promise in .then

> [!question] P5
> What is the difference between returning a value vs returning a promise inside a `.then`?

If you return a **plain value**, the next `.then` gets it directly. If you return a **promise**, the chain **waits** for that promise and passes its resolved value on. This "flattening" is how you sequence dependent async calls.

> [!example]
> ```js
> getId()
>   .then(id => fetchUser(id))   // returns a promise -> chain waits
>   .then(user => user.name);    // gets the resolved user
> ```

> [!success] Pros / Cons
> **Pro:** natural sequencing of async steps. **Con/watch-out:** forgetting `return` (e.g. `.then(id => { fetchUser(id); })`) means the chain does **not** wait — a very common bug.

#### P6. Missing .catch

> [!question] P6
> What happens if you forget to add a `.catch` (unhandled rejection)?

The rejection has nowhere to go and becomes an **unhandled promise rejection**. Browsers fire an `unhandledrejection` event and log a warning; Node emits `unhandledRejection` and, in modern versions, may **crash** the process.

> [!example]
> ```js
> // risky - no catch
> fetch('/api').then(r => r.json());
> // safe
> fetch('/api').then(r => r.json()).catch(err => report(err));
> ```

> [!success] Pros / Cons
> **Best practice pros:** always attaching `.catch` (or `try/catch` with `await`) prevents crashes and silent failures. **Con:** none — unhandled rejections are purely a risk.

#### P7. async / await

> [!question] P7
> What is `async`/`await` and how does it relate to promises? How do you handle errors with it?

`async` marks a function that **always returns a promise**. `await` pauses inside it until a promise settles, giving the resolved value (or throwing on rejection). It is sugar over promises that reads like normal, synchronous code. Handle errors with `try/catch`.

> [!example]
> ```js
> async function load() {
>   try {
>     const user = await fetchUser();
>     return user;
>   } catch (err) {
>     handle(err);
>   }
> }
> ```

> [!success] Pros / Cons
> **Pros:** cleanest to read and debug (real stack traces, normal `try/catch`). **Cons:** easy to accidentally serialise independent awaits (slow — see [[#P12 await in a loop]]); every `await` still yields to the microtask queue.

### Promises — Intermediate

#### P8. Combinators

> [!question] P8
> What is the difference between `Promise.all`, `Promise.allSettled`, `Promise.race`, and `Promise.any`? (also [[#19 Promise combinators|Q19]])

Four ways to combine multiple promises:

- **`all`** — resolves with **all** results in order; **rejects on the first** failure.
- **`allSettled`** — **never rejects**; returns `{status, value|reason}` for each.
- **`race`** — settles with the **first to settle** (success or failure).
- **`any`** — resolves with the **first success**; rejects only if **all** fail (`AggregateError`).

> [!example]
> ```js
> await Promise.all([a(), b()]);          // both, or throw on first fail
> await Promise.allSettled([a(), b()]);   // [{status,...},{status,...}]
> await Promise.race([task(), timeout()]);// whichever finishes first
> await Promise.any([mirror1(), mirror2()]); // first that works
> ```

> [!success] Pros / Cons
> **`all`:** fast; con: loses all results on one failure. **`allSettled`:** robust; con: you must check statuses. **`race`:** great for timeouts; con: a fast rejection wins too. **`any`:** "first success"; con: waits for all to fail before rejecting.

#### P9. Sequence vs parallel

> [!question] P9
> How do you run promises in sequence vs in parallel? Show both.

Sequential = wait for each before starting the next (needed when steps depend on each other). Parallel = start them all, then wait (faster for independent work).

> [!example]
> ```js
> // sequential
> for (const id of ids) { await process(id); }
> // parallel
> await Promise.all(ids.map(id => process(id)));
> ```

> [!success] Pros / Cons
> **Sequential:** required for dependent steps or to limit load; con: slow (sum of durations). **Parallel:** fast (max duration); con: can overwhelm services — combine with a concurrency limit ([[#P19 Concurrency limiter / promise pool]]).

#### P10. Promisify

> [!question] P10
> How do you convert a callback-based function into a promise-based one (promisify)?

Wrap the callback API in a `new Promise`: call `resolve` on success and `reject` on error.

> [!example]
> ```js
> const readFileP = (path) => new Promise((resolve, reject) =>
>   fs.readFile(path, 'utf8', (err, data) => err ? reject(err) : resolve(data)));
> // Node built-in:
> const { promisify } = require('util');
> const readFileP2 = promisify(fs.readFile);
> ```

> [!success] Pros / Cons
> **Pro:** lets old callback APIs join promise chains / `await`. **Con:** manual wrapping is error-prone (forgetting a branch); prefer `util.promisify` for standard error-first callbacks.

#### P11. Timeout a promise

> [!question] P11
> How do you add a timeout to a promise (reject if it takes too long)?

Race the real promise against a timer that rejects after N ms — whichever settles first wins.

> [!example]
> ```js
> const withTimeout = (promise, ms) => Promise.race([
>   promise,
>   new Promise((_, reject) => setTimeout(() => reject(new Error('Timeout')), ms)),
> ]);
> await withTimeout(fetch('/slow'), 3000);
> ```

> [!success] Pros / Cons
> **Pro:** protects against hanging requests, improves UX. **Con:** the underlying operation is **not** actually cancelled (it keeps running) unless you also use `AbortController`; the timer should be cleared to avoid leaks.

#### P12. await in a loop

> [!question] P12
> Why is `await` inside a `for` loop often a performance problem, and how do you fix it?

`for (const x of list) await fn(x)` runs the calls **one at a time**, so total time is the **sum** of all durations. If the calls are independent, run them together instead.

> [!example]
> ```js
> // slow: sequential
> for (const url of urls) { await fetch(url); }
> // fast: parallel
> await Promise.all(urls.map(url => fetch(url)));
> ```

> [!success] Pros / Cons
> **Sequential await:** correct when each step depends on the last, or to throttle load. Con: slow. **Parallel:** much faster; con: unbounded parallelism can overload the server — cap it with a promise pool.

#### P13. for...of await vs forEach await

> [!question] P13
> What is the difference between `for...of` with `await` and `array.forEach` with `await`?

`for...of` genuinely **pauses** at each `await`. `array.forEach(async ...)` does **not** wait — `forEach` ignores the returned promises and fires all callbacks at once, so surrounding code runs before they finish.

> [!example]
> ```js
> for (const x of list) { await save(x); }   // waits each time (correct)
> list.forEach(async x => { await save(x); }); // does NOT wait - bug
> await Promise.all(list.map(x => save(x))); // parallel, awaited (correct)
> ```

> [!success] Pros / Cons
> **`for...of` await:** sequential and awaitable. **`forEach` async:** almost always a bug — cannot be awaited. **Rule:** use `for...of` (sequential) or `Promise.all(map(...))` (parallel), never `forEach` with async.

#### P14. Error propagation

> [!question] P14
> How does error propagation work through a chain of `.then`s?

An error thrown (or a rejection) anywhere in a chain **skips** the remaining `.then` handlers and jumps to the next `.catch`. If that `.catch` handles it without re-throwing, the chain **continues** as fulfilled.

> [!example]
> ```js
> step1()
>   .then(step2)     // if step1/step2 throws...
>   .then(step3)     // ...this is skipped...
>   .catch(handle)   // ...and this runs
>   .then(cleanup);  // then chain resumes here
> ```

> [!success] Pros / Cons
> **Pro:** one place to handle all errors in a flow (like `try/catch` for a block). **Con/watch-out:** a `.catch` that swallows the error silently lets the chain continue with unexpected state — re-throw if you cannot recover.

### Promises — Advanced

#### P15. Microtask ordering

> [!question] P15
> Explain the microtask queue and the exact output order when mixing `setTimeout`, `Promise.then`, and synchronous code. (also [[#17 Output ordering|Q17]])

Promise callbacks go on the **microtask** queue, which fully drains after the current synchronous code and **before** the next macrotask (`setTimeout`). See [[#16 Event loop]].

> [!example]
> ```js
> console.log('A');
> setTimeout(() => console.log('B'));           // macrotask
> Promise.resolve().then(() => console.log('C')); // microtask
> console.log('D');
> // A, D, C, B
> ```

> [!success] Pros / Cons
> **Pro:** microtasks give promises a predictable "run soon, before timers" priority. **Con:** an endless microtask chain can starve timers and rendering (the page never repaints).

#### P16. await desugaring

> [!question] P16
> What does `await` actually desugar to, and why does code after `await` always run in a microtask? (also [[#35 await desugaring|Q35]])

`await expr` is roughly `Promise.resolve(expr).then(continuation)`, where the rest of the async function becomes the `continuation` scheduled as a **microtask**. So even `await Promise.resolve(1)` defers the following lines to a microtask.

> [!example]
> ```js
> async function f() { console.log(1); await 0; console.log(2); }
> f(); console.log(3);
> // 1, 3, 2
> ```

> [!success] Pros / Cons
> **Pro:** consistent, predictable async ordering. **Con:** the "always a microtask" rule is a classic interview trap and a real source of ordering bugs.

#### P17. Implement Promise.all

> [!question] P17
> Implement your own `Promise.all`. (also [[#33 Implement Promise.all|Q33]])

Resolve with all results in order once every input resolves; reject as soon as any input rejects.

> [!example]
> ```js
> function promiseAll(promises) {
>   return new Promise((resolve, reject) => {
>     const results = []; let remaining = promises.length;
>     if (remaining === 0) return resolve(results);
>     promises.forEach((p, i) => {
>       Promise.resolve(p).then(v => {
>         results[i] = v;                 // keep original order
>         if (--remaining === 0) resolve(results);
>       }, reject);                       // first rejection wins
>     });
>   });
> }
> ```

> [!success] Pros / Cons
> **Pro:** parallel, ordered results. **Con:** fail-fast discards other results; use a from-scratch `allSettled` ([[#P18 Implement Promise.allSettled]]) when you need every outcome.

#### P18. Implement Promise.allSettled

> [!question] P18
> Implement `Promise.allSettled` from scratch.

Map each promise to a descriptor that never rejects, then use `Promise.all`.

> [!example]
> ```js
> function allSettled(promises) {
>   return Promise.all(promises.map(p =>
>     Promise.resolve(p).then(
>       value => ({ status: 'fulfilled', value }),
>       reason => ({ status: 'rejected', reason })
>     )
>   ));
> }
> ```

> [!success] Pros / Cons
> **Pro:** you always get every result, success or failure — ideal for batch jobs where partial failure is acceptable. **Con:** you must inspect each `status`; slower to "know it failed" than fail-fast `all`.

#### P19. Concurrency limiter / promise pool

> [!question] P19
> Implement a promise-based concurrency limiter / promise pool (at most N in flight). Tie it to your affiliate request-queueing. (also [[#34 Promise pool / concurrency limiter|Q34]])

Keep at most `limit` tasks running; when one finishes (`Promise.race`), start the next.

> [!example]
> ```js
> async function promisePool(tasks, limit) {
>   const results = []; const executing = new Set();
>   for (let i = 0; i < tasks.length; i++) {
>     const p = Promise.resolve().then(() => tasks[i]());
>     results.push(p); executing.add(p);
>     p.finally(() => executing.delete(p));
>     if (executing.size >= limit) await Promise.race(executing);
>   }
>   return Promise.all(results);
> }
> ```

> [!tip] CV tie-in
> This is your affiliate **"Promise-based request queueing"**: bound concurrent API calls so downstream services are not overwhelmed, and drain as slots free up. See system design in [[06-system-design#12 Design a rate limiter]].

> [!success] Pros / Cons
> **Pros:** protects downstream systems, steady memory/CPU, still much faster than fully sequential. **Cons:** more complex; the ordering of results vs completion needs care; the ideal `limit` needs load testing.

#### P20. Retry with exponential backoff

> [!question] P20
> Implement `retry` with exponential backoff for a flaky async function.

Retry on failure, waiting longer each time (double the delay), up to a limit.

> [!example]
> ```js
> async function retry(fn, retries = 3, delay = 500) {
>   try { return await fn(); }
>   catch (err) {
>     if (retries <= 0) throw err;
>     await new Promise(r => setTimeout(r, delay));
>     return retry(fn, retries - 1, delay * 2); // exponential backoff
>   }
> }
> ```

> [!tip] CV tie-in
> This is how you make **webhook/queue processing resilient** to transient failures — see [[04-nestjs#21 Reliable webhooks]] and [[06-system-design#16 Webhook delivery/consumption]].

> [!success] Pros / Cons
> **Pros:** survives transient errors, backoff avoids hammering a struggling service. **Cons:** retrying **non-idempotent** operations can cause duplicates; add **jitter** (randomness) so many clients do not retry in lockstep; cap total wait time.

#### P21. Sequential promise queue

> [!question] P21
> How would you implement a promise queue that guarantees sequential execution of async tasks?

Chain each new task onto a running "tail" promise, so tasks run strictly one after another no matter when they are added.

> [!example]
> ```js
> class PromiseQueue {
>   constructor() { this.tail = Promise.resolve(); }
>   add(task) {
>     const run = this.tail.then(() => task());
>     this.tail = run.catch(() => {}); // one failure must not break the chain
>     return run;
>   }
> }
> ```

> [!success] Pros / Cons
> **Pros:** guarantees order and prevents overlap (e.g. ordered writes, sequential UI animations). **Cons:** no parallelism (throughput limited to one at a time); must isolate failures so one bad task does not stall the queue.

#### P22. Anti-patterns

> [!question] P22
> What are common promise anti-patterns (the explicit-construction/`new Promise` anti-pattern, forgetting to return, nesting instead of chaining, swallowing errors)?

Common mistakes:
1. **Explicit-construction anti-pattern** — wrapping an already-promise-returning call in `new Promise` instead of just returning it.
2. **Forgetting to `return`** a promise inside `.then` (the chain does not wait).
3. **Nesting** `.then`s instead of chaining (recreates callback hell).
4. **Swallowing errors** with an empty/missing `.catch`.
5. **Mixing** `await` with unnecessary `.then`.
6. Using `forEach` with `async`.

> [!example]
> ```js
> // anti-pattern: pointless wrapper
> function bad(id) { return new Promise((res, rej) => {
>   fetchUser(id).then(res).catch(rej); // just: return fetchUser(id)
> }); }
> // good
> function good(id) { return fetchUser(id); }
> ```

> [!success] Pros / Cons
> **Avoiding these:** flatter, correct, debuggable async code. **Con of ignoring them:** subtle timing bugs, silent failures, and hard-to-read pyramids. **Rule:** prefer `async/await`, always return promises, always handle errors.

---

## Related notes
- [[01-react]] — hooks, effects, and `this`/closure pitfalls in components.
- [[03-nodejs]] — event loop phases, `worker_threads`, memory leaks in the backend.
- [[12-dsa-practical-coding]] — machine-coding versions of debounce, deep clone, `Promise.all`, promise pool, retry.

## References & Further Study
- [MDN: JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide) — the canonical reference for every topic above.
- [JavaScript.info](https://javascript.info/) — clear, beginner-friendly deep dives (closures, prototypes, event loop, promises).
- [MDN: Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises) and [MDN: Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) — interfaces, generics, narrowing, utility/mapped/conditional types.
- [Jake Archibald: In The Loop](https://www.youtube.com/watch?v=cCOL7MC4Pl0) — the definitive event loop talk.
