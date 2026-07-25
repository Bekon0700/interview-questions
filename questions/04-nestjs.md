# NestJS — Questions

Core backend framework from your CV (Affiliate, AdServer, WebChat).

---

## Beginner

1. What is NestJS and what problems does it solve over plain Express?
2. What are modules, controllers, and providers/services in Nest?
3. What is dependency injection and how does Nest's IoC container work?
4. What are decorators in Nest (`@Controller`, `@Injectable`, `@Get`, `@Body`, `@Param`)?
5. What is a DTO and how is validation done (`class-validator`, `ValidationPipe`)?
6. How does routing work in Nest? How do you handle route params, query params, and body?
7. What is the difference between a provider's default (singleton) scope and request scope?

## Intermediate

8. Explain the Nest request lifecycle: middleware → guards → interceptors → pipes → controller → interceptors → exception filters. (commonly asked in Nest interviews)
9. What is the difference between guards, interceptors, pipes, and middleware? Give a use case for each.
10. How do you implement authentication and authorization (Passport, JWT strategy, guards)? (your CV: JWT access/refresh)
11. What are interceptors good for (logging, transformation, caching, timeouts)?
12. What are exception filters and how do you build a global one?
13. How do you configure the app across environments (`@nestjs/config`, validation schema)?
14. How do custom decorators work (e.g., a `@CurrentUser()` param decorator)?
15. How do you connect to MongoDB with Mongoose in Nest (`@nestjs/mongoose`, schemas, models)?
16. How do you structure a Nest project for a growing codebase (feature modules, shared modules)?
17. How do you handle configuration of global pipes/guards/interceptors/filters?

## Advanced

18. How do Nest microservices work? Explain the RabbitMQ transporter and message vs event patterns. (your CV: RabbitMQ)
19. How would you design a queue-based background job system in Nest (Bull/BullMQ + Redis)? (your CV: Redis, RabbitMQ)
20. How do you implement caching in Nest (CacheModule, Redis store, cache interceptor)? (your CV: Redis caching)
21. How do you handle webhooks reliably in Nest (idempotency, signature verification, retries)? (your CV: webhooks, Crisp APIs)
22. How do you test Nest apps (unit tests with mocked providers, e2e with `Test.createTestingModule`)?
23. How do you implement a refresh-token rotation and invalidation strategy in Nest? (your CV: token invalidation)
24. How do you handle database transactions in Nest with Mongoose sessions?
25. How would you design a real-time chat backend in Nest (WebSockets/gateways, Redis pub/sub for scaling, RabbitMQ for delivery)? (your CV: WebChat)
26. How do you prevent circular dependencies between modules, and how does `forwardRef` work?
27. How do you implement rate limiting and throttling in Nest (`@nestjs/throttler`)?
