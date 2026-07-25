---
title: NestJS — Answers
topic: nestjs
tags: [interview, fullstack, nestjs, nodejs, backend]
related: ["[[03-nodejs]]", "[[16-mongodb]]", "[[06-system-design]]", "[[07-cv-deep-dive]]"]
---

# NestJS — Answers

> [!abstract] How to use this note
> Each item has the **question**, a plain-English **explanation**, an **example**, and a **Pros / Cons** box. Numbers match [[questions/04-nestjs|the questions file]].
> [!warning] CV-linked answers
> Several items map to your Affiliate/AdServer/WebChat work — adapt them to your real implementation.

---

## Beginner

### 1. What Nest solves

> [!question] Q1
> What is NestJS and what problems does it solve over plain Express?

NestJS is a **structured** Node framework (built on top of Express/Fastify). Plain Express gives you total freedom but no conventions, so large teams end up with inconsistent, hard-to-test code. Nest adds an Angular-like architecture: modules, dependency injection, TypeScript-first design, decorators, and clear patterns (guards, pipes, interceptors, filters).

> [!success] Pros / Cons
> **Pros:** consistent structure, testable (DI), scales with team size, batteries included. **Cons:** more boilerplate and concepts than raw Express, a learning curve, and some "magic" (decorators/DI) to understand. Great for growing backends like your affiliate platform.

### 2. Modules / controllers / providers

> [!question] Q2
> What are modules, controllers, and providers/services in Nest?

A **module** groups a feature and wires up its parts. A **controller** handles incoming HTTP requests and returns responses (the routing layer). A **provider/service** (`@Injectable`) holds the business logic and is injected where needed.

> [!example]
> ```ts
> @Module({ controllers: [UsersController], providers: [UsersService] })
> export class UsersModule {}
> ```

> [!success] Pros / Cons
> **Pros:** clear separation (HTTP vs logic), easy to test services in isolation, modular growth. **Con:** more files/ceremony for small apps.

### 3. Dependency injection & IoC

> [!question] Q3
> What is dependency injection and how does Nest's IoC container work?

Instead of a class creating its own dependencies, Nest's **IoC container** creates and **injects** them based on the constructor types. You just declare what you need.

> [!example]
> ```ts
> @Injectable()
> export class UsersService {
>   constructor(private readonly db: DbService) {} // Nest injects DbService
> }
> ```

> [!success] Pros / Cons
> **Pros:** loose coupling, trivial mocking in tests, shared singletons by default (efficient). **Con:** the "how does this get created?" indirection can confuse newcomers; circular dependencies need care ([[#26 Circular dependencies]]).

### 4. Decorators

> [!question] Q4
> What are decorators in Nest (`@Controller`, `@Injectable`, `@Get`, `@Body`, `@Param`)?

Decorators are annotations (starting with `@`) that attach **metadata** telling Nest how to treat a class/method/parameter: `@Controller('users')` = a routing class, `@Injectable()` = a provider, `@Get(':id')` = an HTTP route, `@Body()`/`@Param()`/`@Query()` = extract request parts.

> [!example]
> ```ts
> @Controller('users')
> export class UsersController {
>   @Get(':id') findOne(@Param('id') id: string) { return this.svc.find(id); }
> }
> ```

> [!success] Pros / Cons
> **Pros:** declarative, readable, colocated configuration. **Con:** relies on `reflect-metadata` and `experimentalDecorators` (a bit of build setup); heavy decoration can obscure control flow.

### 5. DTOs & validation

> [!question] Q5
> What is a DTO and how is validation done (`class-validator`, `ValidationPipe`)?

A **DTO** (Data Transfer Object) is a class describing the shape of incoming data, annotated with `class-validator` rules. A global **`ValidationPipe`** checks every request body against its DTO and rejects invalid data with a `400` automatically.

> [!example]
> ```ts
> export class CreateUserDto {
>   @IsEmail() email: string;
>   @MinLength(8) password: string;
> }
> ```

> [!success] Pros / Cons
> **Pros:** centralised, declarative validation; controllers stay clean; can strip unknown fields (whitelist). **Con:** duplicates types as classes; decorators-based validation is runtime (not compile-time). Security angle: [[09-security#4 SQL/NoSQL injection]].

### 6. Routing & params

> [!question] Q6
> How does routing work in Nest? How do you handle route params, query params, and body?

The controller's base path + the method decorator define the route. Extract parts with parameter decorators: `@Param('id')` (path), `@Query()` (query string), `@Body()` (request body), `@Headers()`.

> [!example]
> ```ts
> @Get() list(@Query('page') page: number) {}
> @Post() create(@Body() dto: CreateUserDto) {}
> ```

> [!success] Pros / Cons
> **Pros:** typed, declarative handler arguments; no manual `req.params` digging. **Con:** query/params arrive as strings — use pipes (`ParseIntPipe`) or DTOs to coerce/validate.

### 7. Provider scopes

> [!question] Q7
> What is the difference between a provider's default (singleton) scope and request scope?

**Singleton** (default): one shared instance for the whole app — efficient, stateless. **Request scope**: a new instance per request (for request-specific state), at a performance cost. **Transient**: a new instance per consumer.

> [!success] Pros / Cons
> **Singleton (prefer):** fast, cached, no per-request overhead. Con: cannot hold per-request state. **Request scope:** isolates per-request data. Con: slower (created every request), and it "bubbles up" making dependents request-scoped too.

---

## Intermediate

### 8. Request lifecycle

> [!question] Q8
> Explain the Nest request lifecycle: middleware → guards → interceptors → pipes → controller → interceptors → exception filters. (commonly asked in Nest interviews)

The order a request flows through is: **middleware** → **guards** → **interceptors (before)** → **pipes** → **route handler** → **interceptors (after)** → **exception filters** (if anything throws).

> [!example]
> Because guards run **before** pipes, authentication is checked before body validation — you reject unauthenticated requests without even validating their payload.

> [!success] Pros / Cons
> **Pros:** predictable, well-defined extension points for cross-cutting concerns. **Con:** you must remember the order (a common interview question) — putting logic in the wrong layer causes subtle bugs.

### 9. Guards vs interceptors vs pipes vs middleware

> [!question] Q9
> What is the difference between guards, interceptors, pipes, and middleware? Give a use case for each.

- **Middleware** — runs first; generic request processing (logging, CORS, body parsing).
- **Guards** — decide if a request may proceed (auth/authorisation); return true/false. E.g. `JwtAuthGuard`.
- **Pipes** — transform/validate handler inputs. E.g. `ValidationPipe`, `ParseIntPipe`.
- **Interceptors** — wrap the handler to add behaviour before/after (logging, response shaping, caching, timeouts).

> [!success] Pros / Cons
> **Pros:** each concern has a dedicated, reusable place → clean controllers. **Con:** four similar-sounding concepts are easy to confuse; choosing the wrong one (e.g. auth in middleware vs a guard) loses Nest's context/DI benefits.

### 10. Auth (Passport + JWT)

> [!question] Q10
> How do you implement authentication and authorization (Passport, JWT strategy, guards)? (your CV: JWT access/refresh)

Use `@nestjs/passport` with a **JWT strategy** that validates the token and attaches the user to the request; protect routes with a `JwtAuthGuard`. On login, issue a **short-lived access token** (Bearer header) and a **longer-lived refresh token**; call a refresh endpoint when the access token expires. Add roles with a `RolesGuard` + `@Roles()` decorator.

> [!tip] CV tie-in
> This is the access/refresh lifecycle from your affiliate platform; the revocation part is [[#23 Refresh-token rotation & invalidation]].

> [!success] Pros / Cons
> **Pros:** stateless auth (no session store), scales horizontally, standard tooling. **Cons:** JWTs cannot be revoked before expiry (needs short expiry + refresh/blocklist); token storage must resist XSS/CSRF ([[09-security#8 JWT storage]]).

### 11. Interceptors' uses

> [!question] Q11
> What are interceptors good for (logging, transformation, caching, timeouts)?

Interceptors wrap a handler using RxJS so you can run logic **before** and **after** it: request/response logging and timing, transforming/normalising responses (`{ data }` envelope), caching, setting timeouts, and mapping errors.

> [!example]
> ```ts
> intercept(ctx, next) {
>   const now = Date.now();
>   return next.handle().pipe(tap(() => console.log(`${Date.now() - now}ms`)));
> }
> ```

> [!success] Pros / Cons
> **Pros:** DRY cross-cutting behaviour applied globally/per-route. **Con:** RxJS knowledge required; overusing them hides where response shaping happens.

### 12. Exception filters

> [!question] Q12
> What are exception filters and how do you build a global one?

Exception filters catch thrown errors and shape the HTTP response. A **global** filter gives consistent error formatting, logging, and status mapping across the whole app — turning both `HttpException`s and unexpected errors into one standard JSON shape.

> [!example]
> ```ts
> @Catch()
> export class AllExceptionsFilter implements ExceptionFilter {
>   catch(err, host) { /* log + send { statusCode, message } */ }
> }
> ```

> [!success] Pros / Cons
> **Pros:** consistent, safe error responses (no leaked stack traces), central logging. **Con:** a catch-all can hide specific error types if not structured; order/scoping matters. Compare Express: [[03-nodejs#8 Express error handling]].

### 13. Config

> [!question] Q13
> How do you configure the app across environments (`@nestjs/config`, validation schema)?

`@nestjs/config` loads env vars into an injectable `ConfigService`, ideally with a **validation schema** (Joi/zod) that fails fast at startup if required vars are missing or malformed. Register it as a global module and inject typed config instead of reading `process.env` everywhere.

> [!success] Pros / Cons
> **Pros:** typed, validated, centralised config; fails fast on misconfig; testable. **Con:** schema upkeep as config grows; secrets still need a proper store in production ([[09-security#12 Secrets handling]]).

### 14. Custom decorators

> [!question] Q14
> How do custom decorators work (e.g., a `@CurrentUser()` param decorator)?

Use `createParamDecorator` to pull data out of the request into a clean handler argument — e.g. `@CurrentUser()` reads `request.user` (set by the auth guard). Compose multiple decorators with `applyDecorators`.

> [!example]
> ```ts
> export const CurrentUser = createParamDecorator((_, ctx) =>
>   ctx.switchToHttp().getRequest().user);
> // usage: findMe(@CurrentUser() user: User) {}
> ```

> [!success] Pros / Cons
> **Pros:** removes repetitive `req.user` access, self-documenting, reusable. **Con:** hides where the value comes from (must know the guard set it); over-abstraction can confuse.

### 15. Mongoose in Nest

> [!question] Q15
> How do you connect to MongoDB with Mongoose in Nest (`@nestjs/mongoose`, schemas, models)?

Import `MongooseModule.forRoot(uri)` once, then `MongooseModule.forFeature([{ name, schema }])` per feature module. Define schemas with `@Schema()`/`@Prop()`, inject the model with `@InjectModel(Name.name)`, and query it in the service.

> [!example]
> ```ts
> @Schema() export class User { @Prop() email: string; }
> // in service:
> constructor(@InjectModel(User.name) private model: Model<User>) {}
> ```

> [!tip] CV tie-in
> This is the setup behind your affiliate/ads/webchat data layers. Deep MongoDB detail: [[16-mongodb]].

> [!success] Pros / Cons
> **Pros:** typed, DI-friendly DB access, schema validation, middleware/hooks. **Con:** Mongoose adds overhead vs the native driver ([[16-mongodb#18 .lean()]]); another abstraction to learn.

### 16. Project structure

> [!question] Q16
> How do you structure a Nest project for a growing codebase (feature modules, shared modules)?

Organise **by feature** (`users/`, `orders/`, `affiliates/`), each with its controller, service, DTOs, schemas, and module. Put cross-cutting code (guards, interceptors, config, DB) in a `common`/`core`/`shared` module, and export only what other modules need.

> [!success] Pros / Cons
> **Pros:** scales with the codebase, easy to locate/change a feature, clear boundaries. **Con:** more folders/modules; requires discipline to avoid a giant `shared` dumping ground. Frontend parallel: [[01-react#36 Architecting a large React/Next.js frontend]].

### 17. Global enhancers

> [!question] Q17
> How do you handle configuration of global pipes/guards/interceptors/filters?

Bind them either imperatively in `main.ts` (`app.useGlobalPipes(...)`) or as **providers** using the `APP_PIPE`/`APP_GUARD`/`APP_INTERCEPTOR`/`APP_FILTER` tokens (which allows dependency injection into them).

> [!success] Pros / Cons
> **Provider tokens (prefer when DI is needed):** the enhancer can inject services (config, logger). **Imperative in main.ts:** simplest for dependency-free enhancers. Con: cannot inject dependencies that way.

---

## Advanced

### 18. Nest microservices + RabbitMQ

> [!question] Q18
> How do Nest microservices work? Explain the RabbitMQ transporter and message vs event patterns. (your CV: RabbitMQ)

Nest supports **transporters** (RabbitMQ, Kafka, Redis, gRPC, TCP). Over RabbitMQ, services talk via a broker using two patterns: **message pattern** (`@MessagePattern`) is request/response (RPC-style, the caller awaits a reply), and **event pattern** (`@EventPattern`) is fire-and-forget (publish, no reply).

> [!tip] CV tie-in
> Use **events** for decoupled notifications ("order placed") — the model behind your affiliate/ads event flows.

> [!success] Pros / Cons
> **Messages:** get a result back, RPC-style. Con: caller waits (coupling). **Events:** fully decoupled, resilient, absorb spikes. Con: eventual consistency, no direct reply. Broker basics: [[06-system-design#7 Message queue]].

### 19. Background jobs (BullMQ + Redis)

> [!question] Q19
> How would you design a queue-based background job system in Nest (Bull/BullMQ + Redis)? (your CV: Redis, RabbitMQ)

Use `@nestjs/bull`/BullMQ backed by **Redis**. Producers add jobs to named queues; `@Processor` classes consume them asynchronously with concurrency control, retries with backoff, delayed/scheduled jobs, and dead-letter handling for failures.

> [!tip] CV tie-in
> This offloads slow work (emails, reports, commission calculation) from the request path — key to handling order volume with low server load.

> [!success] Pros / Cons
> **Pros:** fast APIs (heavy work moved off the request), retries/resilience, scheduling. **Cons:** adds Redis + worker processes to operate; jobs must be **idempotent** (they can retry) — see [[06-system-design#25 Exactly-once vs at-least-once]].

### 20. Caching

> [!question] Q20
> How do you implement caching in Nest (CacheModule, Redis store, cache interceptor)? (your CV: Redis caching)

Register `CacheModule` with a **Redis** store so the cache is shared across instances. Use the `CacheInterceptor` to auto-cache GET responses, or inject the cache manager to cache specific expensive computations with explicit TTLs and invalidation on writes.

> [!tip] CV tie-in
> This is the Redis caching that reduced load and sped up data retrieval in your WebChat/affiliate reporting.

> [!success] Pros / Cons
> **Pros:** big latency and DB-load reductions, shared across the fleet (Redis). **Con:** cache invalidation is hard (stale data risk); adds a Redis dependency. Strategies/pitfalls: [[05-databases#17 Caching strategies]].

### 21. Reliable webhooks

> [!question] Q21
> How do you handle webhooks reliably in Nest (idempotency, signature verification, retries)? (your CV: webhooks, Crisp APIs)

**Verify** the provider's signature (HMAC of the raw body with a shared secret) before processing. Ensure **idempotency** by storing processed event IDs and ignoring duplicates (providers retry). Respond **fast** with `2xx` and offload heavy work to a queue. Handle out-of-order/retried deliveries.

> [!tip] CV tie-in
> This is exactly your Crisp/WebChat webhook handling.

> [!success] Pros / Cons
> **Pros:** no duplicate processing, resilient to retries, secure against spoofing. **Con:** requires storing event IDs and careful signature handling (raw body needed); adds a queue for heavy work. Related: [[06-system-design#16 Webhook delivery/consumption]] and [[00-javascript#P20 Retry with exponential backoff]].

### 22. Testing

> [!question] Q22
> How do you test Nest apps (unit tests with mocked providers, e2e with `Test.createTestingModule`)?

**Unit:** build a `Test.createTestingModule({ providers: [...] })`, provide **mocked** dependencies, and assert service logic in isolation. **E2E:** build the full app + `app.init()` and hit endpoints with `supertest`, mocking external services and using a test DB.

> [!example]
> ```ts
> const module = await Test.createTestingModule({
>   providers: [UsersService, { provide: getModelToken(User.name), useValue: mockModel }],
> }).compile();
> ```

> [!success] Pros / Cons
> **Pros:** DI makes mocking trivial; fast unit tests + confident e2e. **Con:** e2e with a real DB is slower and needs setup/teardown; over-mocking unit tests can miss integration bugs. More: [[13-testing#15 Unit testing a NestJS service]].

### 23. Refresh-token rotation & invalidation

> [!question] Q23
> How do you implement a refresh-token rotation and invalidation strategy in Nest? (your CV: token invalidation)

Issue a **new** refresh token on each refresh (rotation) and invalidate the old one. Store a **hash** of the current valid refresh token per user/session so you can revoke it (logout, password change, theft). If an already-rotated token is reused, treat it as compromise and revoke the whole session family. Access tokens stay short-lived; the refresh token is the revocable anchor.

> [!success] Pros / Cons
> **Pros:** stateless fast auth **plus** the ability to revoke; detects token theft (reuse). **Con:** requires server-side storage of token hashes and careful session-family logic. This is your affiliate auth invalidation; system view: [[06-system-design#13 JWT auth system]].

### 24. Transactions with Mongoose sessions

> [!question] Q24
> How do you handle database transactions in Nest with Mongoose sessions?

Start a session (`connection.startSession()`), run operations inside `session.withTransaction(async () => {...})` passing `{ session }` to each query, and commit/abort atomically. Requires a MongoDB replica set.

> [!example]
> ```ts
> await session.withTransaction(async () => {
>   await this.accounts.updateOne({ _id }, { $inc: { balance: -amt } }, { session });
>   await this.orders.create([{ ... }], { session });
> });
> ```

> [!success] Pros / Cons
> **Pros:** all-or-nothing consistency across documents (e.g. balance + order). **Cons:** performance overhead, needs a replica set, and a 60s limit — good schema design (embedding) often avoids the need ([[16-mongodb#13 Embed vs reference]]).

### 25. Real-time chat backend

> [!question] Q25
> How would you design a real-time chat backend in Nest (WebSockets/gateways, Redis pub/sub for scaling, RabbitMQ for delivery)? (your CV: WebChat)

Use a Nest **WebSocket gateway** (`@WebSocketGateway`, Socket.IO) for client connections. To scale across instances, use a **Redis pub/sub adapter** so a message received on one node reaches clients on other nodes. Persist messages in MongoDB, use **RabbitMQ** for reliable async tasks (bot replies, notifications, Crisp sync), and Redis for presence/caching. Add auth on connect, rooms per conversation, and delivery acknowledgements.

> [!tip] CV tie-in
> This matches your Redis + RabbitMQ + Crisp WebChat design achieving high responsiveness. Full design: [[06-system-design#20 Real-time chat / support]].

> [!success] Pros / Cons
> **Pros:** scales horizontally (Redis backplane), reliable delivery (queues), low latency (WebSockets). **Cons:** WebSocket scaling/sticky sessions are complex; presence/ordering/reconnection need careful handling.

### 26. Circular dependencies

> [!question] Q26
> How do you prevent circular dependencies between modules, and how does `forwardRef` work?

A circular dependency is when two modules/providers depend on each other, so Nest cannot decide the creation order. `forwardRef(() => OtherModule)` on **both** sides defers resolution to break the cycle — but the better fix is to **remove** the cycle by extracting shared logic into a third module or decoupling with events.

> [!success] Pros / Cons
> **`forwardRef`:** quick unblock. Con: a code smell that hides bad structure and can break in subtle ways. **Refactoring (prefer):** cleaner architecture. Con: more upfront work.

### 27. Rate limiting

> [!question] Q27
> How do you implement rate limiting and throttling in Nest (`@nestjs/throttler`)?

Use `@nestjs/throttler` with a `ThrottlerGuard` to limit requests per time window globally or per-route (`@Throttle`). For multi-instance deployments, back it with a **Redis** storage provider so limits hold across all nodes.

> [!success] Pros / Cons
> **Pros:** easy protection against abuse/brute-force, per-route control. **Con:** in-memory storage does not work across instances (needs Redis); app-level limiting complements but does not replace gateway/CDN limits. Design: [[06-system-design#12 Design a rate limiter]].

---

## Related notes
- [[03-nodejs]] — the runtime and event loop Nest sits on.
- [[16-mongodb]] — the database behind your Nest services.
- [[06-system-design]] — queues, caching, rate limiting, chat, auth at scale.
- [[07-cv-deep-dive]] — telling your Affiliate/WebChat/Ads stories.

## References & Further Study
- [NestJS docs](https://docs.nestjs.com/) — official, thorough, example-rich.
- [NestJS: Guards, Interceptors, Pipes, Exception filters](https://docs.nestjs.com/guards) — the request lifecycle pieces.
- [NestJS: Microservices](https://docs.nestjs.com/microservices/basics) and [Queues (Bull)](https://docs.nestjs.com/techniques/queues).
- [Passport JWT strategy](https://docs.nestjs.com/recipes/passport) — authentication.
