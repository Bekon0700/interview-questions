# This vault

An Obsidian vault with two distinct, non-overlapping purposes. Read [[CONTEXT.md]] for the full vocabulary before doing any nontrivial work here — it defines Topic, Category, and every generated-file type precisely, and stays current as new terms crystallize.

## The two areas

**`questions/` + `answers/`** — fast interview-prep review. Pre-written, polished Q&A for cramming under time pressure. Treat as passive reference: read it, edit it, answer questions about it directly. No tutor behavior applies here, ever.

**`learning/`** — active, from-scratch learning, AI-assisted. Structured as `learning/<category>/<topic>/`. This is where the rules below matter.

## Tutor/Socratic behavior is opt-in only

Never withhold a direct answer or turn a plain question into a quiz on your own initiative — not even for `learning/` content. That behavior is scoped strictly to the `/review` skill (and the category-confirmation step inside `/learn-topic`). If the user asks something to unblock themselves while building, mid-task, or in passing, just answer it. The whole point of scoping this to skills is that the user decides when they want to be quizzed by typing `/review` — don't second-guess that by nudging outside of it.

## `learning/` rules

- A Topic never lives directly under `learning/` — it always belongs to a Category (`learning/<category>/<topic>/`).
- **Always confirm the Category with the user first** — existing or brand-new — before creating any folder. Never auto-assign, even when a fit seems obvious (e.g. "Kafka → message-broker"). The taxonomy is being built collaboratively; guessing defeats that.
- A Topic folder's five files (overview, `all-features.md`, `pros-cons.md`, `implementation.md`, `recall.md`) are AI-drafted for speed, *except* `implementation.md`'s problem set, which the user solves unaided — never write solutions into it, only the one worked example plus problems stated with expected output only.
- A Category folder gets its root file (named after the category) immediately; `comparison.md` and the Category-level `recall.md` only once it has 2+ Topics.
- **Bridging Topics** (subject matter spanning 2+ Categories, e.g. Redis) live under one primary Category only. Their overview file must wikilink explicitly into the other Category's root file — this is a deliberate step, not automatic. There is no separate mind-map file; Obsidian's Graph View is the mind map, and it only reflects relationships that were actually linked.
- Review scheduling is a simple per-Topic (and per-Category, once comparable) ladder — 1d → 3d → 7d → 14d → 30d, advancing on a solid `/review` pass and resetting to 1d on a poor one — tracked in the table in [[learning/README.md]]. Not per-question SRS; don't build that.
- `recall.md` is what makes something reviewable — a Topic or Category isn't due for `/review` until its `recall.md` exists.

## Working in this repo generally

- Keep [[CONTEXT.md]] up to date the moment a new term crystallizes — don't batch it up. It's a glossary only; never put implementation detail, task status, or scratch content in it.
- Follow existing frontmatter/callout conventions in `answers/` and `questions/` notes ([[README.md]] has the legend) when touching that area.
- Prefer wikilinks (`[[path/to/note]]`) over plain paths for anything meant to be navigable inside Obsidian.
