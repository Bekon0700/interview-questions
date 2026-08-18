---
title: Learning Tracker — Map of Content
topic: moc
tags: [learning, moc, obsidian]
---

# Learning Tracker

> [!abstract] What this area is
> Separate from the [[../README|interview question bank]]. This is where subjects get learned from scratch, AI-assisted — not interview-format Q&A. Vocabulary (Topic, Category, etc.) is defined in [[../CONTEXT|the vault glossary]]; read that first if a term below is unclear.

## Quick start (forgot how this works? start here)

| If you... | Do this |
|---|---|
| Want to start learning something new | Type `/learn-topic <subject>` (e.g. `/learn-topic Redis`). It'll ask you to confirm the Category, then generate the 5 files. `implementation.md`'s problems are yours to solve afterward — nothing gets pre-filled there. |
| Just have an idea, not ready to start it | Add a row to the **Want to Learn** table below (Idea / Why / Likely Category). No folder gets created yet. |
| It's time to study | Type `/review`. It picks whatever's due, asks `recall.md` questions one at a time — answer before it tells you anything — then updates the schedule below. |
| Want your `implementation.md` graded | Ask for it directly (e.g. "grade my Redis implementation"). It's never automatic. |
| Just want a quick, direct answer | Ask normally. Outside `/learn-topic` and `/review`, nothing here quizzes you or withholds an answer. |
| Forgot what a term means (Topic vs. Category vs. Bridging Topic, etc.) | Check [[../CONTEXT\|the glossary]]. |

## How this works

- Every Topic lives inside a Category: `learning/<category>/<topic>/`. A Topic never sits directly under `learning/`.
- Start a new Topic with `/learn-topic` — it will always ask you to confirm which Category it belongs to (existing or new) before creating anything.
- A Topic folder holds five files: the overview (named after the topic), `all-features.md`, `pros-cons.md`, `implementation.md`, `recall.md`.
- A Category folder holds its own root file (named after the category, with history/why/alternative names/topic list), plus — once it has 2+ Topics — `comparison.md` and a Category-level `recall.md`.
- Run `/review` to study. It pulls whatever's due (see schedule below), interleaves across Topics/Categories once you have more than one, and can grade your `implementation.md` solutions on request.
- If a Topic genuinely bridges two Categories (e.g. Redis: caching + message-broker), it lives under one primary Category and wikilinks into the other Category's root file — that link is what makes Obsidian's Graph View show the relationship. There's no separate mind-map file, so this only works if the link is actually added; don't skip it when generating a bridging Topic.

## Callout conventions (generated files)

| Callout | Used in |
|---------|---------|
| `> [!abstract]` | Short intro blurb, top of overview/category-root files |
| `> [!example]` | Worked examples and illustrative scenarios |
| `> [!success] Pros` | `pros-cons.md` |
| `> [!warning] Cons` | `pros-cons.md` |
| `> [!tip]` | "Why this was necessary" note in `all-features.md` |
| `> [!question]` | Retrieval prompts in `recall.md` |

## Review schedule

Per-topic ladder, tracked in the table below: **1d → 3d → 7d → 14d → 30d**. A solid `/review` pass advances a Topic (or Category) to the next rung; a poor one resets it to 1d.

## Categories & Topics

| Category | Topic | Status | Last Reviewed | Next Review |
|----------|-------|--------|----------------|--------------|
| | | | | |

## Want to Learn (backlog — no folder yet)

| Idea | Why | Likely Category |
|------|-----|------------------|
| | | |

## Related notes

- [[../README|Interview Question Bank MOC]]
- [[../CONTEXT|Vault Glossary]]
