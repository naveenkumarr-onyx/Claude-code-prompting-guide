# Claude Code Advanced Guide

**The companion to the [Prompting Cheat Sheet](./claude-code-prompting-cheatsheet.md).**

The cheat sheet is about *what to say*. This guide is about *how to drive the tool* — the power features that turn Claude Code from a chat box into an automated engineering harness. Read this once you're comfortable with basic prompting.

> **Heads-up on accuracy:** Claude Code ships fast. Exact CLI flags, file locations, and config schemas can change between versions. The *concepts* here are stable; for exact current syntax always check `claude --help`, the `/help` command, and the official docs at **docs.claude.com/claude-code**. Treat the config snippets as shapes, not gospel.

**Contents**
1. [Permission modes & plan mode](#1-permission-modes--plan-mode)
2. [Subagents & delegation](#2-subagents--delegation)
3. [Multi-agent workflows](#3-multi-agent-workflows)
4. [Skills](#4-skills)
5. [Custom slash commands](#5-custom-slash-commands)
6. [MCP servers (external tools & data)](#6-mcp-servers-external-tools--data)
7. [CLAUDE.md (project memory)](#7-claudemd-project-memory)
8. [Personal & persistent memory](#8-personal--persistent-memory)
9. [Hooks (event automation)](#9-hooks-event-automation)
10. [Permissions & settings](#10-permissions--settings)
11. [Context management (long sessions & big repos)](#11-context-management-long-sessions--big-repos)
12. [Model selection, thinking & cost](#12-model-selection-thinking--cost)
13. [Headless mode, scripting & CI](#13-headless-mode-scripting--ci)
14. [Working in large codebases](#14-working-in-large-codebases)
15. [Putting it together — recipes](#15-putting-it-together--recipes)
16. [Glossary](#16-glossary)

---

## 1. Permission modes & plan mode

Claude Code runs every action through a **permission mode**. Knowing them is the difference between babysitting every keystroke and letting it run safely.

| Mode | What it does | When to use |
|---|---|---|
| **Default** | Asks before edits / commands | Normal work |
| **Accept edits** | Auto-accepts file edits, still asks for risky commands | You trust the direction and want speed |
| **Plan mode** | **Read-only.** It investigates and proposes a plan, makes **no changes** until you approve | Anything non-trivial or unfamiliar |
| **Bypass permissions** | Runs everything without asking | Sandboxes/throwaway envs only — risky |

Cycle modes with **Shift+Tab**. Pick the looser modes only when the blast radius is contained.

### Plan mode — the most underused feature

Plan mode makes Claude Code **think before it touches anything**. It explores the repo, asks you questions, and writes a plan you approve *before* a single file changes. This is how you avoid "it confidently rewrote 14 files the wrong way."

**Use it for:** new features, refactors, migrations, anything in unfamiliar code, anything risky.

**How to invoke:** enter plan mode (Shift+Tab to the plan option), or just ask:
```
Plan this before doing anything: <TASK>. Investigate the relevant code,
ask me any questions, and propose an approach. Don't edit yet.
```

**Refine the plan before approving:**
```
Good plan, but change two things: <X> and <Y>. Also, what happens to
<EDGE CASE>? Update the plan, then I'll approve.
```

**The pattern that wins:** Plan → review/refine → approve → execute → verify. The few minutes planning saves you an hour of undoing.

---

## 2. Subagents & delegation

A **subagent** is a separate Claude instance with its own fresh context window, launched to handle one focused task and report back a result. The main session stays clean; the heavy lifting happens in the subagent.

### Why it matters
- **Context hygiene** — a big search or file-read dump lands in the subagent's context, not yours. You get the conclusion, not the noise.
- **Parallelism** — fan several out at once for independent work.
- **Specialization** — point a reviewer agent, an explorer agent, a builder agent at the right job.

### When to delegate
- Broad searches across many files ("find everywhere we do X")
- Independent chunks of work that can run in parallel
- A focused sub-task you don't want polluting the main thread (e.g. "review this diff")

### How to ask
```
Use subagents to explore these three areas in parallel and report back:
1) how auth works, 2) how the API layer is structured, 3) how tests are set up.
Summarize each — don't dump file contents.
```
```
Delegate this to a subagent: search the whole repo for every place we call
<API> and return a list of file:line with a one-line description each.
```

### Custom agent types
You can define specialized agents (a reviewer, a test-writer, a build-fixer) with their own instructions and tool access, then invoke them by name. They live in `.claude/agents/` (project) or `~/.claude/agents/`.

A minimal agent definition (Markdown with frontmatter):
```markdown
---
name: code-reviewer
description: Reviews diffs for bugs, security, and style. Use after writing code.
tools: Read, Grep, Glob, Bash
---
You are a senior code reviewer. Review the current diff for correctness bugs,
security issues, and deviations from project conventions. Report findings with
file:line and severity. Do not edit code — only report.
```
Then: `"Have the code-reviewer agent review my changes."`

> **Gotcha:** subagents don't share your conversation. Give them all the context they need *in the delegation prompt* — they can't see what you discussed earlier.

---

## 3. Multi-agent workflows

When a task is too big for one context — a repo-wide migration, an exhaustive audit, a large review — you orchestrate **many agents** in a structured pipeline: fan out, verify, synthesize.

### The mental model
- **Fan out:** split the work (per file, per dimension, per module) across parallel agents.
- **Verify:** have independent agents adversarially check the findings (try to *refute* each one).
- **Synthesize:** merge the survivors into one result.

### Common shapes
| Shape | Use for |
|---|---|
| **Find → verify** | Bug hunts, security audits (each finding double-checked before you trust it) |
| **Discover → transform → verify** | Migrations (find all sites, change each, confirm each) |
| **N attempts → judge → synthesize** | Design decisions (generate options, score, pick the best) |
| **Multi-modal sweep** | Research (several agents each search a different way) |
| **Loop-until-dry** | Unknown-size discovery (keep finding until rounds come up empty) |

### How to ask
```
This is a big, parallelizable job. Run it as a workflow:
- Fan out across all <FILES/MODULES> to <DO X>.
- Have each result independently verified before trusting it.
- Synthesize the confirmed results into one report.
```

> **Cost note:** workflows can spawn many agents and burn a lot of tokens. Match the scale to the task — a quick check doesn't need a 20-agent fan-out. (See [§12](#12-model-selection-thinking--cost).)

---

## 4. Skills

**Skills** are packaged capabilities — a folder with instructions (and optionally scripts/resources) that Claude Code loads on demand when your task matches. Think "reusable expertise" the model pulls in automatically or you invoke explicitly.

### Where they live
- Project: `.claude/skills/<skill-name>/SKILL.md`
- User: `~/.claude/skills/<skill-name>/SKILL.md`
- Or bundled in plugins.

### Anatomy
```markdown
---
name: api-validation
description: >
  Patterns for validating API routes with Zod. Use when creating or
  modifying API routes or request validation.
---

# API Validation

<the actual guidance, patterns, code templates, and steps the model should
follow when this skill is active>
```
The `description` is critical — it's how Claude decides when the skill is relevant. Make it specific ("use when X, Y, Z"), not vague.

### How to use
- **Automatic:** when your request matches a skill's description, it activates.
- **Explicit:** invoke it like a command — `/api-validation` — or "use the api-validation skill."

### When to build one
Any time you find yourself re-explaining the same domain knowledge, conventions, or multi-step process. Encode it once as a skill and every future session benefits.

---

## 5. Custom slash commands

A **slash command** is a saved prompt you trigger with `/name`. Perfect for repetitive asks ("review my diff", "write a conventional commit", "generate release notes").

### Where they live
- Project: `.claude/commands/<name>.md`
- User: `~/.claude/commands/<name>.md`

### Anatomy
```markdown
---
description: Review the current diff for bugs and simplifications
argument-hint: [optional area to focus]
---

Review my current git diff. Focus: $ARGUMENTS.
Report correctness bugs and simplification opportunities with file:line,
ranked by severity. Don't change anything — just report.
```

- `$ARGUMENTS` — everything the user typed after the command.
- `$1`, `$2`, … — individual positional args.
- You can run shell and inject output, reference files, etc. (check docs for the current syntax).

Invoke: `/review` or `/review the auth module`.

### Good candidates
Commit messages · PR descriptions · diff review · test generation for a file · changelog · "explain this file" · project-specific setup steps.

---

## 6. MCP servers (external tools & data)

**MCP (Model Context Protocol)** lets Claude Code talk to outside systems — databases, browsers, design tools, issue trackers, your own APIs — through a standard interface. Once connected, the model can call those tools like any built-in.

### What you can plug in
Databases (run queries) · browsers (drive a real browser for testing) · cloud platforms · design tools · documentation servers · payment/test sandboxes · custom internal tools.

### Adding a server (shape)
```bash
# A local stdio server (runs a command)
claude mcp add <name> -- <command-to-start-the-server>

# An HTTP/SSE remote server
claude mcp add --transport http <name> <url>

# List / inspect
claude mcp list
```

### Scopes
- **local** — just you, this machine/project
- **project** — shared with the repo via a checked-in `.mcp.json`
- **user** — available across all your projects

> **Security:** MCP servers can read data and take actions. Only add servers you trust, scope them tightly, and never commit secrets into `.mcp.json`. For sensitive credentials prefer local scope.

### How to use once connected
Just refer to the capability:
```
Use the database tool to find the 10 slowest queries in the last hour.
```
```
Use the browser tool to open the login page, sign in as a test user,
and confirm the dashboard renders.
```

---

## 7. CLAUDE.md (project memory)

`CLAUDE.md` is the file Claude Code reads automatically at the start of every session in a repo. It's where you teach it your project **once** so you don't repeat yourself.

### What belongs in it
- The stack and how to run / build / test
- Folder structure and architecture overview
- **Conventions** (naming, patterns, where things go)
- **Hard rules** ("never do X", "always use Y")
- Gotchas and non-obvious constraints
- Pointers to deeper docs

### What does NOT belong
- Secrets
- Things that change constantly
- Novel-length prose — keep it scannable; it's loaded into every session's context (it costs tokens).

### Locations & precedence
- Project root `CLAUDE.md` — shared with the team (commit it)
- `CLAUDE.local.md` — personal, untracked
- `~/.claude/CLAUDE.md` — applies to all your projects
- Subdirectory `CLAUDE.md` — scoped rules for that part of the repo

### Bootstrap one
```
Read this repo and generate a CLAUDE.md: stack, run/build/test commands,
folder structure, conventions, and any gotchas you find. Keep it concise
and accurate — it gets loaded every session.
```

### Maintain it
When a bug reveals a new gotcha, or you correct the model on a convention, add it:
```
Add a note to CLAUDE.md: we always use <RULE> because <REASON>.
```

---

## 8. Personal & persistent memory

Beyond `CLAUDE.md`, Claude Code can keep **memory** — facts about you and your preferences that persist across sessions (working style, tools you use, recurring context).

- Quickly add a memory by starting a line with `#` and typing the fact, or use the `/memory` command to view/edit.
- Good memories: durable preferences ("I use pnpm, not npm", "always show me the diff before applying"), recurring project context, who you are.
- Bad memories: one-off details, anything in the repo already, secrets.

**Tip:** the best memories are *behavioral* — how you want the agent to work — not facts it can rediscover by reading the code.

---

## 9. Hooks (event automation)

**Hooks** run *your* shell commands automatically at points in Claude Code's lifecycle. This is how you enforce things the model should never forget — because the harness runs them, not the model.

### Common events
| Event | Fires | Typical use |
|---|---|---|
| `PreToolUse` | Before a tool runs | Block/allow an action, validate inputs |
| `PostToolUse` | After a tool runs | Auto-format, run a linter, log |
| `UserPromptSubmit` | When you submit a prompt | Inject context, guardrails |
| `Stop` | When the agent finishes a turn | Notify you, run tests |
| `SubagentStop` | When a subagent finishes | Aggregate results |
| `SessionStart` | New/resumed session | Load context, print status |
| `PreCompact` | Before context compaction | Save state |

### Shape (in settings.json)
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "npx prettier --write \"$CLAUDE_FILE_PATHS\"" }
        ]
      }
    ]
  }
}
```
*(Exact field names/variables evolve — check the hooks docs for the current schema.)*

### Killer use cases
- **Auto-format** every file the model edits (so you never review formatting noise)
- **Run the linter/typecheck** after edits and feed errors back
- **Block** edits to protected paths (`.env`, generated files, `main` branch)
- **Notify** you (sound/desktop) when a long task finishes
- **Run tests** on Stop and report pass/fail

> Hooks are the right tool for *"from now on, always do X when Y happens."* The model can forget; a hook can't.

---

## 10. Permissions & settings

Settings live in JSON, layered from broad to specific:

- `~/.claude/settings.json` — your global defaults
- `.claude/settings.json` — project, shared (commit it)
- `.claude/settings.local.json` — project, personal (don't commit)

### Permission allow/deny
Pre-approve the safe, repetitive stuff so you stop getting prompted; deny the dangerous stuff outright.
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run test:*)",
      "Bash(git status)",
      "Read(./src/**)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(./.env)"
    ]
  }
}
```

### Reduce prompt fatigue (the right way)
1. Notice which safe commands you keep approving.
2. Add them to `allow`.
3. Keep destructive/outbound actions on manual approval.

You can also set env vars, default model, and hooks in the same file. Keep machine-specific or secret-ish settings in `settings.local.json`.

---

## 11. Context management (long sessions & big repos)

The context window is finite. Managing it well is a real skill on long tasks.

### Symptoms of a bloated context
- Slower responses, repetition, "forgetting" earlier decisions.

### Tools & habits
- **`/clear`** — wipe the conversation and start fresh. Use it liberally *between unrelated tasks*. The single biggest hygiene habit.
- **`/compact`** — summarize the conversation to reclaim space while keeping the gist. Happens automatically when full; you can trigger it early with guidance: `/compact focus on the auth refactor decisions`.
- **Delegate to subagents** — push big reads/searches into a subagent so only the conclusion returns ([§2](#2-subagents--delegation)).
- **Point, don't paste** — reference files by path instead of pasting huge blobs; let the model read what it needs.
- **One task per session** — don't let an afternoon of unrelated work pile into one thread.

### Resuming work
- `claude --continue` (or `-c`) resumes the most recent session.
- `claude --resume` lets you pick a past session.
Sessions persist, so you can pick up where you left off without re-explaining.

---

## 12. Model selection, thinking & cost

### Choosing a model
Match horsepower to the task:
- **Most capable model** — deep design, tricky bugs, large refactors, anything where being right matters.
- **Faster/cheaper model** — boilerplate, simple edits, quick lookups, high-volume mechanical work.

Switch with **`/model`**, or launch with `claude --model <name>`. You can let the harness route automatically if available.

### Extended thinking
Ask for more reasoning on hard problems with escalating keywords:
```
think            → a little
think hard       → more
think harder     → even more
ultrathink       → maximum
```
Use it for architecture, subtle bugs, and trade-off decisions. Skip it for trivial edits (it just costs tokens/time).

### Cost awareness
- **Prompt caching** keeps repeated context cheap — long-lived sessions benefit; jumping around resets it.
- **`/clear` between tasks** avoids dragging stale context (which you pay for every turn).
- **Right-size the work** — don't fan out 20 agents for a one-line fix; don't ultrathink a typo.
- Check usage with `/cost` (where available).

---

## 13. Headless mode, scripting & CI

Claude Code isn't just interactive — you can run it programmatically.

### One-shot / piped
```bash
# Print mode: run a prompt, get output, exit
claude -p "summarize the changes in the last commit"

# Pipe data in
cat error.log | claude -p "find the root cause in this log"

# Machine-readable output for scripts
claude -p "list TODOs in src/" --output-format json
```

### Constrain it for automation
```bash
# Only allow specific tools in an unattended run
claude -p "run the tests and report failures" --allowedTools "Bash(npm test:*)" Read
```

### Where this shines
- **CI checks** — automated code review on PRs, changelog generation, doc updates.
- **Batch jobs** — run the same transformation across many inputs.
- **Scheduled tasks** — nightly triage, report generation.

> **Safety in automation:** pin the allowed tools, avoid bypass-permissions unless fully sandboxed, and never expose secrets in prompts or logs.

---

## 14. Working in large codebases

Big repos don't fit in one context. Strategies that scale:

- **Start with a map, not the code.** Ask for the architecture and entry points first ([cheat sheet, Phase 3](./claude-code-prompting-cheatsheet.md#phase-3--understand-the-code)).
- **Let it search instead of you.** It greps the whole repo fast — "find where X is implemented" beats manual hunting.
- **Fan out exploration.** Several subagents, each mapping a different subsystem, returning summaries.
- **Keep a strong `CLAUDE.md`.** Conventions + structure up front means less rediscovery every session.
- **Work in vertical slices.** One feature/flow end-to-end, not the whole layer at once.
- **Use `/clear` between slices** so context stays focused.
- **Point at the pattern to copy.** "Do this the same way `<existing file>` does it" anchors output to your codebase.

---

## 15. Putting it together — recipes

**Ship a non-trivial feature**
```
1. Plan mode: "Plan <feature>. Investigate, ask questions, propose approach."
2. Review & refine the plan; approve.
3. Accept-edits mode: let it implement.
4. "Write tests for the new logic; run them."
5. "/review" (or code-reviewer agent) on the diff.
6. "Commit on branch feat/<x> with a conventional message; open a PR."
```

**Safe repo-wide migration**
```
1. "Find every site that uses <OLD>. List them. Read-only."
2. Plan the transformation; approve.
3. Run as a workflow: transform each site, verify each independently.
4. "Run the full test suite and report."
5. Review the diff in chunks; commit.
```

**Audit / bug hunt**
```
"Run a thorough workflow: fan out finders across the codebase for <CLASS OF
BUG>, have each finding adversarially verified before trusting it, and give
me only the confirmed issues with file:line and a fix suggestion."
```

**Make a recurring task one keystroke**
```
1. Find a prompt you repeat.
2. Save it as .claude/commands/<name>.md with $ARGUMENTS.
3. (Optional) add a hook so a follow-up step runs automatically.
4. Now it's "/<name>" forever.
```

**Tame a messy onboarding**
```
1. "Generate a CLAUDE.md for this repo." Review & commit.
2. Add allow-list permissions for your safe, repeated commands.
3. Add a PostToolUse hook to auto-format edits.
4. Save your top 3 repeated asks as slash commands.
```

---

## 16. Glossary

| Term | Meaning |
|---|---|
| **Agent / subagent** | A separate Claude instance with its own context, run for one focused task |
| **Workflow** | Deterministic orchestration of many agents (fan-out / verify / synthesize) |
| **Skill** | A loadable package of instructions/expertise the model uses on demand |
| **Slash command** | A saved prompt triggered with `/name` |
| **MCP** | Model Context Protocol — standard way to connect external tools/data |
| **Hook** | A shell command the harness runs automatically on a lifecycle event |
| **CLAUDE.md** | Project memory file auto-loaded each session |
| **Plan mode** | Read-only mode: it proposes a plan and changes nothing until you approve |
| **Permission mode** | How much the agent can do without asking (default / accept-edits / plan / bypass) |
| **Headless / print mode** | Running Claude Code non-interactively (`claude -p`) for scripts/CI |
| **Context window** | The working memory of a session; manage with `/clear`, `/compact`, delegation |
| **Extended thinking** | More reasoning on hard problems ("think" → "ultrathink") |

---

*This pairs with the [Prompting Cheat Sheet](./claude-code-prompting-cheatsheet.md). The cheat sheet makes your individual prompts strong; this guide makes your whole setup strong. Together they take you from "chatting with an AI" to "running an engineering harness."*
