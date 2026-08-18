---
name: learn-topic
description: Scaffold and AI-generate a new Topic (or extend an existing one) under learning/<category>/<topic>/ in this Obsidian vault — confirms Category placement with the user first, then drafts the overview, all-features, pros-cons, implementation, and recall files.
disable-model-invocation: true
---

Read [[../../CONTEXT.md]] and [[../../learning/README.md]] first — they define every term and convention used below (Topic, Category, Bridging Topic, the five Topic files, the callout legend).

## 1. Get the subject

If not given as an argument, ask what the user wants to learn.

## 2. Resolve the Category — always confirm, never auto-assign

List existing Categories (subfolders of `learning/`). Then:

- If one existing Category is a clean, unambiguous fit, still propose it and wait for confirmation before creating anything — don't skip the round-trip just because it seems obvious.
- If the subject plausibly spans 2+ Categories (a Bridging Topic — e.g. Redis: caching + message-broker), say so explicitly and ask the user to pick the *primary* Category (where the folder physically lives) — the others become deliberate wikilinks later, not separate folders.
- If no existing Category fits, propose a new Category name and wait for confirmation before creating the folder.

Do not proceed to generation until the Category is confirmed.

## 3. Create the Category (if new)

`learning/<category-slug>/<category-slug>.md` — AI-drafted: history, why the category was necessary, what problem it solves, alternative names it's known by, and a linked list of Topics (starts with just this one). Use `[!abstract]` for the opening blurb.

## 4. Create the Topic folder

`learning/<category-slug>/<topic-slug>/`, five files:

- **`<topic-slug>.md`** — overview: backstory, what/why/how. `[!abstract]` opening blurb.
- **`all-features.md`** — every notable feature as its own subsection, each with a `[!tip]` "why this was necessary" note and an `[!example]`.
- **`pros-cons.md`** — `[!success] Pros` and `[!warning] Cons` callouts, each entry with a concrete example, not just a label.
- **`implementation.md`** — exactly one fully worked example first (solved, explained), then a graduated problem set beginner → advanced. Every problem states its **expected final output only** — never write or imply the solution. The user builds every problem unaided in Obsidian/their own environment.
- **`recall.md`** — a handful of `[!question]` retrieval prompts (short-answer, plus at least one "explain this to a beginner") pulled from this Topic's own `<topic-slug>.md` / `all-features.md` / `pros-cons.md`. This file is what makes the Topic reviewable — don't skip it.

Frontmatter for every generated file: `title`, `category`, `topic`, `tags`, `related` — follow the style already used in `answers/*.md` (see [[../../README.md]] for the legend this vault otherwise uses).

## 5. Bridging Topic wikilink

If this Topic spans a second Category (per step 2), add an explicit wikilink from `<topic-slug>.md` into that other Category's root file, with a one-line note on *why* they're related (e.g. "Also usable as a pub/sub broker — see [[../message-broker/message-broker]]"). This is the only thing that makes Obsidian's Graph View show the relationship — it is not automatic, so don't skip it.

## 6. Category-level files, once 2+ Topics exist

If this Category now has 2 or more Topics, create or update:

- **`comparison.md`** — feature-by-feature comparison table across all Topics in the Category.
- **`recall.md`** — `[!question]` comparison-style prompts (e.g. "when would you choose X over Y, and why?"), pulled from `comparison.md`.

If the Category still has only 1 Topic, skip both — don't create them empty.

## 7. Update the tracker

In `learning/README.md`:
- Add/update the row for this Topic in the **Categories & Topics** table: Category, Topic, Status (`currently-learning`), Last Reviewed (blank), Next Review (today + 1 day, since a fresh Topic should get its first `/review` pass soon).
- If it was previously listed in **Want to Learn**, remove that row.

## Do not

- Do not write solutions or solution code into `implementation.md`.
- Do not auto-pick a Category without explicit user confirmation, even for "obvious" cases.
- Do not slip into quiz/Socratic mode during generation — this skill drafts content; `/review` is what tests the user on it.
