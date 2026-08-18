---
title: Interview Question Bank — Map of Content
topic: moc
tags: [interview, fullstack, moc, obsidian]
---

# Full Stack Interview Question Bank (3+ YOE)

> [!abstract] What this vault is
> A CV-tailored interview prep vault for a **3+ years full stack** role (React, Next.js, Node.js, NestJS, system design), plus topics top companies commonly test. Open this note as your **Map of Content (MOC)** — then jump into topic notes via wikilinks.

## How this vault is organized

| Folder | Purpose |
|--------|---------|
| `questions/` | Practice view — numbered questions only, **Beginner → Intermediate → Advanced** |
| `answers/` | Study view — self-contained Obsidian Q&A notes (question embedded + elaborate answer) |
| `learning/` | Separate tracker for things you want to learn / are currently learning — see [[learning/README\|Learning Tracker MOC]] |

Matching numbers stay aligned: e.g. Q7 in `questions/01-react.md` ↔ Q7 in [[01-react]].

Questions follow top-company style (Google, Amazon, Meta, Microsoft, Netflix, Uber, Stripe, Shopify). CV-linked answers are flagged so you can adapt them to what you actually did.

## Callout conventions (every answer note)

| Callout | Meaning |
|---------|---------|
| `> [!question]` | The interview question |
| `> [!example]` | Worked code or scenario |
| `> [!success] Pros / Cons` | Advantages / disadvantages (or Strengths / Watch-outs when there is no real trade-off) |
| `> [!tip]` | Interview tip or CV tie-in |
| `> [!info]` | Further study link for that concept |
| `> [!warning]` | CV template — replace with your real details |

Each answer note also has YAML **frontmatter** (`title`, `topic`, `tags`, `related`) and ends with **Related notes** + **References & Further Study**.

## Topics (wikilinks)

| # | Topic | Practice | Study (answers) |
|---|-------|----------|-----------------|
| 00 | JavaScript & TypeScript | [[questions/00-javascript]] | [[answers/00-javascript]] |
| 01 | React | [[questions/01-react]] | [[answers/01-react]] |
| 02 | Next.js (incl. migration) | [[questions/02-nextjs]] | [[answers/02-nextjs]] |
| 03 | Node.js | [[questions/03-nodejs]] | [[answers/03-nodejs]] |
| 04 | NestJS | [[questions/04-nestjs]] | [[answers/04-nestjs]] |
| 05 | Databases (general + Redis) | [[questions/05-databases]] | [[answers/05-databases]] |
| 06 | System Design (CV-tailored) | [[questions/06-system-design]] | [[answers/06-system-design]] |
| 07 | CV Deep Dive | [[questions/07-cv-deep-dive]] | [[answers/07-cv-deep-dive]] |
| 08 | Behavioral | [[questions/08-behavioral]] | [[answers/08-behavioral]] |
| 09 | Web Security | [[questions/09-security]] | [[answers/09-security]] |
| 10 | Networking & HTTP | [[questions/10-networking-http]] | [[answers/10-networking-http]] |
| 11 | Web Performance | [[questions/11-web-performance]] | [[answers/11-web-performance]] |
| 12 | DSA & Practical Coding | [[questions/12-dsa-practical-coding]] | [[answers/12-dsa-practical-coding]] |
| 13 | Testing | [[questions/13-testing]] | [[answers/13-testing]] |
| 14 | Docker & Deployment | [[questions/14-docker-deployment]] | [[answers/14-docker-deployment]] |
| 15 | SQL | [[questions/15-sql]] | [[answers/15-sql]] |
| 16 | MongoDB (deep dive) | [[questions/16-mongodb]] | [[answers/16-mongodb]] |

> [!note] Obsidian tip
> Open this folder as an Obsidian vault. Prefer path-prefixed links (`[[answers/00-javascript]]`) because the same filenames also exist under `questions/`.

## Suggested study order

Work top-down. Inside each note: Beginner → Advanced.

### Phase 1 — Core strengths
1. [[answers/00-javascript]] — foundation; tested everywhere (includes dedicated Promises track P1–P22)
2. [[answers/01-react]] — core frontend
3. [[answers/02-nextjs]] — migration story (Docker size, page load)
4. [[answers/03-nodejs]] + [[answers/04-nestjs]] — backend stack
5. [[answers/16-mongodb]] — major CV strength (37s → 1s)

### Phase 2 — Design & resume
6. [[answers/05-databases]] — general DB + Redis
7. [[answers/06-system-design]] — Affiliate / WebChat / Ads / e-commerce designs
8. [[answers/07-cv-deep-dive]] — rehearse every metric out loud

### Phase 3 — Top-company staples
9. [[answers/09-security]] — likely (JWT on CV)
10. [[answers/10-networking-http]] — classic openers
11. [[answers/11-web-performance]] — theory behind 11s → 3s
12. [[answers/12-dsa-practical-coding]] — coding round

### Phase 4 — Fill gaps
13. [[answers/13-testing]] — not on CV; prepare a clear stance
14. [[answers/14-docker-deployment]] — 2.1GB → 170MB story
15. [[answers/15-sql]] — PostgreSQL / MySQL on CV

### Always in parallel
- [[answers/08-behavioral]] — prepare 5–6 STAR stories from your CV

## How to practice

1. Open a `questions/` note and answer out loud **before** checking the answer.
2. Open the matching answer note; read the Pros / Cons and Further study links.
3. For CV-linked answers (`> [!warning]`), replace templates with your real numbers and decisions.
4. Revisit weak topics until you can explain the **why**, not only the **what**.
5. Use Obsidian graph / backlinks to jump between related notes.

## CV-tailored answers (important)

Answers about migration metrics, the 37s→1s query, affiliate / WebChat / Ads architecture, token invalidation, and Promise-based request queueing are **strong templates** based on your resume. Verify each against what you actually built before the interview — interviewers will drill in.

## Related notes

- Start here for study: [[answers/00-javascript]], [[answers/01-react]], [[answers/02-nextjs]], [[answers/04-nestjs]], [[answers/16-mongodb]], [[answers/07-cv-deep-dive]]
