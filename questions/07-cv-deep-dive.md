# CV Deep Dive — Questions

These are the questions an interviewer will ask directly from your resume. Every metric and claim on your CV is fair game — be ready to explain HOW, not just WHAT. Answers use the STAR format (Situation, Task, Action, Result).

---

## Rokomari Frontend (jQuery era)

1. You removed 1,800 lines of unoptimized code. What was wrong with it, how did you identify it, and how did you verify nothing broke?
2. How did you improve the accuracy of data sent to Google Analytics and Facebook Pixel? What was inaccurate before?
3. What specific frontend performance optimizations did you make in the jQuery codebase?

## Rokomari Affiliate (NestJS)

4. Walk me through the affiliate platform architecture end to end. What were the main components?
5. You built the auth flow with access/refresh tokens. Explain the full token lifecycle and how you handle invalidation.
6. Explain the JavaScript Promise-based request queueing you built for concurrent API requests. Why was it needed and how does it work?
7. How did the affiliate program contribute to a 13% sales increase? How do you attribute that?
8. How did you handle 600+ daily orders reliably? What happens if a step fails?
9. How did you design the database structure and business workflows?
10. You optimized a query from 37s to 1s using explain() and indexes. Walk through exactly what you did. (also in MongoDB file)
11. How did you achieve "low server load"? What did you measure and change?
12. How does RabbitMQ fit into the affiliate system?

## Rokomari Ads (NestJS)

13. Explain the three backend features: automated ad placement, budget-based ad prioritization, and performance analytics tracking.
14. How does budget-based prioritization work? How do you prevent overspending a budget?
15. How did you track performance analytics at scale without slowing down ad delivery?
16. How did you refactor the REST APIs for maintainability?

## Rokomari Next.js Migration

17. You LED the migration. What did leading involve — planning, coordination, decisions?
18. Why migrate to Next.js at all? What was the business/technical justification?
19. How did you migrate incrementally without breaking the live site?
20. Explain the Docker optimization from 2.1GB to 170MB in detail. (also in Next.js/Docker files)
21. Explain how you reduced page load from 11s to 3s. (also in Next.js file)
22. How did you conduct load testing and what did you find?
23. How did you enhance the lazy loading system?

## Rokomari WebChat (NestJS)

24. Walk me through the real-time WebChat architecture (NestJS, Redis, RabbitMQ, Crisp).
25. How does the automated bot reply system work?
26. How did you use Redis caching here and what improved?
27. How did you handle webhooks and ensure reliability?
28. How did you achieve 78% customer satisfaction? How is that measured and what did you change?

## General / cross-cutting

29. Which project are you most proud of and why?
30. Tell me about the hardest bug or incident you faced in production and how you resolved it.
31. You went from fresher to Software Engineer. What was the biggest thing you learned?
32. If you could redesign one of these systems today, what would you do differently?
