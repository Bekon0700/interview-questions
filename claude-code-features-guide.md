# Claude Code — Complete Feature Guide

*Compiled Aug 2026 from code.claude.com docs. Command names/flags can change between releases — run `claude --help` or `/help` in a session to confirm what's current in your installed version.*

---

## 1. Installation & CLI Basics

Install: `npm install -g @anthropic-ai/claude-code` (needs Node.js 22+, and a Claude Pro/Max/Team/Enterprise account — free claude.ai doesn't work).

```bash
claude                          # start interactive session in current directory
claude "Review this codebase for security issues"   # start with a task
claude -p "Summarize this repo"                      # print mode: run once, no interactive loop
claude --continue                                    # resume most recent session
claude --resume <session-id>                          # resume a specific session
claude --model claude-opus                            # pick a model at startup
claude --permission-mode plan "Refactor the DB layer" # start in plan mode
```

## 2. Interactive Mode

The default mode: Claude proposes actions, shows diffs before editing files, and asks permission before running risky commands. Useful keys:

- `Shift+Tab` — cycle permission modes (default → auto → accept-edits → plan)
- `Escape` — cancel a prompt/menu
- `Ctrl+O` / `Cmd+O` — expand/collapse Claude's "thinking" blocks
- Double-tap `Escape` — open the rewind menu (see Checkpointing, below)

Example: type `Add input validation to the signup form` — Claude reads the relevant files, shows a diff, and waits for your approval before writing.

## 3. Slash Commands

Typed inside a session, starting with `/`.

| Command | What it does |
|---|---|
| `/help` | List commands and shortcuts |
| `/init` | Generate a starter `CLAUDE.md` for the project |
| `/model` | Switch model for this session (e.g. `/model claude-opus`) |
| `/clear` | Reset conversation context (file history still rewindable) |
| `/compact` | Summarize the conversation to save tokens |
| `/rewind` | Restore conversation + files to an earlier checkpoint |
| `/permissions` | Review current tool-permission rules |
| `/agents` | List/manage custom subagents |
| `/mcp` | Manage connected MCP servers |
| `/hooks` | View/manage configured hooks |
| `/config` | Open settings, or set one value: `/config model=claude-opus` |

Custom commands: drop a markdown file in `.claude/commands/my-audit.md` — it becomes usable as `/my-audit`, with the file's content as the instructions Claude follows.

## 4. CLAUDE.md — Project Memory

Claude reads these automatically at session start, so it doesn't need re-explaining your project every time.

```
CLAUDE.md                 # project memory, shared via git
.claude/CLAUDE.local.md   # local-only notes, not committed
~/.claude/CLAUDE.md       # personal, applies to all your projects
```

Example `CLAUDE.md`:

```markdown
# Project: MyApp

## Coding Standards
- Use async/await, not callbacks
- Tests required for all public functions

## Architecture
- API layer: /src/api
- Components: /src/components
```

## 5. Permissions

Controls what Claude can do without asking. Modes (toggle with `Shift+Tab` or `--permission-mode`):

- **default** — asks before every risky action
- **acceptEdits** — auto-approves file edits, still asks before shell commands
- **plan** — proposes a plan, executes nothing until you approve
- **bypassPermissions** — allows everything (CI/sandboxes only — avoid on your own machine)

Fine-grained rules live in `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": ["Read", "Edit", "Bash(npm run *)"],
    "deny": ["Bash(rm -rf *)", "Bash(git push *)"],
    "ask": ["Bash(*)"]
  }
}
```

## 6. Hooks

Scripts that run automatically at points in Claude's workflow — e.g., auto-format a file right after Claude edits it, or block a dangerous command before it runs.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "./scripts/format-code.sh" }]
      }
    ]
  }
}
```

Common events: `PreToolUse` (before a tool runs — can block/modify it), `PostToolUse` (after — can add context or trigger follow-up actions).

## 7. Subagents

Specialized assistants with their own system prompt, tool access, and context window — useful for offloading a well-defined task (like code review) without cluttering your main conversation.

`.claude/agents/code-reviewer.md`:

```markdown
---
name: Code Reviewer
description: Reviews code for bugs, security issues, and style
allowedTools: [Read, Glob, Grep]
---
You are an expert reviewer. Flag bugs, security issues, and
style violations with line numbers and suggested fixes.
```

Invoke it naturally: "Have the code-reviewer check src/auth.js."

## 8. MCP (Model Context Protocol) Servers

Connects Claude Code to external tools — databases, GitHub, Slack, etc.

```bash
claude mcp add github                      # add a pre-built server
claude mcp add --transport http my-db https://api.example.com/mcp/
claude mcp list                            # see connected servers
```

Once added, Claude can call that server's tools automatically when relevant to your request — e.g., "look up the open issues on this repo" if a GitHub MCP server is connected.

## 9. Plugins & Marketplaces

Plugins bundle skills, agents, hooks, and MCP servers into one installable package.

```bash
claude plugin install name/repo     # from a marketplace
claude plugin install ./my-plugin   # from a local folder
claude plugin list
```

## 10. Output Styles & Custom Slash Commands

Output styles change Claude's overall behavior/tone (e.g., "Teaching" mode explains its reasoning more, "Minimal Coding" is terse). Switch via `/config`. Custom slash commands (see §3) let you package a repeatable prompt/workflow as a one-word command.

## 11. IDE Integrations

VS Code and JetBrains (IntelliJ, PyCharm, WebStorm, etc.) extensions embed Claude Code directly in the editor:

- `Ctrl/Cmd+Esc` — open/toggle the Claude panel
- Selected code is automatically shared with Claude
- Diffs display in the IDE's native diff viewer
- `@path/to/file.js:10-20` — reference a specific file/line range in chat

## 12. Claude Agent SDK

For developers who want to build their own agent on top of Claude Code's engine (tools, permissions, context management) rather than use the CLI directly.

```bash
pip install claude-agent-sdk          # Python
npm install @anthropic-ai/claude-agent-sdk   # TypeScript
```

```python
from claude_agent_sdk import query

async for result in query("Find and fix bugs in src/app.py"):
    if result.type == "message":
        print(result.text)
```

Supports custom tools, structured JSON output, subagents, and hooks defined in code rather than config files.

## 13. Configuration

Settings cascade: CLI flags > `.claude/settings.local.json` > `.claude/settings.json` (project, committed) > `~/.claude/settings.json` (personal, all projects).

Key environment variables:

```bash
ANTHROPIC_API_KEY=sk-...        # auth (optional if logged in)
ANTHROPIC_MODEL=claude-opus     # default model override
```

`.claude/settings.json` example:

```json
{
  "model": "claude-opus",
  "permissions": { "allow": ["Read", "Glob", "Grep"] },
  "env": { "NODE_ENV": "development" }
}
```

## 14. Git / GitHub Integration

Claude can review PRs, analyze diffs, and help fix CI failures.

```bash
claude "Review this PR for bugs and security issues"
```

GitHub Actions integration (`.github/workflows/claude-review.yml`) can trigger a review automatically whenever a PR opens, using the `anthropics/claude-code-action`.

## 15. Extended Thinking / Reasoning Effort

Controls how much "thinking" Claude does before responding — more thinking generally means better answers on hard problems, at the cost of time and tokens.

```bash
/effort high        # inside a session
claude --effort high   # at startup
```

Lower it (`/effort low`) for simple, fast tasks to save cost; raise it for gnarly bugs or architecture decisions.

## 16. Checkpointing / Rewind

Claude automatically snapshots your files and conversation at each prompt, so you can undo without git.

```bash
/rewind          # pick a checkpoint to restore
/rewind -3        # go back 3 turns
```

Works even across sessions — `claude --resume <id>` then `/rewind` still works.

---

### Quick note on freshness

Claude Code ships frequent updates, and some flags/commands above (especially around cloud sessions, scheduling, and newer effort levels) may have changed names or scope since this was written. When in doubt, `claude --help` and `/help` inside a live session are the source of truth.
