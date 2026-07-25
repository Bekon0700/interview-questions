---
title: Behavioral — Answers
topic: behavioral
tags: [interview, behavioral, star]
related: ["[[07-cv-deep-dive]]"]
---

# Behavioral — Answers

> [!abstract] How to use this note
> Each item has the **question**, a plain-English **explanation**, an **example** (spoken answer or STAR bullets), and a **Pros / Cons** box. Numbers match [[questions/08-behavioral|the questions file]]. Keep answers **~1.5–2 minutes** spoken; always end with a result and a learning.

---

## Beginner / warm-up

### 1. Tell me about yourself

> [!question] Q1
> Tell me about yourself. (2-minute pitch)

A tight **present → past → future** pitch: who you are now, how you got here with 2–3 proof points, why this role/company. Not your life story — your **professional narrative**.

> [!example]
> "I'm a software engineer with **3.5+ years at Rokomari**, Bangladesh's largest online bookstore. I started in **frontend** and grew into **full-stack**, building production systems with React, Next.js, Node, and NestJS.
>
> Highlights: I **led our Next.js migration** — cutting page load from **11s to 3s** and Docker images from **2.1GB to 170MB** — and I **bootstrapped backend platforms** from scratch: an **affiliate system** serving **100k+ affiliates** and a **real-time WebChat** stack with Redis and RabbitMQ.
>
> I enjoy **owning systems end to end** and optimizing for performance at scale. I'm now looking for a role where I can take on **deeper system-design and full-stack challenges**, which is why this position excites me."
>
> **Timing:** ~90 seconds at conversational pace. Adjust company name and metrics to match the job description.

> [!success] Pros / Cons
> **Strengths of this framing:** Structured, metric-backed, ends with why-here. **Watch-outs:** Don't recite your entire CV. Don't start with childhood or university unless asked. Mirror 1–2 keywords from the job posting naturally — don't force it.

> [!tip] Practice with a timer. If you hit 3 minutes, cut the second project detail.

See [[07-cv-deep-dive#29. Project you're most proud of]].

---

### 2. Why leave / why this role

> [!question] Q2
> Why are you looking to leave Rokomari / why are you interested in this role?

Be **positive** about your current employer (growth, gratitude, impact), then pivot to what you want **next**: bigger scale, deeper architecture, new domain, tech alignment, mentorship — tied to **this specific role**.

> [!example]
> "I've had a great **3.5 years at Rokomari** — I grew from fresher to software engineer, led a major migration, and shipped systems that moved business metrics. I'm grateful for that runway.
>
> I'm exploring next steps because I want **[specific thing from job description — e.g., larger scale, distributed systems, international team, Golang backend]** — challenges I can't fully get at my current scale. This role stood out because **[1–2 specifics: their product, stack, engineering blog post, team structure]**, and it aligns with where I want to grow over the next few years."
>
> **Never say:** "I'm bored," "pay is low," "management is bad," or anything that sounds like running away.

> [!success] Pros / Cons
> **Strengths:** Forward-looking, respectful, researched. **Watch-outs:** Badmouthing current employer is an instant red flag. Vague "I want new challenges" without specifics sounds unfocused. Research the company — mention something real.

---

### 3. Strengths and weaknesses

> [!question] Q3
> What are your strengths and weaknesses?

One **evidenced strength** (with a CV story) and one **genuine weakness** with **active improvement** — not a humble-brag disguised as weakness.

> [!example]
> **Strength — Ownership + performance focus:**
> "When I see a bottleneck, I drive it to resolution. On the affiliate platform, a report query took **37 seconds** — I used `explain()`, added the right compound index, and got it to **~1 second**. I don't stop at 'it works'; I measure and optimize."
>
> **Weakness — Optimization timing (example):**
> "I sometimes dive into optimization before confirming impact. On an early project I spent days micro-optimizing a path that wasn't on the critical user journey. I've since adopted a rule: **profile first, prioritize by user/business impact**, and time-box optimization. I also push myself to write more automated tests — I've been adding integration tests on new NestJS modules."
>
> **Other honest weaknesses (pick one real):** delegating too long before asking for help; over-documenting; public speaking — always pair with what you're doing about it.

> [!success] Pros / Cons
> **Strengths:** Strength is provable; weakness shows self-awareness + growth. **Watch-outs:** Fake weaknesses ("I work too hard") insult the interviewer's intelligence. Don't pick a weakness that's a core job requirement ("I'm bad at communication" for a lead role).

See [[07-cv-deep-dive#10. Query optimization 37s → 1s]].

---

### 4. Where in 3–5 years

> [!question] Q4
> Where do you see yourself in 3-5 years?

Show **ambition + realistic fit** with the company: senior/lead engineer owning larger systems, mentoring, deeper architecture — aligned with their ladder, not "your job in five years."

> [!example]
> "In **3–5 years** I want to be a **senior engineer** who owns critical systems end to end — architecture, reliability, and mentoring juniors. I'm especially interested in deepening **system design and backend scale** — my CV mentions Golang, and I'm building toward that alongside my Node/NestJS production experience.
>
> I'd love to grow **with a team like yours** where I can take on increasing scope — from shipping features to shaping technical direction — and help others ramp the way my seniors helped me at Rokomari."

> [!success] Pros / Cons
> **Strengths:** Shows retention intent and growth path. **Watch-outs:** Don't say "your manager role" unless interviewing for a track toward it. Don't say "founding my own startup" unless true. Tie answer to **this company's** domain.

See [[08-behavioral#17. Interest in Golang]].

---

## Core behavioral

### 5. Ownership from scratch (affiliate platform)

> [!question] Q5
> Tell me about a time you took ownership of a project from scratch.

Amazon LP: **Ownership**, **Bias for Action**. Story: zero-to-one affiliate platform — ambiguity, full-stack delivery, production impact.

> [!example]
> **Situation:** Rokomari wanted an affiliate program; **no existing platform**, unclear specs beyond "track referrals and pay commissions."
>
> **Task:** Build the system **end to end** — backend, auth, attribution, async processing, affiliate dashboard.
>
> **Action:** I bootstrapped a **NestJS** backend and frontend. Designed MongoDB schema around report queries, built **JWT access/refresh auth**, referral link generation, order attribution, **RabbitMQ** workers for commission calculation, Redis caching for dashboards, and optimized a **37s → 1s** report query.
>
> **Result:** **100k+ affiliates**, **600+ daily orders** handled reliably, contributed to **~13% sales growth** (business-attributed). I learned to thrive in ambiguity — define MVP, ship, iterate on production feedback.
>
> *Spoken:* "Nobody handed me a spec — I talked to stakeholders, drew the architecture on a whiteboard, and owned it through production incidents and performance tuning."

> [!success] Pros / Cons
> **Strengths:** Clear Ownership LP mapping; quantified impact. **Watch-outs:** Say "I" for your actions, "we" for company-wide metrics. Don't claim you did design/frontend/backend alone unless true — clarify team support.

See [[07-cv-deep-dive#4. Affiliate platform architecture]], [[07-cv-deep-dive#7. 13% sales increase attribution]].

---

### 6. Led a project (Next.js migration)

> [!question] Q6
> Tell me about a time you led a project or initiative.

Amazon LP: **Ownership**, **Deliver Results**, **Earn Trust**. Leadership ≠ title — coordinating strategy, people, and risk on the Next.js migration.

> [!example]
> **Situation:** Legacy **jQuery frontend** — slow (**11s** loads), hard to maintain, blocking product velocity on a high-traffic e-commerce site.
>
> **Task:** **Lead migration to Next.js** without breaking the live site or SEO/revenue.
>
> **Action:** I defined the **strangler-fig** rollout (page-by-page rewrites), chose rendering strategy per page type (ISR for catalog), coordinated **UI/UX** on a shared component library and **backend** on API contracts, set up **load testing** and performance budgets, drove **Docker optimization** (2.1GB→170MB) and lazy-loading work, and ran a per-page SEO/analytics parity checklist.
>
> **Result:** Page load **11s → 3s**, Docker **2.1GB → 170MB**, smooth transition with **no major outages**. I learned leadership is **coordination and de-risking**, not being the loudest coder.
>
> *Spoken:* "I didn't lead because of a title — I led because someone had to own the migration plan, and I made the rollout boring and reversible."

> [!success] Pros / Cons
> **Strengths:** Shows lead behaviors without manager title. **Watch-outs:** Credit the team for parallel work. If a manager co-signed decisions, say so honestly if asked.

See [[07-cv-deep-dive#17. What leading the migration involved]], [[02-nextjs]].

---

### 7. Conflict or disagreement

> [!question] Q7
> Describe a time you had a conflict or disagreement with a teammate or manager. How did you resolve it?

Amazon LP: **Have Backbone; Disagree and Commit**, **Earn Trust**. Focus on **respect**, **data**, and **outcome** — not who "won."

> [!example]
> **Situation:** During the Next.js migration, a teammate wanted to **big-bang rewrite** all checkout pages in one release; I advocated **incremental strangler-fig** migration.
>
> **Task:** Align on an approach that minimized revenue risk without blocking progress.
>
> **Action:** I listened to their concerns — they were tired of maintaining two stacks and wanted a clean cut. I prepared a **comparison doc**: rollback time, blast radius, SEO risk, and a timeline showing incremental could finish checkout in 6 weeks with weekly shippable slices. We reviewed with our lead; we agreed on incremental with a **hard cutoff date** for legacy checkout decommission.
>
> **Result:** Checkout migrated on schedule with **zero downtime**; relationship stayed strong because I addressed their frustration (dual maintenance) with a clear end date, not dismissal.
>
> *Spoken:* "I disagreed on approach, not on goals. We both wanted the legacy stack gone — I showed data on risk, they got a committed sunset date."

> [!success] Pros / Cons
> **Strengths:** Mature conflict resolution; no villain. **Watch-outs:** **Use a real story** — replace placeholders with your actual disagreement. Never say "they were wrong and stupid." Don't pick a conflict where you were clearly the problem without showing growth.

> [!tip] If you lack a migration example, use: SSR vs CSR for a page, MongoDB embed vs reference, or analytics implementation approach.

---

### 8. Learned a new technology quickly

> [!question] Q8
> Tell me about a time you had to learn a new technology quickly.

Amazon LP: **Learn and Be Curious**, **Bias for Action**. Frontend engineer taking on NestJS/Redis/RabbitMQ is a strong CV-aligned story.

> [!example]
> **Situation:** I was hired as a **frontend** engineer; Rokomari needed the **affiliate backend** built, and there was no dedicated backend engineer available.
>
> **Task:** Learn **NestJS, MongoDB, and RabbitMQ** and ship a production affiliate platform within a aggressive timeline.
>
> **Action:** I spent the first week on **NestJS docs + a toy CRUD API**, read Mongoose schema design guides, built a small **queue prototype** (publish/consume/retry), paired with a senior on auth patterns, and shipped MVP in slices — auth first, then attribution, then async commissions.
>
> **Result:** Production affiliate platform serving **100k+ affiliates**; I became the go-to engineer for NestJS services (WebChat, AdServer). I learned structured docs + small prototypes beat tutorial hopping.
>
> *Spoken:* "I didn't wait to feel ready — I built the smallest production slice, deployed it, and expanded. Production feedback taught faster than any course."

> [!success] Pros / Cons
> **Strengths:** Demonstrates ramp speed with shipped outcome. **Watch-outs:** Don't claim expert-level depth in two weeks — show **effective** learning. Mention resources honestly (docs, pairing, not "I watched one YouTube video").

See [[04-nestjs]].

---

### 9. Missed deadline or mistake

> [!question] Q9
> Describe a time you missed a deadline or made a mistake. What did you do?

Amazon LP: **Ownership**, **Insist on the Highest Standards**. Pick a **real, non-catastrophic** mistake; show **accountability**, **communication**, **fix**, and **prevention**.

> [!example]
> **Situation:** I deployed an affiliate report change that added a new filter field but **forgot a compound index** — reports worked in staging (small dataset) but **timed out in production**.
>
> **Task:** Restore reports quickly and prevent recurrence.
>
> **Action:** I **owned it immediately** — told my lead within 30 minutes, rolled back the deploy, ran `explain()` on production query, added the missing index, redeployed during low traffic, and added an **index review step** to our PR template for any query change.
>
> **Result:** Reports restored in ~2 hours; no data loss. The PR checklist caught two similar issues later. I learned **staging volume must mirror production** for performance changes.
>
> *Spoken:* "I didn't blame QA or staging — I missed the index, I communicated early, fixed it, and changed our process."

> [!success] Pros / Cons
> **Strengths:** Honesty + systematic fix. **Watch-outs:** Don't pick a fireable offense. Don't blame others. Don't pick something that reveals incompetence in a core skill without a clear learning arc.

See [[07-cv-deep-dive#10. Query optimization 37s → 1s]].

---

### 10. Improved performance or optimized something

> [!question] Q10
> Tell me about a time you improved performance or optimized something.

Amazon LP: **Dive Deep**, **Deliver Results**. Two strong CV stories: **37s→1s query** or **11s→3s page load**. Pick one; keep the other as backup.

> [!example]
> **Story A — MongoDB query (37s → 1s):**
> **S:** Affiliate dashboard reports unusable at peak — **37s** query time, DB CPU spiking.
> **T:** Get reports under **2s** without hardware throw-money-at-it.
> **A:** Ran `explain('executionStats')` → `COLLSCAN` + in-memory `SORT`. Added ESR compound index, projection for covered fields, moved `$match` early in aggregation.
> **R:** **~37s → ~1s**; affiliate satisfaction improved; added explain() to our performance playbook.
>
> **Story B — Page load (11s → 3s):**
> **S:** Next.js migration target; Lighthouse showed JS + images as top costs.
> **T:** Cut load time on key templates by **60%+**.
> **A:** `next/image`, code splitting, ISR for catalog, `next/font`, lazy-loaded below-fold components; re-measured each sprint.
> **R:** **11s → 3s** on homepage/PLP; improved LCP and conversion inputs.
>
> *Spoken (A):* "I profiled before optimizing — explain() told me exactly which index was missing. One index, not a new server."

> [!success] Pros / Cons
> **Strengths:** Metrics + method = credible. **Watch-outs:** Always state **how you measured**. Don't say "I made it faster" without numbers.

See [[07-cv-deep-dive#10. Query optimization 37s → 1s]], [[07-cv-deep-dive#21. Page load 11s → 3s]], [[16-mongodb#11 37s → 1s walkthrough]].

---

### 11. Collaborated across teams

> [!question] Q11
> Describe a time you had to collaborate across teams.

Amazon LP: **Earn Trust**, **Customer Obsession**. Cross-functional work: migration with design/backend, or analytics accuracy with management/data.

> [!example]
> **Situation:** Next.js migration required **UI/UX**, **backend**, and **QA** alignment — design wanted pixel-perfect parity, backend had API constraints, QA needed regression coverage across hundreds of pages.
>
> **Task:** Ship migrated pages without visual regressions, broken APIs, or SEO loss.
>
> **Action:** I set up a **shared component library** with design tokens, weekly **migration sync** (30 min, fixed agenda: blockers, next pages, sign-offs), a **parity checklist** (screenshot diff, meta tags, GA events) with QA, and a **Slack channel** for fast backend API questions. I translated technical trade-offs ("ISR vs SSR for this page") into business terms for product.
>
> **Result:** Migrated **X pages** (fill in real count) over **Y months** with no major visual or analytics regressions. Design and QA adopted the checklist for future projects.
>
> *Spoken:* "My job was to be the glue — same vocabulary, shared checklist, no surprise deploys."

> [!success] Pros / Cons
> **Strengths:** Shows communication across disciplines. **Watch-outs:** Don't make other teams the villain. Quantify scope if possible (pages migrated, teams involved).

Alternative story: GA/Pixel accuracy work with **management/marketing** — see [[07-cv-deep-dive#2. Improved GA and Facebook Pixel accuracy]].

---

### 12. Received difficult feedback

> [!question] Q12
> Tell me about a time you received difficult feedback. How did you respond?

Amazon LP: **Learn and Be Curious**, **Earn Trust**. Show you **listened**, **reflected**, **changed behavior**, and **improved**.

> [!example]
> **Situation:** In a code review, my lead said my NestJS PRs were **"technically correct but hard to maintain"** — fat services, inconsistent error handling, no DTOs.
>
> **Task:** Improve code quality without slowing delivery.
>
> **Action:** I didn't get defensive. I asked for **examples of good PRs** in our repo, studied our senior's AdServer module structure, refactored my next feature to use **DTOs + thin controllers + shared exception filter**, and asked for early design review on the following ticket.
>
> **Result:** Review cycles shortened; my AdServer refactor PR was cited as a template. I now **request design feedback before the big implementation** on non-trivial work.
>
> *Spoken:* "The feedback stung because I thought 'works' was enough. It wasn't — maintainability is part of the job. I changed how I start tickets, not just how I finish them."

> [!success] Pros / Cons
> **Strengths:** Growth mindset with concrete behavior change. **Watch-outs:** Don't pick feedback you've clearly ignored. Don't say "they were wrong but I complied." Show genuine improvement.

---

### 13. Decision with incomplete information

> [!question] Q13
> Describe a time you had to make a decision with incomplete information.

Amazon LP: **Bias for Action**, **Are Right, A Lot**. Show **pragmatism**: gather what you can, choose reversible/low-risk path, validate fast, adjust.

> [!example]
> **Situation:** We needed to choose **ISR revalidate interval** for product pages before Black Friday — no historical data on optimal staleness vs server load.
>
> **Task:** Pick a strategy that balanced **fresh prices/stock** with **CDN cache hit rate** under unknown peak traffic.
>
> **Action:** I benchmarked **60s vs 300s vs on-demand-only** in staging under load test, checked with product on acceptable price staleness (**5 min OK for non-flash-sale items**), shipped **60s ISR with on-demand revalidation webhook** for price updates, and added monitoring on revalidation rate and origin load.
>
> **Result:** Peak event held; origin load stayed flat; we tuned to **120s** post-event based on data. I learned **decide with best available evidence, instrument, iterate**.
>
> *Spoken:* "Perfect data would've taken weeks we didn't have. I time-boxed research, picked a reversible default, and measured from day one."

> [!success] Pros / Cons
> **Strengths:** Shows judgment under uncertainty. **Watch-outs:** Don't pick a reckless gamble that worked by luck. Emphasize **reversibility** and **monitoring**.

See [[02-nextjs#3 Rendering strategies]], [[02-nextjs#32 Caching/revalidation]].

---

### 14. Production incident under pressure

> [!question] Q14
> Tell me about a time you handled a production incident under pressure.

Amazon LP: **Dive Deep**, **Ownership**, **Customer Obsession**. STAR with calm triage, communication, root cause, fix, and prevention.

> [!example]
> **Situation:** **Affiliate reports down** during business hours — DB CPU at 100%, support getting affiliate complaints.
>
> **Task:** Restore service without taking checkout offline.
>
> **Action:** I joined the war room, **communicated status every 30 min** to stakeholders, checked slow query log and `explain()` — found a new report query doing **COLLSCAN**. Short-term: **killed long-running queries**, enabled **query timeout**, routed traffic to cached summary endpoint. Root fix: **added compound index**, deployed in maintenance window. Post-incident: added **alert on totalDocsExamined ratio** and index review in PR checklist.
>
> **Result:** Reports restored in **~2 hours**; root fix deployed next day; no checkout impact. Zero recurrence in following quarter.
>
> *Spoken:* "Under pressure I default to: stabilize → communicate → root cause → prevent. Users and affiliates needed honesty more than instant perfection."

> [!success] Pros / Cons
> **Strengths:** Structured incident response. **Watch-outs:** Don't panic-narrate. Don't claim you solo-fixed a major outage if a team responded. Include **communication** explicitly.

See [[07-cv-deep-dive#30. Hardest production bug or incident]].

---

### 15. Disagreed with a technical approach

> [!question] Q15
> Tell me about a time you disagreed with a technical approach. How did you handle it?

Amazon LP: **Have Backbone; Disagree and Commit**. Similar to Q7 but emphasize **advocacy with evidence** and **committing to the team decision**.

> [!example]
> **Situation:** For WebChat message persistence, a colleague proposed **storing all messages only in Redis** for speed; I advocated **MongoDB primary + Redis cache**.
>
> **Task:** Choose a storage approach before building.
>
> **Action:** I built a **quick prototype** showing Redis memory cost at projected message volume (~**X GB/month** — fill in if known) and data-loss risk on restart. I documented **MongoDB + Redis cache-aside** latency (still sub-100ms for hot paths) in a one-pager. We debated in tech review; the team chose **MongoDB + Redis**.
>
> **Result:** System shipped with durable history and fast reads. When the decision went my way, I helped my colleague implement the cache layer — **no grudges**. When decisions go the other way, I **disagree and commit** fully.
>
> *Spoken:* "I advocate with prototypes and numbers, not opinions. Once we decide, I execute like it was my idea."

> [!success] Pros / Cons
> **Strengths:** Evidence-based disagreement + team player. **Watch-outs:** Also prepare a story where **you lost** the argument and still delivered — that's the real "disagree and commit" test.

See [[07-cv-deep-dive#24. Real-time WebChat architecture]], [[07-cv-deep-dive#26. Redis caching in WebChat]].

---

## Growth / career

### 16. Fresher to engineer journey

> [!question] Q16
> You started as a fresher and grew into an engineer. Tell me about that journey.

Narrate the **arc**: frontend hire → proactive backend ownership → leading migration → increasing scope and impact. Mindset shift from tasks to outcomes.

> [!example]
> "I joined Rokomari as a **fresher frontend developer** working on jQuery — cleanup, analytics, performance. When the company needed an **affiliate platform**, I volunteered to learn **NestJS** and built it end to end — that was my inflection point from 'implement tickets' to 'own systems.'
>
> I then built **WebChat** and **AdServer** backends, optimized a **37s database query**, and eventually **led the Next.js migration** — the biggest frontend initiative in the company.
>
> Over **3.5 years**, my scope grew from single-page fixes to **architecture, async processing, and cross-team leadership**. The biggest shift was learning that **engineering success is measured in production outcomes**, not merged PRs."
>
> **Optional timeline:** Year 1 — jQuery/frontend. Year 2 — affiliate + NestJS. Year 3 — WebChat/Ads + migration lead.

> [!success] Pros / Cons
> **Strengths:** Clear progression narrative. **Watch-outs:** Don't oversell — if promotions were informal, say "grew into engineer responsibilities" rather than "I was promoted twice." Match your actual title history.

See [[07-cv-deep-dive#31. Fresher to Software Engineer — biggest learning]].

---

### 17. Interest in Golang

> [!question] Q17
> Your CV mentions interest in Golang-based full-stack development. Why, and how are you pursuing it?

Be **genuine**: why Go appeals for backend/systems work, and **concrete** learning steps — not "I read about it once."

> [!example]
> "I've shipped production backends in **Node/NestJS** and hit its limits on **CPU-heavy concurrency** and **binary deployment simplicity**. **Golang** appeals for its **goroutine model**, **fast compile times**, **single static binary**, and growing ecosystem for **APIs and infrastructure tools**.
>
> I'm pursuing it by **[pick what's true]:** building a small **REST API side project** with chi/gin, reading *The Go Programming Language*, completing **Tour of Go** and exercism, and comparing patterns from my NestJS services (auth, queues) in Go. I want my next role to let me **apply Go in production**, not just tutorials."
>
> *Spoken:* "I'm not jumping on a hype train — I want a tool that fits systems problems I've already faced at Rokomari, at larger scale."

> [!success] Pros / Cons
**Strengths:** Connects Go to your real experience. **Watch-outs:** Don't claim production Go experience you don't have. Don't badmouth Node — frame Go as **expanding** your toolkit.

---

### 18. Keeping skills up to date

> [!question] Q18
> How do you keep your skills up to date?

Concrete **habits with evidence** — not "I read blogs sometimes."

> [!example]
> - **At work:** Adopt new patterns in production — e.g., **App Router**, **ISR + tag revalidation**, NestJS micro-patterns from our AdServer refactors.
> - **Side projects:** Small tools to try new stacks (Go API, Docker compose setups).
> - **Reading:** Framework release notes (Next.js blog), **MongoDB performance docs**, engineering blogs (Highscalability, Martin Fowler).
> - **Community:** Local meetups or online talks; occasionally write internal docs sharing what I learned (e.g., explain() guide for the team).
>
> *Spoken:* "I learn best by shipping — a side project or a scoped experiment at work beats passive reading. Release notes are my first stop when Next.js or NestJS ships a major version."

> [!success] Pros / Cons
> **Strengths:** Specific, credible habits. **Watch-outs:** Vague "I follow trends" sounds shallow. Have one **recent example** (e.g., "I migrated a personal project to App Router last month").

---

### 19. Team and engineering culture

> [!question] Q19
> What kind of team/engineering culture do you thrive in?

Describe what **genuinely** helps you — then **align** with the company's stated values (research their engineering blog, job posting, Glassdoor themes).

> [!example]
> "I thrive where:
> - **Ownership** is real — engineers own services in production, not throw code over the wall.
> - **Code review and knowledge sharing** are normal — I grow from feedback and give it constructively.
> - Decisions are **data-driven** — we profile before optimizing, measure after shipping.
> - **Cross-functional collaboration** is expected — I work well with design and product, as I did on the migration.
> - There's **room to learn** — seniors mentor, juniors are trusted with real scope.
>
> From what I've read about **[Company]**, your emphasis on **[specific value — e.g., customer obsession, operational excellence]** matches how I already work at Rokomari."

> [!success] Pros / Cons
> **Strengths:** Authentic + tailored to employer. **Watch-outs:** Don't describe a culture that contradicts the role (e.g., "I hate process" for a regulated fintech). Research the company — generic answers feel unprepared.

---

### 20. Questions for the interviewer

> [!question] Q20
> Do you have any questions for us? (always prepare 2-3)

Always have **2–3 thoughtful questions** ready. Shows curiosity and helps you evaluate the role. Avoid salary/perks as your **only** questions in early rounds.

> [!example]
> **Strong questions (pick 2–3):**
> 1. "What does the team's **tech stack and architecture** look like today, and what are the **biggest technical challenges** in the next 6–12 months?"
> 2. "How does the team approach **code quality, testing, and deployment** — what does a typical PR cycle look like?"
> 3. "What would **success in this role** look like in the **first 6 months**?"
> 4. "How do you support **engineer growth** — mentorship, conference budget, internal talks?"
> 5. "What's the **on-call/incident** culture like — how often, how is it shared?"
> 6. (For manager) "What's your philosophy on **balancing feature delivery vs tech debt**?"
>
> **Follow-up technique:** Reference something from the interview: "You mentioned migrating to Kubernetes — what's driving that timeline?"
>
> **Avoid as only questions:** "What's the salary?" (save for HR/recruiter), "How much vacation?", "Can I work fully remote?" (unless critical — frame positively).

> [!success] Pros / Cons
> **Strengths:** Signals serious interest and helps you decide. **Watch-outs:** Don't ask anything answered on their website. Don't say "no questions." Don't lead with compensation in a technical round.

> [!tip] Write 5 questions in your notes; use 2–3 based on time and what was already covered.

---

## Related notes

- [[07-cv-deep-dive]] — technical depth behind every CV story (metrics, architecture, STAR bullets).
- [[02-nextjs]] — migration leadership and performance stories (Q6, Q10).
- [[04-nestjs]] — affiliate, WebChat, AdServer ownership stories (Q5, Q8).
- [[16-mongodb]] — 37s→1s optimization story (Q10, Q14).
- [[06-system-design]] — architecture framing for "tell me about a project" questions.
- [[11-web-performance]] — 11s→3s page load story (Q10).
- [[questions/08-behavioral]] — question list only (no answers).

## References & Further Study

- [STAR method (MIT CAPD)](https://capd.mit.edu/resources/the-star-method-for-behavioral-interviews/) — structure every behavioral answer: Situation, Task, Action, Result (+ Learning).
- [Amazon Leadership Principles](https://www.amazon.jobs/content/en/our-workplace/leadership-principles) — map stories to Ownership, Dive Deep, Bias for Action, Learn and Be Curious, Deliver Results, Have Backbone; Disagree and Commit.
- [Amazon behavioral interview guide (Glassdoor/community summaries)](https://www.amazon.jobs/en/landing_pages/interviewing-at-amazon) — official Amazon interviewing resources.
- *Cracking the Coding Interview* (McDowell) — behavioral prep chapter and "tell me about yourself" framework.
- Practice aloud with a **timer** (1.5–2 min per answer) — spoken pacing differs from reading.
