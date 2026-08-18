# Agentic Coding — Learning Plan (Mid-Level Engineer, TypeScript, Intensive Pace)

Goal: go from "uses Copilot/Claude to write code" to "builds and ships AI agents that autonomously use tools to complete multi-step tasks." Timeline below assumes several hours/day — realistically **2–3 weeks** to finish both projects with solid understanding, not just working code.

---

## Part 1: Core Concepts (do this before touching code)

You already know how to code, so skip tutorials that re-teach programming. Focus on the concepts unique to agentic systems:

**1. What makes something "agentic" vs. just "AI-powered"**
A chatbot answers questions. An agent decides *what to do next*, takes an action (calls a tool, edits a file, hits an API), observes the result, and decides again — in a loop, without you scripting every step. The loop is: *reason → act → observe → repeat until done*. This is the whole ballgame; everything else is plumbing around this loop.

**2. Tool use / function calling**
The mechanism by which an LLM says "call this function with these arguments" instead of just returning text. Learn the request/response shape (tool definitions with JSON schemas, the model's tool-call output, how you execute it and feed the result back in). This is the foundation every agent framework sits on top of.

**3. Context management**
Agents fail when their context window fills with junk or loses the important thread. Learn: system prompts vs. conversation history, summarization/compaction, and why long-running agents need deliberate context pruning (this is exactly what Claude Code's `/compact` and subagents solve).

**4. MCP (Model Context Protocol)**
The emerging standard (Anthropic-originated, now widely adopted) for how agents connect to external tools/data — file systems, databases, APIs, SaaS products — without custom glue code per integration. Understand it as "USB-C for AI tools": one protocol, many servers, any compliant client can use any compliant server.

**5. Permissions & guardrails**
The difference between a toy agent and a production one is almost entirely about what it's *allowed* to do without asking, what it must confirm, and what's hard-blocked. Learn permission models (allow/deny/ask lists), sandboxing, and why "just let it run bypass mode" is a demo trick, not a practice.

**6. Multi-agent orchestration**
When one agent's context gets overloaded or a task has cleanly separable sub-tasks, you delegate to subagents — isolated agents with their own context and tool access that report back a single result. Learn the difference between orchestration patterns: sequential handoff, parallel fan-out/fan-in, and supervisor/worker.

**7. Observability & evals**
How do you know your agent is actually doing the right thing, not just producing plausible-looking output? Learn to log tool calls, trace decision chains, and write simple evals (does the agent achieve the goal, how many turns did it take, did it call the right tools).

### Suggested study order (roughly days 1–4)
1. Read Anthropic's Agent SDK docs (overview + TypeScript quickstart) — you'll use this directly in Project 1.
2. Read the MCP spec overview and skim 2-3 existing MCP servers' source (e.g., filesystem, GitHub) to see the tool-definition pattern.
3. Read up on ReAct-style agent loops (the original reasoning pattern most frameworks still build on) — enough to explain it in your own words.
4. Skim LangGraph's docs just to understand *why* graph-based orchestration exists (state machines for agents) — you don't need to adopt it, but you should know what problem it solves and how it differs from Anthropic's SDK approach.

You'll know you're ready for projects when you can explain, without notes: the agent loop, how tool calling actually works under the hood, what MCP standardizes, and when you'd reach for a subagent vs. just a longer prompt.

---

## Part 2: The Two Projects

Both use **TypeScript** and the **Claude Agent SDK** (`@anthropic-ai/claude-agent-sdk`). Project 1 teaches the fundamentals in isolation; Project 2 forces you to combine everything — tools, MCP, permissions, multi-agent — into something that resembles real production work, which is what will actually impress PMs/POs, since it produces a visible, demoable workflow improvement.

### Project 1: "Repo Doctor" — a single-purpose coding agent (Week 1)

**What it does:** A CLI tool that, given a codebase, autonomously finds and fixes a specific class of problem — start with "missing/outdated JSDoc comments" or "unused imports and dead code" (pick whichever's more useful to you). It should: scan the repo, identify issues, propose fixes, show diffs, and apply them only after you approve (or auto-approve in a `--yolo` flag you build yourself, so you understand *why* that's dangerous).

**Why this project:** It's small enough to finish in days, but touches every core concept: agent loop, tool use (file read/write, running a linter/formatter as a shell tool), permission gating, and basic context management (repos are bigger than one context window — you'll have to decide how to chunk/prioritize files).

**Build milestones:**
1. Get a minimal agent running that can read a file and describe what it sees (proves your SDK setup + auth works).
2. Add a custom tool for scanning the repo (glob + grep equivalents) and a tool for proposing an edit.
3. Add a diff-preview + approval step before any write happens — this is your permissions layer.
4. Add a summary report at the end ("fixed 14 files, skipped 3 due to ambiguity") — this is your first taste of evals/observability.
5. Stretch goal: add a `--dry-run` mode and a simple log file of every tool call the agent made, with timestamps — this is the observability habit that separates hobby agents from production ones.

**Definition of done:** You can point it at a real repo (your own or an open-source one), run it, and get a genuinely correct set of proposed fixes with a clean diff — not something that half-works and needs manual cleanup.

---

### Project 2: "Standup Bot" — a multi-tool, multi-agent workflow assistant (Week 2)

**What it does:** An agent that automates a real recurring workflow — e.g., pulls yesterday's merged PRs and open issues from GitHub (via MCP), summarizes what shipped and what's blocked, drafts a standup update or a short status doc, and (optionally) posts it somewhere (Slack, a file, an email draft). Pick a workflow that's genuinely useful to you or your team so you stay motivated and end up with something demoable to your PM.

**Why this project:** This is where you go from "agent that edits code" to "agent that orchestrates across tools and systems," which is the skill PMs/POs actually notice — visible, recurring workflow automation, not just faster code review.

**Build milestones:**
1. Connect an MCP server (GitHub's is a good, well-documented starting point) and get the agent to pull real data (recent PRs/issues) — this is your first real MCP integration, not a toy example.
2. Add a second data source (could be another MCP server, or a custom tool hitting an internal API/spreadsheet) — this forces you to practice tool composition, not just single-tool calls.
3. Split the work into two agents: a "researcher" subagent that gathers and structures the raw data, and a "writer" subagent that turns structured data into a polished summary. This is your multi-agent orchestration practice — notice how much cleaner this is than one agent doing both jobs in one long context.
4. Add a permission boundary around anything that "publishes" (posting to Slack, sending email) — nothing external-facing should happen without an explicit confirm step, same lesson as Project 1 but now with real external-system stakes.
5. Wire it up to run on a schedule (cron, or a scheduled task if you're running this inside Claude Code/Cowork) so it's a genuinely recurring tool, not a one-off script.

**Definition of done:** It runs unattended (or with one confirm click) and produces a status update good enough that you'd actually send it to your team without editing it.

---

## Part 3: After the Two Projects

Once both are done, you'll have working proof for PMs/POs — that matters more than certificates. A few natural next steps if you want to keep going: add real evals (a small test suite that scores agent output quality, not just "did it run"), try a second orchestration framework (LangGraph) to compare its graph-based model against the SDK's more direct loop, and look at the emerging Agent-to-Agent (A2A) protocol if your work starts involving multiple *independently-owned* agents talking to each other (different from MCP, which is agent-to-tool).

---

### Reference links
- [Claude Agent SDK Overview](https://code.claude.com/docs/en/agent-sdk/overview)
- [Claude Agent SDK TypeScript Reference](https://code.claude.com/docs/en/agent-sdk/typescript)
- [MCP specification / docs](https://modelcontextprotocol.io)
- [The AI Agents Stack (2026 Edition) — O'Reilly Radar](https://www.oreilly.com/radar/the-ai-agents-stack-2026-edition/)
