---
title: Testing — Answers
topic: testing
tags: [interview, testing, jest]
related: ["[[01-react]]", "[[04-nestjs]]"]
---

# Testing — Answers

> [!abstract] How to use this note
> Each item embeds the **question**, a plain-English **explanation**, a worked **example**, and a **Pros / Cons** box. Numbers match [[questions/13-testing|the questions file]]. NestJS items (Q15–Q16) are expanded for backend interview rounds.

---

## Beginner

### 1. Why is automated testing important?

> [!question] Q1
> Why is automated testing important? What does it give you beyond manual testing?

Automated tests are **code that verifies your code**. You write them once, then run them on every change — locally and in CI — in seconds. Manual testing means a human clicks through the app every time something changes: slow, expensive, inconsistent, and impossible to repeat for every edge case on every pull request.

> [!example]
> ```js
> // Without tests: "I changed the discount logic — did I break checkout?"
> // → manually test 20 flows, hope you remember them all.
>
> // With tests: one command
> // npm test  →  847 tests in 12s  →  confidence to merge
> ```

> [!success] Pros / Cons
> **Pros:** catches **regressions** early; documents expected behaviour; enables **safe refactoring**; runs fast and repeatedly; scales with team size; blocks broken code in CI.
> **Cons:** upfront cost to write and maintain tests; can give false confidence if assertions are weak; requires discipline and tooling setup.
> **Beyond manual:** repeatability, speed, coverage of edge cases you would forget, and automatic execution on every PR.

> [!info] Further study
> - [Jest: Getting Started](https://jestjs.io/docs/getting-started)

### 2. Unit vs integration vs end-to-end tests

> [!question] Q2
> What is the difference between unit, integration, and end-to-end (e2e) tests?

Think of **scope** and **confidence vs speed**:

- **Unit test:** one function, class, or component in **isolation** — dependencies are mocked. Fast, pinpoint failures.
- **Integration test:** several real pieces working **together** — e.g. a service with a real (test) database, or a React component with its Redux store. Catches wiring bugs mocks miss.
- **End-to-end (e2e) test:** the **whole system** as a user would use it — browser automation (Playwright/Cypress) or HTTP calls against a running API (supertest). Highest confidence, slowest and most brittle.

> [!example]
> ```text
> Unit:        calculateDiscount(100, 0.2) → 80        (mock nothing)
> Integration: OrderService.create() → row in test DB   (real DB, mock payment API)
> E2e:         User clicks "Buy" → sees confirmation     (full stack running)
> ```

> [!success] Pros / Cons
> **Unit:** fast (ms), cheap, precise — but misses integration bugs.
> **Integration:** catches real wiring/DB issues — slower, needs test infrastructure.
> **E2e:** highest user confidence — slow (seconds/minutes), flaky if overused.
> **Rule:** use all three at different levels; don't replace unit tests with only e2e.

### 3. The testing pyramid

> [!question] Q3
> What is the testing pyramid?

A **guideline for test mix**, not a law. The base is **many fast unit tests**; the middle has **fewer integration tests**; the top has **a small number of slow e2e tests**. Shape like a pyramid because unit tests are cheap to run in CI on every commit; e2e tests are expensive.

> [!example]
> ```text
>         /  E2E  \          ← few (critical user journeys)
>        / Integr. \         ← some (service + DB, API contracts)
>       /   Unit    \        ← many (pure logic, components in isolation)
> ```

> [!success] Pros / Cons
> **Pyramid (recommended):** fast feedback, low CI cost, good coverage of logic at the base.
> **Ice-cream cone (anti-pattern):** mostly e2e, few unit tests — slow CI, hard to debug failures, flaky pipeline.
> **Trophy / honeycomb variants:** valid for microservices or heavy UI — still keep most logic tested at low levels.

> [!info] Further study
> - [Martin Fowler: The Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)

### 4. Mock, stub, and spy

> [!question] Q4
> What is a mock, a stub, and a spy?

Test **doubles** replace real dependencies:

- **Stub:** returns **canned data** — "when you call `getUser(1)`, return `{ id: 1, name: 'Ana' }`." No assertion on how it was called.
- **Mock:** a stub **plus expectations** — "I expect `sendEmail` to be called exactly once with `{ to: 'a@b.com' }`." You assert on call count and arguments after the act.
- **Spy:** wraps a **real function** (or provides a passthrough) and **records** calls — args, return values, call count — without necessarily changing behaviour. `jest.spyOn(console, 'log')` is a spy.

> [!example]
> ```js
> const stub = jest.fn().mockReturnValue(42);           // stub
> const mock = jest.fn().mockResolvedValue({ ok: true }); // mock (assert later)
> const spy = jest.spyOn(api, 'fetchUser');              // spy on real method
>
> await service.doWork();
> expect(mock).toHaveBeenCalledWith({ id: 1 });
> expect(spy).toHaveBeenCalledTimes(1);
> spy.mockRestore();
> ```

> [!success] Pros / Cons
> **Stub:** simple, isolates unit under test — doesn't verify interactions.
> **Mock:** verifies behaviour/contracts — tests can become brittle if over-specified.
> **Spy:** observe real code paths — must restore after test to avoid leaking state.
> **Jest:** `jest.fn()`, `jest.spyOn()`, and `jest.mock()` cover all three patterns.

> [!info] Further study
> - [Jest: Mock Functions](https://jestjs.io/docs/mock-functions)

### 5. AAA pattern (Arrange–Act–Assert)

> [!question] Q5
> What is the AAA (Arrange-Act-Assert) pattern?

Structure every test in **three clear phases**:

1. **Arrange** — set up inputs, mocks, and preconditions.
2. **Act** — call the one thing you are testing.
3. **Assert** — verify the outcome (return value, side effect, thrown error).

One behaviour per test. If Arrange is huge, extract a helper or factory.

> [!example]
> ```js
> it('applies 20% discount to orders over $100', () => {
>   // Arrange
>   const order = { total: 150, items: [{ price: 150 }] };
>
>   // Act
>   const result = applyDiscount(order, { threshold: 100, rate: 0.2 });
>
>   // Assert
>   expect(result.total).toBe(120);
> });
> ```

> [!success] Pros / Cons
> **Pros:** readable, consistent, easy to spot missing setup or assertions.
> **Cons:** can feel verbose for trivial one-liners — still worth the structure in interviews and team codebases.
> **Variant:** Given-When-Then (BDD) — same idea, different names.

### 6. Code coverage

> [!question] Q6
> What is code coverage and is 100% coverage a good goal?

**Code coverage** measures what percentage of your source lines/branches/functions were **executed** during tests. Jest reports this with `--coverage`. It is a **signal for untested areas**, not proof of quality.

> [!example]
> ```bash
> npm test -- --coverage
> # Statements: 78%  Branches: 65%  Functions: 82%  Lines: 78%
> ```

> [!success] Pros / Cons
> **Useful coverage (60–80%+ on critical paths):** finds dead code and gaps; trends upward over time.
> **100% as a hard gate:** bad goal — you can execute every line with weak `expect(true).toBe(true)` assertions; wastes time on getters, boilerplate, and generated code.
> **Better goal:** meaningful tests on **business logic**, edge cases, and error paths — coverage rises as a side effect.
> **Watch branch coverage:** higher bar than line coverage — catches untested `if/else`.

> [!info] Further study
> - [Jest: Configuration — collectCoverageFrom](https://jestjs.io/docs/configuration#collectcoveragefrom-array)

---

## Intermediate

### 7. Writing a unit test in Jest

> [!question] Q7
> How do you write a unit test in Jest? What do `describe`, `it`, `expect`, `beforeEach` do?

Jest is the default test runner in most React and Node projects. **`describe`** groups related tests into a suite. **`it`** (alias **`test`**) defines one test case. **`expect(value).matcher()`** asserts outcomes. **`beforeEach`** runs setup before **each** test in the block; **`afterEach`** cleans up. **`beforeAll`/`afterAll`** run once per suite.

> [!example]
> ```js
> // sum.js
> export function sum(a, b) { return a + b; }
>
> // sum.test.js
> import { sum } from './sum';
>
> describe('sum', () => {
>   beforeEach(() => {
>     // fresh setup per test — reset mocks, seed data, etc.
>   });
>
>   it('adds two positive numbers', () => {
>     expect(sum(2, 3)).toBe(5);
>   });
>
>   it('handles negatives', () => {
>     expect(sum(-1, 1)).toBe(0);
>   });
> });
> ```

> [!success] Pros / Cons
> **Common matchers:** `toBe` (strict equality), `toEqual` (deep), `toThrow`, `toHaveBeenCalledWith`, `resolves`/`rejects`.
> **`describe` nesting:** organize by feature/method — avoid more than 2–3 levels deep.
> **`beforeEach` vs `beforeAll`:** prefer `beforeEach` for isolation; `beforeAll` only when setup is expensive and tests don't mutate shared state.

> [!info] Further study
> - [Jest: Using Matchers](https://jestjs.io/docs/using-matchers)
> - [Jest: Setup and Teardown](https://jestjs.io/docs/setup-teardown)

### 8. Mocking modules and functions in Jest

> [!question] Q8
> How do you mock a module or function in Jest?

**`jest.mock('./module')`** hoists and replaces an entire module with auto-mocked exports. Provide custom implementations with **`jest.fn()`**, **`mockReturnValue`**, **`mockResolvedValue`**. **`jest.spyOn(obj, 'method')`** wraps one method while leaving the rest real. Reset between tests with **`jest.clearAllMocks()`** or **`jest.resetAllMocks()`**.

> [!example]
> ```js
> jest.mock('./emailService', () => ({
>   sendEmail: jest.fn().mockResolvedValue({ sent: true }),
> }));
>
> import { sendEmail } from './emailService';
> import { registerUser } from './authService';
>
> beforeEach(() => jest.clearAllMocks());
>
> it('sends welcome email on register', async () => {
>   await registerUser({ email: 'a@b.com' });
>   expect(sendEmail).toHaveBeenCalledWith(
>     expect.objectContaining({ to: 'a@b.com', subject: expect.stringContaining('Welcome') })
>   );
> });
>
> // Spy on a single method
> const spy = jest.spyOn(Math, 'random').mockReturnValue(0.5);
> // ... test ...
> spy.mockRestore();
> ```

> [!success] Pros / Cons
> **Module mock:** full isolation — fast, but tightly coupled to import paths.
> **Manual mock (`__mocks__/`):** reusable across tests.
> **Spy:** partial mock — good when most behaviour should stay real.
> **Pitfall:** forgetting to reset mocks → tests pass alone, fail together.

> [!info] Further study
> - [Jest: ES Module Mocking](https://jestjs.io/docs/ecmascript-modules)
> - [Jest: jest.mock()](https://jestjs.io/docs/jest-object#jestmockmodulename-factory-options)

### 9. Testing asynchronous code in Jest

> [!question] Q9
> How do you test asynchronous code (promises, async/await) in Jest?

Jest must **wait** for async work to finish before marking the test pass/fail. Three valid patterns:

1. Mark the test **`async`** and **`await`** the result.
2. **`return`** the promise from the test function.
3. Use **`expect(promise).resolves`** / **`.rejects`** matchers.

If you forget to await/return, the test passes immediately while assertions never run — a classic silent bug.

> [!example]
> ```js
> // async/await (preferred)
> it('fetches user', async () => {
>   const user = await fetchUser(1);
>   expect(user.name).toBe('Ana');
> });
>
> // resolves / rejects
> it('resolves with data', async () => {
>   await expect(fetchUser(1)).resolves.toEqual({ id: 1, name: 'Ana' });
> });
>
> it('throws on 404', async () => {
>   await expect(fetchUser(999)).rejects.toThrow('Not found');
> });
>
> // return promise (older style)
> it('works with return', () => {
>   return fetchUser(1).then(user => expect(user.id).toBe(1));
> });
> ```

> [!success] Pros / Cons
> **`async/await`:** clearest, easiest to add try/catch — preferred.
> **`.resolves/.rejects`:** concise for pass/fail of whole promise.
> **Anti-pattern:** `it('...', () => { fetchUser(1).then(...) })` without return — test exits early.
> **Timers:** combine with fake timers (Q18) for debounce/setTimeout code.

> [!info] Further study
> - [Jest: Async Testing](https://jestjs.io/docs/asynchronous)

### 10. Testing React components with React Testing Library

> [!question] Q10
> How do you test React components with React Testing Library? What is its guiding philosophy?

**React Testing Library (RTL)** renders components into a fake DOM and lets you query elements the way a **user** would — by label text, role, or visible content — not by component instance, internal state, or CSS class names.

**Philosophy:** *"The more your tests resemble the way your software is used, the more confidence they can give you."* Test **behaviour**, not implementation. Refactors that don't change UX should not break tests.

> [!example]
> ```jsx
> import { render, screen } from '@testing-library/react';
> import userEvent from '@testing-library/user-event';
> import { LoginForm } from './LoginForm';
>
> it('shows error when submitting empty form', async () => {
>   const user = userEvent.setup();
>   render(<LoginForm />);
>
>   await user.click(screen.getByRole('button', { name: /sign in/i }));
>
>   expect(screen.getByRole('alert')).toHaveTextContent(/email is required/i);
> });
> ```

> [!success] Pros / Cons
> **Pros:** resilient to refactors; encourages accessible markup (roles, labels); aligns with user expectations.
> **Cons:** async UI needs `findBy`/`waitFor`; over-mocking providers reduces integration confidence.
> **Avoid:** testing `useState` values, private methods, or snapshot-only tests with no behaviour assertions.

> [!info] Further study
> - [Testing Library: Guiding Principles](https://testing-library.com/docs/guiding-principles)
> - [RTL: React Setup](https://testing-library.com/docs/react-testing-library/setup)

### 11. getBy vs queryBy vs findBy in RTL

> [!question] Q11
> What is the difference between `getBy`, `queryBy`, and `findBy` in RTL?

All three locate elements, but differ on **presence** and **async**:

| Query | Returns | Throws if missing? | Async? |
|-------|---------|-------------------|--------|
| `getBy*` | element | **Yes** | No |
| `queryBy*` | element or `null` | No | No |
| `findBy*` | Promise → element | Yes (after timeout) | **Yes** |

Use **`getBy`** when the element **should exist now**. Use **`queryBy`** to assert **absence** (`expect(queryBy...).not.toBeInTheDocument()`). Use **`findBy`** when the element **appears after async work** (fetch, animation, state update). Each has `*AllBy` variants for lists.

> [!example]
> ```jsx
> // Element should be there immediately
> expect(screen.getByRole('heading', { name: /welcome/i })).toBeInTheDocument();
>
> // Element should NOT be there
> expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
>
> // Element appears after fetch
> expect(await screen.findByText(/hello, ana/i)).toBeInTheDocument();
> ```

> [!success] Pros / Cons
> **`getBy`:** fails fast with helpful DOM dump — good default.
> **`queryBy`:** only way to assert non-existence without try/catch.
> **`findBy`:** wraps `waitFor` + `getBy` — default 1000ms timeout; fix flaky tests by awaiting properly, not by raising timeout blindly.

> [!info] Further study
> - [RTL: About Queries](https://testing-library.com/docs/queries/about)

### 12. Testing a custom React hook

> [!question] Q12
> How do you test a custom React hook?

Use **`renderHook`** from `@testing-library/react`. It runs your hook inside a test component and exposes **`result.current`**. Wrap state updates in **`act()`** so React flushes effects. Pass a **`wrapper`** when the hook needs Context providers (Router, QueryClient, Redux).

> [!example]
> ```jsx
> import { renderHook, act } from '@testing-library/react';
> import { useCounter } from './useCounter';
>
> it('increments count', () => {
>   const { result } = renderHook(() => useCounter(0));
>
>   act(() => result.current.increment());
>
>   expect(result.current.count).toBe(1);
> });
>
> // With provider wrapper
> const wrapper = ({ children }) => (
>   <QueryClientProvider client={testQueryClient}>{children}</QueryClientProvider>
> );
> renderHook(() => useUsers(), { wrapper });
> ```

> [!success] Pros / Cons
> **`renderHook`:** direct, fast hook testing — no dummy component boilerplate.
> **Full component test:** more integration confidence — slower, more setup.
> **`act` required:** missing `act` causes "not wrapped in act" warnings and stale assertions.
> **See also:** [[01-react]] for hook patterns under test.

> [!info] Further study
> - [RTL: renderHook API](https://testing-library.com/docs/react-testing-library/api#renderhook)

### 13. Mocking API calls in frontend tests

> [!question] Q13
> How do you mock API calls in frontend tests (msw, jest mocks)?

Two main approaches:

1. **MSW (Mock Service Worker)** — intercepts requests at the **network layer**. Your app code uses real `fetch`/axios; tests control responses via handlers. Closest to production behaviour.
2. **Jest module mocks** — mock `fetch`, axios, or your API module directly. Faster to set up but couples tests to import structure.

> [!example]
> ```js
> // MSW setup
> import { rest } from 'msw';
> import { setupServer } from 'msw/node';
>
> const server = setupServer(
>   rest.get('/api/users/1', (req, res, ctx) =>
>     res(ctx.json({ id: 1, name: 'Ana' }))
>   ),
> );
> beforeAll(() => server.listen());
> afterEach(() => server.resetHandlers());
> afterAll(() => server.close());
>
> // Jest mock alternative
> jest.spyOn(global, 'fetch').mockResolvedValue({
>   ok: true,
>   json: async () => ({ id: 1, name: 'Ana' }),
> });
> ```

> [!success] Pros / Cons
> **MSW:** reusable handlers, works in browser + Node, tests real request code — slightly more setup.
> **Jest mock:** quick for unit tests — brittle if fetch is wrapped in many layers.
> **Avoid:** hitting real APIs in tests (slow, flaky, needs network).

> [!info] Further study
> - [MSW Documentation](https://mswjs.io/docs/)
> - [Testing Library: Example — MSW](https://testing-library.com/docs/react-testing-library/example-intro#full-example)

### 14. What to test vs what not to test

> [!question] Q14
> What should you test vs not test?

**Test:**
- Business logic and calculations (discounts, permissions, validation rules).
- Critical user flows (login, checkout, create resource).
- Edge cases and error paths (empty input, 404, network failure).
- Regressions for bugs you fixed (add a test with the ticket number).

**Don't test (usually):**
- Third-party libraries (Jest/React already test themselves).
- Trivial getters/setters with no logic.
- Implementation details (private methods, internal state shape, exact CSS classes).
- Framework wiring you didn't write (e.g. "does React render?").

> [!example]
> ```js
> // ✅ Test behaviour
> expect(formatPrice(1099)).toBe('$10.99');
> expect(screen.getByRole('button')).toBeDisabled(); // when form invalid
>
> // ❌ Don't test implementation
> expect(component.state.isLoading).toBe(true);     // RTL anti-pattern
> expect(mockDispatch).toHaveBeenCalledWith({ type: 'SET_LOADING' }); // too coupled
> ```

> [!success] Pros / Cons
> **Risk-based testing:** highest ROI on code that breaks often or costs money when wrong.
> **Over-testing:** slows development, brittle tests break on every refactor.
> **Under-testing:** no safety net for refactors or new team members.
> **Interview answer:** "I test behaviour and contracts, not internals — prioritized by business risk."

---

## Advanced

### 15. Unit testing a NestJS service with mocked dependencies

> [!question] Q15
> How do you unit test a NestJS service with mocked dependencies? (your CV: NestJS)

NestJS is built for testability via **dependency injection**. For a **unit test**, create a **`TestingModule`** with `Test.createTestingModule()`, register the service under test as a **provider**, and replace its dependencies with **mock providers** using `{ provide: Token, useValue: mock }`. Call `.compile()`, then `module.get(YourService)` to obtain the instance. Assert the service's **business logic** without hitting a real database or HTTP layer.

> [!example]
> ```ts
> // users.service.ts
> @Injectable()
> export class UsersService {
>   constructor(
>     @InjectModel(User.name) private userModel: Model<UserDocument>,
>     private emailService: EmailService,
>   ) {}
>
>   async create(dto: CreateUserDto) {
>     const existing = await this.userModel.findOne({ email: dto.email });
>     if (existing) throw new ConflictException('Email taken');
>     const user = await this.userModel.create(dto);
>     await this.emailService.sendWelcome(user.email);
>     return user;
>   }
> }
>
> // users.service.spec.ts
> import { Test, TestingModule } from '@nestjs/testing';
> import { getModelToken } from '@nestjs/mongoose';
> import { ConflictException } from '@nestjs/common';
> import { UsersService } from './users.service';
> import { User } from './schemas/user.schema';
> import { EmailService } from '../email/email.service';
>
> describe('UsersService', () => {
>   let service: UsersService;
>
>   const mockUserModel = {
>     findOne: jest.fn(),
>     create: jest.fn(),
>   };
>
>   const mockEmailService = {
>     sendWelcome: jest.fn().mockResolvedValue(undefined),
>   };
>
>   beforeEach(async () => {
>     const module: TestingModule = await Test.createTestingModule({
>       providers: [
>         UsersService,
>         { provide: getModelToken(User.name), useValue: mockUserModel },
>         { provide: EmailService, useValue: mockEmailService },
>       ],
>     }).compile();
>
>     service = module.get<UsersService>(UsersService);
>     jest.clearAllMocks();
>   });
>
>   it('creates user and sends welcome email', async () => {
>     mockUserModel.findOne.mockResolvedValue(null);
>     mockUserModel.create.mockResolvedValue({ id: '1', email: 'a@b.com' });
>
>     const result = await service.create({ email: 'a@b.com', name: 'Ana' });
>
>     expect(result.email).toBe('a@b.com');
>     expect(mockEmailService.sendWelcome).toHaveBeenCalledWith('a@b.com');
>   });
>
>   it('throws ConflictException when email exists', async () => {
>     mockUserModel.findOne.mockResolvedValue({ email: 'a@b.com' });
>
>     await expect(service.create({ email: 'a@b.com', name: 'Ana' }))
>       .rejects.toThrow(ConflictException);
>     expect(mockUserModel.create).not.toHaveBeenCalled();
>   });
> });
> ```

> [!success] Pros / Cons
> **Unit test with mocks:** fast (ms), isolates logic, pinpoints failures — ideal for service methods, guards, pipes.
> **Mock Mongoose model:** stub `findOne`, `create`, `findById` — don't boot MongoDB.
> **Mock other services:** verify interaction (`toHaveBeenCalledWith`) without sending real emails.
> **vs integration test:** unit won't catch schema mismatches or query bugs — add a few integration tests for critical paths.
> **TypeORM variant:** `{ provide: getRepositoryToken(User), useValue: mockRepo }`.

> [!tip] Interview talking points
> Mention `@nestjs/testing`, custom providers, testing pure services vs controllers (controllers often tested via e2e/supertest), and keeping mocks minimal — only stub what the method touches.

> [!info] Further study
> - [NestJS: Testing](https://docs.nestjs.com/fundamentals/testing)
> - [[04-nestjs]] — DI, modules, and service patterns your tests mirror

### 16. E2E tests for a NestJS API

> [!question] Q16
> How do you write e2e tests for a NestJS API (`Test.createTestingModule`, supertest)?

**E2e tests** boot a real (or test-configured) Nest application and send **HTTP requests** with **supertest** — no browser, but the full HTTP pipeline runs: guards, pipes, interceptors, controllers, and usually a **test database**. Use `Test.createTestingModule({ imports: [AppModule] })`, override providers if needed (mock external APIs), call `createNestApplication()`, `app.init()`, then `request(app.getHttpServer()).get/post/...`.

> [!example]
> ```ts
> // test/app.e2e-spec.ts
> import { Test, TestingModule } from '@nestjs/testing';
> import { INestApplication, ValidationPipe } from '@nestjs/common';
> import * as request from 'supertest';
> import { AppModule } from '../src/app.module';
> import { getModelToken } from '@nestjs/mongoose';
> import { User } from '../src/users/schemas/user.schema';
>
> describe('Users API (e2e)', () => {
>   let app: INestApplication;
>
>   const mockUserModel = {
>     find: jest.fn().mockReturnValue({ exec: jest.fn().mockResolvedValue([]) }),
>     create: jest.fn(),
>   };
>
>   beforeAll(async () => {
>     const moduleFixture: TestingModule = await Test.createTestingModule({
>       imports: [AppModule],
>     })
>       .overrideProvider(getModelToken(User.name))
>       .useValue(mockUserModel)
>       .compile();
>
>     app = moduleFixture.createNestApplication();
>     app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
>     await app.init();
>   });
>
>   afterAll(async () => {
>     await app.close();
>   });
>
>   it('GET /users returns 200 and array', () => {
>     return request(app.getHttpServer())
>       .get('/users')
>       .expect(200)
>       .expect('Content-Type', /json/)
>       .expect((res) => {
>         expect(Array.isArray(res.body)).toBe(true);
>       });
>   });
>
>   it('POST /users returns 400 for invalid body', () => {
>     return request(app.getHttpServer())
>       .post('/users')
>       .send({ email: 'not-an-email' })
>       .expect(400);
>   });
>
>   it('POST /users returns 201 for valid user', () => {
>     mockUserModel.create.mockResolvedValue({ id: '1', email: 'a@b.com', name: 'Ana' });
>     return request(app.getHttpServer())
>       .post('/users')
>       .send({ email: 'a@b.com', name: 'Ana' })
>       .expect(201)
>       .expect((res) => {
>         expect(res.body.email).toBe('a@b.com');
>       });
>   });
> });
> ```

> [!example]
> ```json
> // package.json — e2e script
> {
>   "scripts": {
>     "test:e2e": "jest --config ./test/jest-e2e.json"
>   }
> }
> ```

> [!success] Pros / Cons
> **E2e with supertest:** tests routing, validation, guards, serialization — high confidence on API contracts.
> **Test DB / in-memory Mongo:** swap `MONGO_URI` in test env or use `mongodb-memory-server` for real persistence without mocks.
> **Override providers:** mock Stripe/S3/external APIs; keep your code paths real.
> **Cons:** slower than unit tests; need careful setup/teardown; can be flaky if DB state leaks between tests.
> **Best practice:** seed known fixtures, clean DB in `beforeEach`/`afterEach`, close app in `afterAll`.

> [!tip] CV tie-in
> For your affiliate/NestJS APIs, e2e tests should cover auth guards, webhook signature validation, and critical CRUD endpoints — the flows that break deployments.

> [!info] Further study
> - [NestJS: End-to-end testing](https://docs.nestjs.com/fundamentals/testing#end-to-end-testing)
> - [Supertest GitHub](https://github.com/ladjs/supertest)

### 17. Testing code that uses a database

> [!question] Q17
> How do you test code that uses a database (test DB, in-memory, transactions/rollback)?

Database tests need **isolation** — each test starts from a known state and does not depend on another test's rows. Common strategies:

1. **Dedicated test database** — separate `DATABASE_URL`; truncate tables in `beforeEach`.
2. **In-memory database** — e.g. `mongodb-memory-server`, SQLite in-memory for TypeORM — fast, no external deps, great for CI.
3. **Transaction rollback** — wrap each test in a transaction and roll back after (PostgreSQL/MySQL); fast and clean if your ORM supports it.
4. **Docker test container** — real Postgres/Mongo in CI for maximum fidelity.

> [!example]
> ```ts
> // mongodb-memory-server setup (Jest globalSetup or beforeAll)
> import { MongoMemoryServer } from 'mongodb-memory-server';
> import mongoose from 'mongoose';
>
> let mongod: MongoMemoryServer;
>
> beforeAll(async () => {
>   mongod = await MongoMemoryServer.create();
>   await mongoose.connect(mongod.getUri());
> });
>
> afterEach(async () => {
>   const collections = mongoose.connection.collections;
>   for (const key of Object.keys(collections)) {
>     await collections[key].deleteMany({});
>   }
> });
>
> afterAll(async () => {
>   await mongoose.disconnect();
>   await mongod.stop();
> });
> ```

> [!success] Pros / Cons
> **In-memory:** fast, portable CI — slight behaviour differences vs production DB.
> **Test DB + truncate:** closest to prod — slower, needs infra.
> **Transactions:** very fast reset — not all drivers/ORMs support nested transactions.
> **Fixtures/factories:** seed predictable data; avoid hard-coded IDs that collide.

> [!info] Further study
> - [[16-mongodb]] — Mongoose patterns under test
> - [[05-databases]] — transaction and isolation concepts

### 18. Testing time-dependent code with Jest fake timers

> [!question] Q18
> How do you test time-dependent code (timers, debounce) with Jest fake timers?

Real `setTimeout`/`setInterval` make tests slow and flaky. **`jest.useFakeTimers()`** replaces timers with controllable fakes. Advance time with **`jest.advanceTimersByTime(ms)`**, **`jest.runAllTimers()`**, or **`await jest.runAllTimersAsync()`** for async timer chains. Restore with **`jest.useRealTimers()`** in `afterEach`.

> [!example]
> ```js
> jest.useFakeTimers();
>
> it('debounces search by 300ms', () => {
>   const search = jest.fn();
>   const debounced = debounce(search, 300);
>
>   debounced('a');
>   debounced('b');
>   debounced('c');
>   expect(search).not.toHaveBeenCalled();
>
>   jest.advanceTimersByTime(300);
>   expect(search).toHaveBeenCalledTimes(1);
>   expect(search).toHaveBeenCalledWith('c');
> });
>
> afterEach(() => jest.useRealTimers());
> ```

> [!success] Pros / Cons
> **Fake timers:** deterministic, instant — essential for debounce, throttle, polling, `retry` backoff.
> **Cons:** doesn't cover real timer edge cases; `@testing-library/react` may need `{ advanceTimers: jest.advanceTimersByTime }` in userEvent config.
> **Modern Jest:** prefer `jest.runAllTimersAsync()` when promises are involved in timer callbacks.

> [!info] Further study
> - [Jest: Timer Mocks](https://jestjs.io/docs/timer-mocks)
> - [[12-dsa-practical-coding#21 Implement debounce|Debounce implementation]]

### 19. Flaky tests — causes and fixes

> [!question] Q19
> How do you handle flaky tests and what causes them?

A **flaky test** passes and fails non-deterministically without code changes. Common causes:

- **Async races** — assertion runs before promise/state update (missing `await`, `findBy`, or `waitFor`).
- **Shared mutable state** — mocks, DB rows, or global variables leaking between tests.
- **Real time/network/randomness** — `Date.now()`, live APIs, `Math.random()` without mocks.
- **Order dependence** — test B passes only if test A ran first.
- **Animation/layout timing** in browser e2e.

> [!example]
> ```js
> // ❌ Flaky — no await
> it('loads user', () => {
>   render(<UserProfile id={1} />);
>   expect(screen.getByText('Ana')).toBeInTheDocument(); // may fail — still loading
> });
>
> // ✅ Stable
> it('loads user', async () => {
>   render(<UserProfile id={1} />);
>   expect(await screen.findByText('Ana')).toBeInTheDocument();
> });
> ```

> [!success] Pros / Cons
> **Fix strategy:** reproduce locally → identify race/shared state → add proper async waits, isolate setup, mock time/network → quarantine only as last resort.
> **CI:** retry flaky e2e once if needed, but **fix root cause** — retries hide debt.
> **Prevention:** `beforeEach` reset, fake timers, MSW, no `sleep()` in tests.

### 20. Introducing testing culture into an untested codebase

> [!question] Q20
> How would you introduce a testing culture into a codebase that has no tests? (your CV context)

Start **pragmatically**, not with a "100% coverage" mandate that blocks delivery:

1. **Tooling first** — Jest + RTL (frontend) or `@nestjs/testing` + supertest (backend); run in CI on every PR.
2. **Critical paths** — auth, payments, webhooks, data mutations — highest business risk.
3. **New code rule** — every new feature and bug fix includes at least one regression test.
4. **Ratchet, don't gate** — track coverage **trend** upward; optional soft targets (e.g. 50% on new modules).
5. **Lead by example** — write tests in your PRs; review for testability (DI, pure functions).
6. **Boy scout rule** — when you touch a file, add tests for the code you changed.

> [!example]
> ```text
> Week 1: Jest in CI, 10 tests on auth module
> Week 2: MSW for API layer, test top 3 user flows
> Month 2: 40% coverage on services, e2e on deploy pipeline
> Ongoing: no merge without test for bug fixes
> ```

> [!success] Pros / Cons
> **Incremental approach:** delivers value immediately, builds team habit — realistic for CV gap ("no testing listed").
> **Big-bang backfill:** high cost, demoralizing, often abandoned.
> **Interview framing:** "I'd prioritize safety nets on revenue-critical paths and enforce tests on new code — not rewrite everything day one."

### 21. TDD — pros and cons

> [!question] Q21
> What is TDD and what are its pros and cons?

**Test-Driven Development:** write a **failing test first** (Red), write the **minimal code to pass** (Green), then **refactor** (Refactor). Repeat in small cycles. Forces you to design from the caller's perspective before implementation.

> [!example]
> ```js
> // 1. Red — test fails (function doesn't exist yet)
> expect(calculateTax(100, 0.2)).toBe(20);
>
> // 2. Green — minimal implementation
> function calculateTax(amount, rate) { return amount * rate; }
>
> // 3. Refactor — extract, rename, dedupe while tests stay green
> ```

> [!success] Pros / Cons
> **Pros:** clear requirements before coding; high coverage by construction; better design (testable, decoupled modules); confidence to refactor; living documentation.
> **Cons:** slower upfront; learning curve; awkward for exploratory UI/spike work; risk of over-mocking if taken too far on integration boundaries.
> **Pragmatic use:** excellent for pure logic (validators, parsers, pricing rules); many teams use TDD selectively rather than for every line.

> [!info] Further study
> - [Test-Driven Development by Example (Kent Beck)](https://www.oreilly.com/library/view/test-driven-development/0321146530/)

---

## Related notes
- [[01-react]] — component patterns, hooks, and RTL testing philosophy in practice.
- [[04-nestjs]] — DI, modules, guards, and the services you unit/e2e test.
- [[03-nodejs]] — async patterns and test isolation on the server.
- [[12-dsa-practical-coding]] — fake timers for debounce/throttle tests.
- [[16-mongodb]] — Mongoose models and in-memory Mongo for integration tests.

## References & Further Study
- [Jest Documentation](https://jestjs.io/docs/getting-started) — matchers, mocks, async, coverage, fake timers.
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) — queries, user-event, renderHook.
- [MSW — Mock Service Worker](https://mswjs.io/) — network-level API mocking for frontend tests.
- [NestJS Testing Guide](https://docs.nestjs.com/fundamentals/testing) — `Test.createTestingModule`, e2e, overriding providers.
- [Supertest](https://github.com/ladjs/supertest) — HTTP assertions for Nest/Express e2e tests.
- [Martin Fowler: Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) — balancing unit, integration, and e2e.
