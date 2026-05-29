# Claude Code Prompting Cheat Sheet

**For developers who can code but are new to Claude Code.**

You already know how to find and fix bugs. The missing skill is *talking to the agent* — knowing **which situations it's great at** and **how to phrase the request** so you get a strong result the first time.

This is a copy-paste reference covering the **entire development lifecycle, end to end** — from starting a project to operating it in production. Find your situation, grab a template, fill in the `<SLOTS>`, paste it in.

> **How to use this doc:** Sections 0–4 teach the *principles* (read once). [Section 5](#5-end-to-end-template-library) is the **exhaustive template bank** organized by development phase — that's your daily reference. [Section 6](#6-working-style-modifiers) has snippets you append to *any* prompt.

**Jump to a phase:**
[Setup](#phase-1--project-setup--inception) ·
[Plan & Design](#phase-2--plan--design) ·
[Understand Code](#phase-3--understand-the-code) ·
[Build Features](#phase-4--implement--build) ·
[Debug & Fix](#phase-5--debug--fix) ·
[Refactor](#phase-6--refactor--evolve) ·
[Data & DB](#phase-7--data--databases) ·
[Test](#phase-8--test--verify) ·
[Review](#phase-9--review--quality) ·
[Document](#phase-10--document) ·
[Git](#phase-11--version-control--collaboration) ·
[CI/CD & Deploy](#phase-12--build-cicd--deploy) ·
[Operate](#phase-13--operate--maintain-production)

---

## 0. How to think about Claude Code

- It's a **pair-programmer that can read your whole repo** — but it **can't read your mind**.
- Every good prompt has four ingredients: **Context + Goal + Constraints + how you'll verify**.
- Let it **explore before it edits**. For anything non-trivial, ask it to **plan first** or **ask you questions** before changing code.
- **Verify the output.** Read the diff, run the tests. Don't merge on trust.
- Talking to it costs almost nothing. **Over-explain, then narrow** — a slightly long prompt beats five rounds of back-and-forth.
- **One job per prompt.** A focused request beats a kitchen-sink one. Chain them.

---

## 1. The anatomy of a good prompt

The single biggest upgrade you can make:

> ❌ **Weak:** `fix the login bug`
>
> ✅ **Strong:**
> ```
> Users get logged out randomly after ~5 min. It started after we
> switched to JWT in apps/web/lib/auth.ts.
> Goal: find why the session drops and fix the root cause.
> Constraints: don't change the public API; keep it minimal.
> Verify: log in, wait, confirm the session survives a refresh.
> ```

Same four ingredients every time:

| Ingredient | What it answers | Example |
|---|---|---|
| **Context** | What's happening + where | "logged out after 5 min, started after the JWT switch in `auth.ts`" |
| **Goal** | What "done" looks like | "find the root cause and fix it" |
| **Constraints** | What it must/must not do | "don't change the public API; keep it minimal" |
| **Verify** | How you'll know it worked | "log in, wait, refresh, session survives" |

If a prompt is going badly, it's almost always because one of these is missing.

---

## 2. Scenario library (learn the pattern)

Five quick before/afters so the principle sticks. The full per-situation templates are in [Section 5](#5-end-to-end-template-library).

| Situation | ❌ Weak | ✅ Strong | Why |
|---|---|---|---|
| Bug, cause unknown | `the page is broken` | `Dashboard shows a blank chart for new users only. No console error. Investigate the cause before changing anything, then propose a fix.` | symptom + who + "investigate first" |
| Build failing | `build is failing` | `Build fails with the error below. Fix the type errors only — no refactors. <paste output>` | the real error text is the highest-signal input |
| Add a feature | `add dark mode` | `Add a dark-mode toggle. Acceptance: persists across reloads, respects OS default, uses our Tailwind tokens. Match existing theming. Plan first.` | acceptance criteria + match patterns + plan |
| Refactor | `clean up this file` | `Refactor checkout.ts for readability. Behavior must stay identical. Run the tests after to confirm nothing changed.` | "behavior identical" + "run tests" = safe |
| "Not working" | `it doesn't work` | `On submit, nothing happens. Expected: success toast + redirect. Actual: button spins, no network request. Component: <path>. Find why.` | expected vs actual vs observed |

---

## 3. Power features new devs miss

| Feature | What it is | How to ask for it |
|---|---|---|
| **Plan mode** | It designs the change and waits for approval before editing | "Plan this first, don't edit yet." |
| **Parallel exploration** | Fans out many searches across a big/unknown repo at once | "Search the whole repo for everywhere we do X." |
| **Skills & slash commands** | Prebuilt workflows like `/review`, `/code-review` | Type `/` to list them |
| **Make it ask YOU** | It interviews you to kill ambiguity before working | "Ask me clarifying questions before you start." |
| **@file / logs / screenshots** | Feed it real signal instead of describing it | Reference the path, paste the full error, drop a screenshot |
| **CLAUDE.md / memory** | Teach it your stack & rules once | "Add a CLAUDE.md with our conventions" |

---

## 4. Anti-patterns (what NOT to do)

- ❌ **Vague one-liners** — "fix it", "it's broken". No context = it guesses.
- ❌ **Hiding the error** — describing an error instead of pasting the actual text/trace.
- ❌ **Ten asks in one prompt** — do one thing well, then the next.
- ❌ **No constraints** — and then it refactors 12 files you didn't want touched.
- ❌ **Merging on trust** — accepting a diff without reading it or running anything.
- ❌ **Fighting it** — re-sending the same prompt louder. Add the missing context instead.
- ❌ **Asking it to guess your intent** — "do the right thing" when *you* don't know it yet. Decide first, or ask it to lay out options.

---

## 5. End-to-end template library

The main event. Copy a block, fill the `<SLOTS>`, paste. Append anything from [Section 6](#6-working-style-modifiers) to tune behavior.

---

### Phase 1 — Project setup & inception

**Start a new project from scratch**
```
Set up a new <TYPE, e.g. Next.js + TypeScript> project for <PURPOSE>.
Include: <linting, formatting, testing, env handling>.
Use <PACKAGE MANAGER>. Explain the folder structure you create.
Plan the structure first, then scaffold it.
```

**Scaffold a new module/feature folder**
```
Scaffold a new <MODULE> following the structure of the existing
<EXAMPLE MODULE>. Create the files but leave the logic as TODOs with
clear comments. Match our naming conventions.
```

**Choose / evaluate a library**
```
I need a library for <NEED, e.g. date handling>. Compare 2-3 popular,
well-maintained options for: bundle size, API ergonomics, maintenance,
and fit with our stack (<STACK>). Recommend one and say why. Don't install yet.
```

**Set up tooling (lint / format / test / CI)**
```
Add <TOOL, e.g. ESLint + Prettier> to this project with a sensible config
for <STACK>. Wire it into package.json scripts. Don't reformat the whole
repo yet — just set it up and show me the config.
```

**Set up a monorepo / workspace**
```
Convert this into a <pnpm/turbo/nx> monorepo with packages/ and apps/.
Move <CURRENT CODE> into <app/package>. Plan the migration first and show
me the new structure before moving anything.
```

**Bootstrap a CLAUDE.md for this repo**
```
Read through this repo and create a CLAUDE.md that documents: the stack,
how to run/build/test, the folder structure, naming conventions, and any
gotchas you notice. Keep it concise and accurate.
```

**Set up the local dev environment**
```
I just cloned this repo. Walk me through getting it running locally:
prerequisites, install steps, env vars I need, and the command to start it.
Check the actual scripts/config — don't guess.
```

---

### Phase 2 — Plan & design

**Turn a vague idea into a spec**
```
I want to build <ROUGH IDEA>. Help me turn this into a clear spec:
ask me the questions you need answered, then write user stories and
acceptance criteria. Don't write code yet.
```

**Design the architecture**
```
Design the architecture for <FEATURE/SYSTEM>. Constraints: <SCALE, STACK,
LIMITS>. Cover: components, data flow, key trade-offs, and what could go
wrong. Give me a diagram in text and a recommendation. No code yet.
```

**Compare approaches / trade-offs**
```
I'm deciding between <APPROACH A> and <APPROACH B> for <PROBLEM>.
Lay out the trade-offs (complexity, performance, maintenance, risk),
considering our context: <CONTEXT>. Recommend one.
```

**Write a design doc / RFC**
```
Write a design doc for <FEATURE>: problem statement, goals/non-goals,
proposed approach, alternatives considered, risks, and a rollout plan.
Base it on how this repo actually works. Markdown.
```

**Break a feature into tasks**
```
Break <FEATURE> into small, ordered, independently-shippable tasks.
For each: what it does, which files it touches, and its dependencies.
Flag anything risky or uncertain.
```

**Design a data model / schema**
```
Design the database schema for <DOMAIN>. List tables, columns, types,
relationships, and indexes. Follow the conventions in our existing schema.
Explain the key modeling decisions. Don't write the migration yet.
```

**Design an API**
```
Design a REST API for <RESOURCE>. List the endpoints (method + path),
request/response shapes, status codes, auth, and error format.
Follow the patterns of our existing API. No implementation yet.
```

---

### Phase 3 — Understand the code

**Onboard me to this repo**
```
I'm new to this codebase. Give me a high-level map: main entry points,
how a typical request/action flows through the layers, the key folders,
and the conventions I should follow. Read-only.
```

**Trace a feature end-to-end**
```
Trace how <FEATURE> works end-to-end, from <ENTRY POINT> to <END>.
List each file and function in the path. Read-only.
```

**Find where something is implemented**
```
Find where <BEHAVIOR/STRING/FEATURE> is implemented. Show me the file(s)
and the specific function responsible. Read-only.
```

**Explain a file / function**
```
Explain <FILE OR FUNCTION>: what it does, inputs/outputs, and anything
surprising or risky. Don't change it.
```

**Map data flow for an entity**
```
Show me everywhere <ENTITY> is read from and written to — DB, API routes,
and UI. Read-only.
```

**Understand a dependency / library usage**
```
How is <LIBRARY> used in this repo? Show me where it's configured, the main
call sites, and any custom wrappers around it. Read-only.
```

**Understand the build / config**
```
Explain how this project builds and what each config file
(<list, e.g. tsconfig, vite.config, etc.>) actually controls.
Point out anything non-standard.
```

---

### Phase 4 — Implement & build

**Add a feature**
```
Add <FEATURE>.
Acceptance criteria:
- <CRITERION 1>
- <CRITERION 2>
Match how similar things are done in this repo (tell me the pattern you
follow). Plan it first, then implement.
```

**Add an API endpoint / route**
```
Add a <METHOD> endpoint at <PATH> that <DOES WHAT>.
Input: <SHAPE>. Output: <SHAPE>. Validate input, handle errors, and follow
the existing route patterns in <FOLDER>. Add a test.
```

**Add a form with validation**
```
Add a <FORM NAME> form with fields <LIST>. Validate <RULES>, show inline
errors, and submit to <ENDPOINT>. Use our existing form/validation setup.
Handle loading and error states.
```

**Add a UI component**
```
Build a <COMPONENT> that <DOES WHAT>. Match our design system
(<TOKENS/LIBRARY>), reuse existing components, mobile-first.
Show me the result, then I'll review.
```

**Add authentication / authorization**
```
Add <AUTH REQUIREMENT, e.g. "role-based access so only admins can hit /admin">.
Follow our existing auth pattern in <FILE>. Fail closed (deny by default).
Add tests for allowed and denied cases.
```

**Add a background job / cron**
```
Add a scheduled job that <DOES WHAT> every <INTERVAL>. Follow the existing
cron/job pattern in <FOLDER>. Make it idempotent and log what it does.
```

**Add realtime / websockets**
```
Add realtime updates for <FEATURE> so <WHO> sees <WHAT> without refreshing.
Use <our realtime mechanism, e.g. Supabase Realtime / WebSocket>. Handle
reconnects. Follow existing patterns — no polling.
```

**Add file upload / storage**
```
Add file upload for <PURPOSE>. Accept <TYPES>, max <SIZE>, store in
<STORAGE>. Validate on the server, return the URL. Handle the error cases.
```

**Add email / notifications**
```
Send a <NOTIFICATION TYPE> when <EVENT>. Use <PROVIDER>. Follow the existing
email/notification pattern. Make the template, wire the trigger, handle send
failures gracefully.
```

**Add search / filter / pagination**
```
Add <search / filtering / pagination> to <LIST/ENDPOINT>. Params: <LIST>.
Keep it efficient (push it to the DB, not in-memory). Follow existing query
patterns. Add a couple of tests.
```

**Add caching**
```
Cache <EXPENSIVE OPERATION> to speed up <WHAT>. Use <CACHE, e.g. in-memory
with TTL>. Make sure it invalidates correctly when <DATA CHANGES>. Keep it
simple — don't over-engineer.
```

**Add a CLI command / script**
```
Add a script that <DOES WHAT>. Args: <LIST>. Follow the pattern of existing
scripts in <FOLDER>. Default to a dry-run; require a flag to actually execute.
```

**Add a feature flag / config**
```
Add a feature flag <NAME> (default <VALUE>) that gates <BEHAVIOR>. Follow the
existing flag pattern. Off by default.
```

**Implement an algorithm / business rule**
```
Implement <RULE/ALGORITHM>: <DESCRIBE PRECISELY, with examples of input →
output>. Handle the edge cases <LIST>. Add tests covering each case.
```

**Wire a third-party integration**
```
Integrate <SERVICE> to <PURPOSE>. Use <SDK/METHOD>. Keep secrets out of code
(use <ENV SOURCE>). Follow how other integrations are wired here. Plan first.
```

**Add internationalization (i18n)**
```
Make <COMPONENT/PAGE> translatable. Add the keys to <MESSAGES FILE> first
(English), then replace hardcoded strings with our translation hook.
Don't leave any hardcoded user-facing text.
```

---

### Phase 5 — Debug & fix

**Bug — cause unknown**
```
Bug: <SYMPTOM>. Happens when <CONDITION>, affects <WHO/WHEN>.
Expected: <EXPECTED>. Actual: <ACTUAL>.
Investigate the root cause before changing anything, then propose a fix.
```

**Bug — cause known**
```
The bug is in <FILE:FUNCTION>: <WHAT'S WRONG>. Fix it so <CORRECT BEHAVIOR>.
Smallest change that works — don't refactor unrelated code.
```

**Build / type / compile error**
```
The build fails with the output below. Fix only what's needed to compile —
no refactors, no behavior changes.

<PASTE FULL BUILD OUTPUT>
```

**Runtime error / stack trace**
```
I get this error when I <ACTION>. Trace it to the source and fix the root
cause, not the symptom.

<PASTE FULL STACK TRACE>
```

**Wrong output (logic bug)**
```
<FUNCTION/FEATURE> gives the wrong result. For input <INPUT> I get <ACTUAL>
but expected <EXPECTED>. Find the logic error and fix it. Add a test that
locks in the correct behavior.
```

**Intermittent / hard-to-reproduce bug**
```
<SYMPTOM> happens intermittently — maybe 1 in <N> times, under <SUSPECTED
CONDITIONS>. Investigate likely causes (timing, state, caching, race). Don't
patch randomly — explain the mechanism before fixing.
```

**Race condition / concurrency bug**
```
I suspect a race condition in <AREA>: <WHAT GOES WRONG> when <CONCURRENT
THING>. Find the unsafe access and fix it properly (no sleep/delay hacks).
Explain why your fix is correct.
```

**Memory leak / growing resource use**
```
<WHAT> grows over time / leaks memory. Find what isn't being released
(listeners, timers, subscriptions, caches) and fix it. Explain the leak.
```

**Works locally, breaks in production/staging**
```
<FEATURE> works locally but fails in <ENV>. Symptom there: <SYMPTOM>.
Likely differences: env vars, build mode, data, timing. Investigate the
env-specific cause — don't change behavior that already works locally.
```

**Dependency / version conflict**
```
Getting <ERROR> related to <PACKAGE> versions. Diagnose the conflict and
resolve it with the minimal, safest change. Explain what was incompatible.

<PASTE ERROR>
```

**Network / API call failing**
```
The call to <ENDPOINT/SERVICE> fails with <STATUS/ERROR>. Check the request
(headers, body, auth, URL) and the handling. Find why and fix it.

<PASTE REQUEST/RESPONSE OR ERROR>
```

**CORS / 401 / 403 issue**
```
Getting <CORS error / 401 / 403> when <ACTION> from <ORIGIN>. Trace the
auth/headers/config and fix the root cause. Don't disable security to make
it pass — fix it properly.
```

**Failing or flaky test**
```
Test <NAME> in <FILE> is <failing / flaky>. Find the real cause and fix it.
Don't loosen the assertion or add retries to hide it.
```

**Regression — worked before, broke after a change**
```
<FEATURE> worked before but broke after <CHANGE/COMMIT/DEPLOY>. Compare what
changed, find what broke it, fix it, and confirm the original behavior is back.
```

**Live production incident**
```
Production issue right now: <SYMPTOM + IMPACT>. Help me triage fast:
likely causes ranked, what to check first, and the safest immediate
mitigation. We'll do the proper fix after. <PASTE any logs/errors>
```

**Debug from an error report (Sentry, etc.)**
```
Here's an error report from <TOOL>: <PASTE>. Map it to the code, explain
the cause, and propose a fix. Note how often it fires / who it hits if known.
```

**Debug from logs**
```
Here are the logs around the failure: <PASTE>. Reconstruct what happened
step by step, identify where it goes wrong, and fix the cause.
```

---

### Phase 6 — Refactor & evolve

**Refactor without changing behavior**
```
Refactor <FILE/AREA> for <readability / smaller functions / less duplication>.
Behavior must stay identical. Run the existing tests afterward and confirm
nothing changed.
```

**Rename across the codebase**
```
Rename <OLD> to <NEW> everywhere — code, imports, references. No behavior
change. Show me the full list of files touched.
```

**Extract / dedupe shared logic**
```
<LOGIC> is duplicated in <FILES>. Extract it into one shared <function/module>
and update all call sites. Keep behavior identical; run the tests.
```

**Split a large file / module**
```
<FILE> is too big and does too much. Split it into focused modules by
responsibility, keeping the public interface stable. Update imports.
No behavior change; run the tests.
```

**Reduce coupling / improve structure**
```
<AREA> is tightly coupled to <DEPENDENCY>. Refactor to decouple them
(<e.g. introduce an interface / inject the dependency>) without changing
behavior. Explain the new structure.
```

**Convert to TypeScript / add types**
```
Convert <FILE(S)> from JS to TypeScript with accurate types — no `any`
unless truly unavoidable (explain where). Don't change runtime behavior.
Make sure it type-checks.
```

**Modernize legacy code**
```
Modernize <FILE/AREA> to current idioms (<e.g. callbacks→async/await,
class→hooks>) without changing behavior. Do it incrementally and run the
tests after each step.
```

**Upgrade a framework / major version**
```
Upgrade <PACKAGE> from <OLD> to <NEW>. Read the migration guide, find every
breaking change that affects us, and apply the fixes. Plan it first; show me
the breaking changes before editing. Verify the build and tests pass.
```

**Migrate from library A to B**
```
Replace <LIBRARY A> with <LIBRARY B> across the repo. Map the equivalent APIs,
update all call sites, and remove A. Plan first, show me one example diff,
then proceed. Keep behavior identical.
```

**Mass edit / codemod**
```
Apply this change everywhere it occurs: <CHANGE>. First show me how many files
are affected and one example diff, then proceed once I confirm.
```

**Risky op — plan first, confirm before applying**
```
I want to <RISKY ACTION>. First find everything that references it (code + DB)
and show me the impact. Do NOT change anything until I say go.
```

**Remove dead code**
```
Find code in <AREA / the repo> that's unused (no references, dead branches,
orphaned files) and list it with evidence. Don't delete yet — show me the
list, then I'll confirm what to remove.
```

**Tech-debt cleanup**
```
Review <AREA> for tech debt: duplication, confusing names, missing error
handling, risky patterns. Give me a prioritized list with effort/impact.
Don't change anything yet.
```

---

### Phase 7 — Data & databases

**Design a schema**
```
Design the schema for <DOMAIN>: tables, columns, types, relationships,
indexes, constraints. Follow our existing schema conventions. Explain the
key decisions. No migration yet.
```

**Write a migration**
```
Write a migration to <CHANGE>. First check the current schema and our
migration pattern in <FOLDER>. Make it reversible if possible. Update any
affected types/queries. Show me the migration before applying.
```

**Write a complex query**
```
Write a query that <RETURNS WHAT> from <TABLES>, joining on <RELATIONSHIPS>,
filtered by <CONDITIONS>. Verify the column/table names against the actual
schema first — don't guess.
```

**Optimize a slow query**
```
This query is slow: <PASTE QUERY + timing>. Explain why (check indexes,
joins, scans), then propose an optimized version. Confirm it returns the
same results.
```

**Backfill / data migration**
```
I need to backfill <FIELD/DATA> for existing rows where <CONDITION>. Write a
safe, batched, idempotent script. Default to dry-run; require a flag to write.
Show me the plan before running anything.
```

**Seed / test data**
```
Create seed data for <SCENARIO> so I can test <FEATURE> locally. Follow our
test-data conventions. Make it easy to identify and clean up.
```

**Fix a data integrity issue**
```
We have bad data: <DESCRIBE, e.g. orphaned rows / wrong values>. First write
a read-only query to show me how many rows are affected. Then propose the fix.
Don't modify data until I confirm.
```

---

### Phase 8 — Test & verify

**Write unit tests**
```
Write unit tests for <FUNCTION/MODULE>. Cover happy path, edge cases
(<null, empty, boundary, error>), and failure modes. Use the existing test
framework and style.
```

**Write integration tests**
```
Write integration tests for <FLOW, e.g. "create order → charge → confirm">.
Test the real interaction between <COMPONENTS>. Follow our existing
integration-test setup.
```

**Write E2E tests**
```
Write an end-to-end test for <USER JOURNEY>: <STEPS>. Use our E2E setup
(<tool>). Assert the user-visible outcome at each step.
```

**Increase coverage (risk-based)**
```
Find the most important untested logic in <AREA> and add tests for it.
Prioritize risk, not line count. Use the existing setup.
```

**Fix a flaky test**
```
<TEST> is flaky. Find the source of nondeterminism (timing, order, shared
state, real network/clock) and make it deterministic. Don't just add retries.
```

**Set up mocks / fixtures / test data**
```
Set up <mocks/fixtures> for <DEPENDENCY> so I can test <UNIT> in isolation.
Follow the existing mocking pattern. Keep fixtures minimal and realistic.
```

**Property-based / fuzz tests**
```
Add property-based tests for <FUNCTION>: define the invariants it must always
satisfy and let the framework generate inputs. Use <tool if any>.
```

**Manually verify a change**
```
Verify that <CHANGE> actually works by running the app and exercising
<SCENARIO>. Tell me the steps you took and what you observed — don't just
assume it works.
```

---

### Phase 9 — Review & quality

**Review my diff before commit**
```
Review my current diff for correctness bugs and anything to simplify or
reuse. Be specific (file + line), ranked by severity. Don't change anything —
just report.
```

**Review someone else's PR**
```
Review this PR/branch <REF>. Focus on: correctness, edge cases, security,
and whether it fits our patterns. Give actionable comments with file + line.
Read-only.
```

**Security review**
```
Security-review <AREA>. Look for injection, auth/authorization gaps, secret
leakage, SSRF, and unsafe input handling. Report issues with severity and a
suggested fix. Read-only.
```

**Accessibility review**
```
Review <COMPONENT/PAGE> for accessibility: semantics, keyboard nav, focus,
labels, contrast, ARIA. List issues with severity and the fix. Read-only.
```

**Performance review / web vitals**
```
Review <PAGE/FLOW> for performance. Find the biggest wins (bundle size,
renders, queries, blocking work) and rank them by impact vs effort.
Don't change anything yet.
```

**API design review**
```
Review the design of <API/ENDPOINTS> for consistency, correctness, naming,
status codes, and error handling. Suggest improvements. Read-only.
```

---

### Phase 10 — Document

**Write / update the README**
```
Update the README <SECTION> to match the current behavior (check the actual
<code/scripts/config>). Keep the existing tone and structure. Don't document
things that aren't true.
```

**Write API docs**
```
Document the <API/ENDPOINTS> in <FORMAT>: method, path, params, request/
response, auth, errors, and an example. Generate it from the actual route
code so it stays accurate.
```

**Add docstrings / comments**
```
Add docstrings to the public functions in <FILE> explaining what, why, params,
and return. Comment only the non-obvious "why" in the body — don't narrate
obvious lines.
```

**Write an architecture / decision doc (ADR)**
```
Write an ADR for <DECISION>: context, the decision, alternatives considered,
consequences. Base it on how the code actually works. Markdown.
```

**Write a changelog / release notes**
```
Generate changelog entries for the changes since <REF/TAG>. Group by
Added/Changed/Fixed/Removed, user-facing language, concise.
```

**Write an onboarding guide**
```
Write a "getting started for new developers" guide for this repo: setup,
architecture overview, key conventions, how to run/test, and common gotchas.
Base it on the real repo.
```

---

### Phase 11 — Version control & collaboration

**Write a commit message**
```
Write a clear conventional-commit message for my staged changes — summarize
what changed and why. Don't commit yet, just draft it.
```

**Branch + commit**
```
Commit my changes on a new branch <BRANCH> with a clear conventional-commit
message. Don't push yet.
```

**Open a PR**
```
Open a PR for this branch into <BASE>. Write a summary (what + why), a test
plan, and call out anything risky. <Link issue #N if any>.
```

**Resolve a merge conflict**
```
I have merge conflicts merging <SOURCE> into <TARGET>. Walk through each
conflict, explain both sides, and resolve them preserving the intent of both
changes. Show me each resolution before finalizing.
```

**Revert a change**
```
Revert <COMMIT/CHANGE> safely without undoing unrelated work since then.
Explain what you're reverting and verify nothing else breaks.
```

**Cherry-pick / port a change**
```
Port the change <COMMIT/DESCRIPTION> from <BRANCH> to <BRANCH>. Adapt it to
any differences between the branches. Show me the diff before applying.
```

**Clean up commit history**
```
My branch <BRANCH> has messy commits. Propose a clean, logical commit
structure (what to squash/reword) before I rebase. Don't rewrite history
until I confirm the plan.
```

---

### Phase 12 — Build, CI/CD & deploy

**Set up a CI pipeline**
```
Set up CI on <PLATFORM, e.g. GitHub Actions> that runs <install, lint,
typecheck, test, build> on every PR. Match our scripts. Show me the config
before adding it.
```

**Fix failing CI**
```
CI is failing on <JOB/STEP> with the log below. Find the cause and fix it.
If it passes locally but fails in CI, focus on the env/config differences.

<PASTE CI LOG>
```

**Dockerize the app**
```
Write a Dockerfile for <APP> that builds and runs it for production. Multi-
stage, small image, non-root user. Add a .dockerignore. Explain the choices.
```

**Optimize build / bundle size**
```
Our <build is slow / bundle is large (<SIZE>)>. Find the biggest contributors
and propose fixes ranked by impact. Show me before changing anything.
```

**Configure deployment**
```
Set up deployment to <PLATFORM> for <ENV>. Cover build command, output, env
vars, and any platform config. Follow our existing deploy setup. Don't deploy —
just configure and explain.
```

**Manage env vars / secrets**
```
I need <VAR(S)> available in <ENV>. Wire it through our config system
(<SOURCE>) without hardcoding secrets anywhere. Tell me what I need to set
and where. Don't print secret values.
```

**Set up preview / staging**
```
Set up <preview deployments per PR / a staging environment> on <PLATFORM>
mirroring production config but pointing at <STAGING RESOURCES>. Explain the
setup.
```

**Roll back a deploy**
```
We need to roll back <ENV> to <previous good version>. Walk me through the
safest rollback steps and what to verify after. Don't run anything destructive
without confirming.
```

---

### Phase 13 — Operate & maintain (production)

**Investigate a production incident**
```
Production incident: <SYMPTOM + IMPACT + when it started>. Help me triage:
ranked likely causes, what to check first, and the safest mitigation now.
Proper fix after. <PASTE logs/metrics/errors>
```

**Add logging / tracing**
```
Add structured logging to <AREA> so I can debug <PROBLEM> in production.
Log the right context (ids, timings, outcomes) without leaking PII/secrets.
Follow our existing logging pattern.
```

**Add monitoring / alerting**
```
Set up monitoring for <METRIC/CONDITION> so we get alerted when <THRESHOLD>.
Use <TOOL>. Explain what it watches and why that threshold.
```

**Hotfix to production**
```
Urgent: <BUG + IMPACT>. I need the smallest, safest hotfix. Find the root
cause, make the minimal change, and tell me exactly what to test before
shipping. Don't refactor.
```

**Fix a security vulnerability (CVE / advisory)**
```
We have a <CVE/advisory> in <PACKAGE>. Assess whether we're actually affected
(do we use the vulnerable path?), then fix it with the minimal safe upgrade or
mitigation. Verify the build/tests pass.
```

**Upgrade dependencies safely**
```
Upgrade our dependencies. Group them: safe patch/minor vs risky major. Do the
safe ones, run the tests, and give me a separate plan for each major upgrade.
Don't do majors without my go-ahead.
```

**Deprecate / sunset a feature**
```
We're removing <FEATURE>. Find everything tied to it (code, routes, DB, docs,
flags) and give me a safe removal plan in order. Don't delete anything yet —
show me the full impact first.
```

**Capacity / scaling review**
```
We expect <LOAD CHANGE, e.g. 10x traffic>. Review <SYSTEM/AREA> for what
breaks first (queries, memory, connections, rate limits) and what to fix.
Be realistic for our actual scale: <SCALE>. No changes yet.
```

---

## 6. Working-style modifiers

Drop any of these onto the **end of any prompt** to steer *how* Claude Code works:

```
Ask me clarifying questions before you start.
```
```
Plan it first — don't edit any files yet.
```
```
Make the smallest change that works.
```
```
Don't add try/catch, fallbacks, or null-checks to hide the problem — fix the root cause.
```
```
Show me the diff and explain it before applying.
```
```
Reuse existing utilities and patterns; don't reinvent or add new dependencies.
```
```
Verify by running <COMMAND> and report the actual result.
```
```
Stop and ask me before doing anything irreversible.
```
```
Keep going until it's fully done and verified — don't hand it back half-finished.
```
```
Explain your reasoning as you go so I can follow the decisions.
```

---

## 7. What good context looks like

A generic prompt gets generic code. The fastest way to get code that fits *your* repo is to **point at the convention, name the file to reuse, and state the hard constraints** right in the prompt. Examples (replace the names with your repo's real ones):

**Point it at the convention:**
```
Add an `archived_at` column to the orders table. First read docs/conventions.md
and follow the migration rules there. Make it reversible. Show me the migration
before applying.
```

**Reuse over reinvent — name the helper:**
```
I need to fetch a user's active subscriptions. Don't write a raw query — use the
existing helpers in src/lib/users.ts (getUserById / getActiveSubscriptions).
```

**State the hard constraint:**
```
Update the data-export logic. Constraint: do NOT import anything from the
legacy/ folder — it's deprecated. Use the new module in src/export/ instead.
```

**Encode the project rule in the ask:**
```
Add the new "Save changes" button label. Add the key to locales/en.json first
(English), then use our translation hook in the component. Don't hardcode the
string.
```

> The pattern transfers to any codebase: the more you encode *your* rules into the prompt, the less you'll need to fix afterward.

---

## 8. Advanced ways to drive Claude Code

Once the basics are second nature, these unlock bigger work:

| Capability | What it's for | How to invoke / ask |
|---|---|---|
| **Plan mode** | Large/uncertain changes — design + approve before any edits | Enter plan mode, or "plan this, don't edit yet" |
| **Subagents** | Delegating focused sub-tasks (explore, review, build) in parallel | "Spin up agents to explore X, Y, Z in parallel" |
| **Multi-agent workflows** | Big sweeps: audit/migrate/review across many files with fan-out + verification | Ask to "run a workflow" for large, parallelizable work |
| **Slash commands & skills** | Prebuilt, repeatable flows (`/code-review`, `/test-coverage`, project commands) | Type `/` to browse; invoke by name |
| **MCP servers** | Let it reach external tools/data (DBs, browsers, APIs, design tools) | Ask it to use the connected tool by name |
| **CLAUDE.md & memory** | Persisting conventions, gotchas, and preferences across sessions | "Add this to CLAUDE.md" / "remember that we…" |
| **Model & effort selection** | Matching horsepower to the task (quick fix vs deep design) | Pick the model/mode; or "think hard about this first" |
| **@-references & pasted artifacts** | Highest-signal input: exact files, full errors, screenshots, logs | Reference the path; paste the whole thing |

**Rule of thumb:** small/clear task → just ask. Big/uncertain/multi-file task → **plan or fan out first**, then execute.

---

*Tip: keep this open in a tab. After a week of grabbing templates, the structure (Context + Goal + Constraints + Verify) becomes automatic — and you won't need it anymore.*
