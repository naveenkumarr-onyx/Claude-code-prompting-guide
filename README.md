# Claude Code Guides

<p align="center">
  <img src="./assets/banner.png" alt="Claude Code Guides — a good prompt = Context + Goal + Constraints + Verify" width="100%">
</p>

Practical, copy-paste references for getting the most out of **[Claude Code](https://docs.claude.com/claude-code)** — Anthropic's agentic coding tool.

These guides are written for developers who can already code but are **new to Claude Code** and want to go from "chatting with an AI" to "running an engineering harness."

> ⚠️ **This is a template and a guide, not a rulebook.** Everything here is a starting point — copy it, fork it, and **modify it to fit your own workflow, stack, and conventions.** Rename the files, cut sections you don't need, add your own templates. Use whatever's convenient for you.

---

## 📚 What's inside

| File | What it covers |
|------|----------------|
| [**claude-code-prompting-cheatsheet.md**](./claude-code-prompting-cheatsheet.md) | *What to say.* A 13-phase, ~130-template reference covering the entire development lifecycle — setup, planning, understanding code, building, debugging, refactoring, databases, testing, review, docs, git, CI/CD, and production ops. Find your situation, grab a template, fill the `<SLOTS>`, paste. |
| [**claude-code-advanced-guide.md**](./claude-code-advanced-guide.md) | *How to drive the tool.* The power features: plan mode, subagents, multi-agent workflows, skills, custom slash commands, MCP servers, `CLAUDE.md`, memory, hooks, permissions, context management, model/cost, and headless/CI usage. |

**The cheat sheet makes your individual prompts strong. The advanced guide makes your whole setup strong.**

---

## 🎯 Who this is for

- Developers trying Claude Code (or any AI coding agent) for the first time
- People who can spot a bug but aren't sure **how to phrase the request** to get a good result
- Teams who want a shared baseline for prompting and tooling conventions

---

## 🚀 How to use

1. **Start with the [cheat sheet](./claude-code-prompting-cheatsheet.md).** Read sections 0–4 once (the principles), then keep the template library handy as a daily reference.
2. **When you're comfortable, read the [advanced guide](./claude-code-advanced-guide.md)** to unlock plan mode, subagents, hooks, and workflows.
3. **Adapt them.** Swap the generic example names (`src/lib/users.ts`, `docs/conventions.md`, etc.) for your repo's real ones. Delete what doesn't apply. Add your own.

The single biggest idea, if you remember nothing else:

> **Every good prompt = Context + Goal + Constraints + how you'll verify.**

---

## 🧭 The core principle

```
❌ Weak:   fix the login bug

✅ Strong: Users get logged out after ~5 min, started after the JWT switch
           in lib/auth.ts.
           Goal: find why the session drops and fix the root cause.
           Constraints: don't change the public API; keep it minimal.
           Verify: log in, wait, confirm the session survives a refresh.
```

---

## ⚙️ A note on accuracy

Claude Code ships fast. Exact CLI flags, file locations, and config schemas can change between versions. The **concepts** in these guides are stable, but for exact current syntax always check:

- `claude --help` and the `/help` command
- The official docs: **[docs.claude.com/claude-code](https://docs.claude.com/claude-code)**

Treat any config snippets (hooks, settings, MCP, headless flags) as *shapes*, not gospel.

---

## 🤝 Contributing

Found a better template? Spotted something outdated? Have a scenario that's missing?

1. Fork the repo
2. Make your change (keep it generic — no company/product-specific details)
3. Open a pull request

Suggestions, corrections, and new templates are all welcome.

---

## 📄 License

Released under the **MIT License** — free to use, modify, and share. (Add a `LICENSE` file with your chosen license before publishing.)

---

*Made to be remixed. If these save you time, pass them on.*
