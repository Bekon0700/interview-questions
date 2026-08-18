# Interview & Learning Vault — Context

An Obsidian vault with two distinct purposes: fast interview-prep review, and active, AI-assisted deep learning of individual topics. The vocabulary below distinguishes them so future work (CLAUDE.md, skills) doesn't blur the two.

## Language

**Quick-review material**:
The `questions/` + `answers/` folder pair. Pre-written, polished interview Q&A meant for fast cramming under time pressure. Read passively — no forced-retrieval discipline applies here.
_Avoid_: "study notes", "learning notes" (reserved for Topic material below)

**Topic**:
A subject under active, from-scratch learning (e.g. Kafka), living at `learning/<category-slug>/<topic-slug>/` as a folder of fixed files (see below), not a single note. A Topic never lives directly under `learning/` — it always belongs to a Category.
_Avoid_: "learning note" (a Topic is a folder, not one file)

**Category** (mother concept):
A grouping folder under `learning/` for related Topics (e.g. `message-broker/` containing `kafka/`, `rabbitmq/`). Created on demand — the first time a Topic needs it, not pre-declared. Which Category a Topic belongs to (existing or new) is always confirmed with the user first; never auto-assigned, since the taxonomy itself is being built collaboratively, not inferred.

**Category-root file**:
Named after the Category itself (e.g. `message-broker/message-broker.md`). Contains: history, why the category was necessary, what problem it solves, alternative names the concept goes by, and a linked list of Topics under it. AI-drafted.

**comparison.md**:
Category-level file, created once a Category has 2+ Topics. Feature-by-feature comparison table across the Topics in that Category (e.g. Kafka vs. RabbitMQ vs. SQS on throughput, ordering guarantees, delivery semantics). AI-drafted.

**Bridging Topic**:
A Topic whose subject matter genuinely spans 2+ Categories (e.g. Redis: caching + message-broker). It physically lives under one primary Category only — no duplication. Its overview file must explicitly wikilink to the other Category's root file so Obsidian's native Graph View surfaces the relationship. There is no separate mind-map artifact; the Graph View *is* the mind map, so this linking is a deliberate, maintained step in topic generation, not optional.

**Topic overview file**:
The narrative file inside a Topic folder, named after the topic itself (e.g. `learning/message-broker/kafka/kafka.md`). Covers backstory, what/why/how. AI-drafted.

**all-features.md**:
Comprehensive feature list for a Topic, each entry paired with why it was necessary and an example. AI-drafted.

**pros-cons.md**:
Detailed pros/cons list for a Topic, each entry with a concrete example. AI-drafted.

**implementation.md**:
The hands-on file for a Topic. Contains one AI-provided worked example (fully solved, studied first) followed by a graduated problem set (beginner → advanced), each problem stated with its expected final output only — never the solution. The user implements every problem unaided in Obsidian/their own environment and self-evaluates against the stated expected output.
_Avoid_: "exercises.md", "practice.md"

**recall.md**:
Generated retrieval-practice file — short-answer questions plus a few "explain this to a beginner" prompts. Exists at the Topic level (pulled from that Topic's own theory files) and, once a Category has 2+ Topics, also at the Category level (pulled from `comparison.md`, e.g. "when would you choose Kafka over RabbitMQ, and why?"). This is the artifact the review skill draws from; nothing is reviewable until its `recall.md` exists.

**Interleaved review**:
Once 2+ Topics exist, review sessions mix items from multiple Topics in one sitting rather than exhausting one Topic before moving to the next.
