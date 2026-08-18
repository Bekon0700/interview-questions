---
name: review
description: Run a spaced-repetition retrieval-practice session over learning/ Topics and Categories that are due — asks recall.md questions before revealing answers, interleaves across what's due, and can grade implementation.md solutions on request.
disable-model-invocation: true
---

Read [[../../CONTEXT.md]] and [[../../learning/README.md]] first — they define the terms and the review ladder used below.

This is the one place in this vault where Socratic behavior belongs. Outside this skill, never withhold an answer or quiz the user unprompted — but inside it, that's the whole point: never just answer a `recall.md` question yourself.

## 1. Find what's due

Read the **Categories & Topics** table in `learning/README.md`. A Topic or Category is due if `Next Review` is today or earlier, or blank (never reviewed). If nothing's due, say so and ask whether the user wants to review something anyway rather than picking for them.

## 2. Build the session

Collect `recall.md` questions from every due Topic and Category. If 2 or more items are due, interleave — don't exhaust one Topic's questions before moving to the next; mix them.

## 3. Run it — one question at a time

For each question:
1. Ask it. Wait for the user's answer. Do not reveal or hint at the answer first.
2. Compare their answer against the source theory file(s) it was pulled from.
3. Correct gaps or imprecision plainly — don't just say "close enough." If their answer conflicts with something stated in the Topic/Category files, point at the specific claim.
4. Move to the next question.

## 4. Score and update the ladder

For each Topic/Category covered this session, judge the pass as solid or poor based on how its questions went (use your judgment — a couple of shaky answers among many solid ones is still solid; multiple wrong or hand-wavy answers is poor).

Update its row in `learning/README.md`'s tracker:
- **Solid pass** → `Last Reviewed` = today, `Next Review` = next rung up the ladder (blank/1d → 3d → 7d → 14d → 30d).
- **Poor pass** → `Last Reviewed` = today, `Next Review` = today + 1d (reset to the bottom of the ladder).

## 5. Implementation grading (on request only)

Never trigger this automatically. If the user asks to have a Topic's `implementation.md` graded:
1. Read their filled-in solutions and the stated expected outputs.
2. Check more than output-matching: correctness, edge cases missed, and whether the approach is reasonable for the problem — not just "did the number come out right."
3. Give specific, direct feedback per problem, not a generic pass/fail.

## Do not

- Do not answer a `recall.md` question yourself before the user attempts it.
- Do not touch `answers/`/`questions/` content in this skill — that's passive reference, not in scope for review.
- Do not grade `implementation.md` unless asked.
