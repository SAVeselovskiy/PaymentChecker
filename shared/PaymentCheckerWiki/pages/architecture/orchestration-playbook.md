---
title: Orchestration Playbook
category: architecture
tags: [orchestration, claude, workflow, agents]
related: [[agent-system]], [[monorepo-structure]]
updated: 2026-04-25
---

# Orchestration Playbook

This page is the playbook **Claude (in the main thread)** follows when coordinating multi-step or multi-package work. There is no `orchestrator` subagent — coordination is performed inline by Claude using the `Agent` tool to spawn specialist subagents.

> [!note] Why no orchestrator subagent?
> An earlier design defined `.claude/agents/orchestrator.md` as a delegating subagent. In practice the runtime did not (or could not reliably) grant the `Agent` tool to a subagent, and the orchestrator looped on self-doubt about its own tool availability. The playbook content is preserved here; the executable agent file was removed.

## When to use this playbook

Not every prompt needs the full orchestration flow. Use this taxonomy to decide.

### Tasks that NEED the flow

Run Step 0 → wiki context → specialist subagent(s) → reviewer-agent → commit → wiki LOG when ANY of the following is true:

- Touches more than one package (e.g. "add an endpoint and a button for it")
- Changes or adds an API contract in `shared/api/`
- Changes observable behaviour of a shipped feature (bug fix that alters output, new field, removed field, renamed command)
- Adds a new dependency, env var, or deploy config (touches `go.mod`, `package.json`, `Dockerfile`, `docker-compose.yml`)
- Introduces a new pattern ("add caching", "switch auth model", "add a background worker")
- Touches storage schema (migrations, new table, new index)

Typical phrasing: *"add X feature"*, *"implement Y endpoint"*, *"ship Z"*, *"fix the bug where users see W"*.

### Tasks that DON'T need the flow

Skip the flow — go direct to a single specialist (or just answer inline). No review cycle, no wiki LOG:

- Pure documentation ("fill the README", "add a comment", "explain X in CLAUDE.md")
- Local exploration / questions ("how does X work", "where is Y defined", "show me the storage schema")
- Single-file refactor with no behaviour change (rename a variable, extract a helper used in one place)
- Throwaway / scratch work ("write a script to count X", "what's the diff between branches")
- Config tweaks the user controls (`.env.example` value, ESLint preference)
- Reading and reporting (reviewer-agent or general queries — read-only)
- Already-approved follow-ups when the original work is still in context ("commit and push" — same cycle, not a new one)

Typical phrasing: *"what is"*, *"where is"*, *"show me"*, *"rename"*, *"fix typo"*, *"fill README"*, *"tweak the X setting"*.

### Explicit tags the user can set in a prompt

When the lane is ambiguous, the user can override Claude's judgement with a tag:

| Tag | Meaning |
|-----|---------|
| `/quick` or *"just do it"* | Skip the orchestration flow even if the topic looks code-touching. |
| `/full` or *"full flow"* | Run the orchestration flow even for something small (useful for first-of-its-kind endpoints that deserve a contract + ADR). |
| `/plan` | Research and propose, no execution. |
| `/no-commit` | Run the flow but stop before commit so the user can inspect. |

### Default behaviour

If the user doesn't tag, Claude states the **complexity verdict** (Step 0) before doing the work, naming which lane it picked. The user can correct it with a one-word reply.

### Edge cases worth knowing

- **README and docs-only**: never trigger the flow, even at 200+ lines. Wiki LOG is also skipped — docs are not a behaviour change.
- **"Fix the failing test"**: depends. If the test is wrong, single-agent fix. If the code is wrong and the fix changes user-visible behaviour, full flow.
- **Anything affecting `shared/`**: always full flow — by definition it ripples to multiple packages.
- **Mid-task corrections** ("actually, change X to Y" while a flow is running): fold into the current cycle, do not start a new one.

## Step 0 — Complexity check (always first, always explicit)

Before any delegation, state your complexity verdict:

> **Complexity verdict:** [complex | simple] — [one sentence reason]

A task is **complex** if it meets one or more of these criteria:
- Touches more than one package (backend + web, backend + mobile, etc.)
- Requires a new API contract or changes an existing one in `shared/`
- Introduces a new architectural pattern or cross-cutting concern
- The implementation approach is non-obvious or has meaningful trade-offs

**Exception — routine endpoint additions:** adding a new endpoint that follows an established pattern in this codebase is **not** complex, even if it technically meets the criteria above. Only invoke `architector` if the approach is genuinely non-obvious or introduces a new design.

**If complex:** invoke the `architector` subagent (or the built-in `Plan` agent) via the `Agent` tool. It will return a draft plan with trade-offs and open questions inline. Present that plan to the user verbatim and wait for explicit approval before proceeding. Only after the user approves should you delegate. If the user requests changes, relay them back for a revised plan. Do not improvise beyond the approved plan.

**If simple** (single-package, clear scope, no design decisions, or any task explicitly exempted): skip the planning step and proceed with the delegation order. State why it qualifies as simple. The review cycle and wiki update still apply.

## Step 0.5 — Wiki context lookup

Before delegating to any specialist subagent:

1. Read `shared/PaymentCheckerWiki/index.md`.
2. Identify the pages whose summaries match the task domain — e.g. a backend storage change → `[[storage]]`, `[[schema]]`; a new feature → its feature page if it exists; an ADR-relevant decision → related ADR pages. Typically 1–3 pages, rarely more than 5.
3. Read each relevant page.
4. Include a **"Wiki context"** block in every specialist subagent prompt with the key facts pulled from those pages — a short bullet list, under 150 words. Synthesize only what's relevant to the task; do not dump raw page content.

Skip this step only if the wiki index has no pages that match the task domain.

## Delegation order

1. **architector** (or built-in `Plan` agent) — planning (complex tasks only)
2. **shared-agent** — update API contracts / types first (if needed)
3. **reviewer-agent** — review `shared/` changes before any downstream subagent starts (if shared-agent made contract changes)
4. **backend-agent** — implement server-side changes (if backend changes needed)
5. **web-agent** and/or **mobile-agent** — implement client-side changes in parallel (if client changes needed)
6. **reviewer-agent** — mandatory after any code changes (see review cycle below)
7. **shared-agent** — update wiki and append LOG entry after work is complete

## Rules

- Always delegate code changes to a specialist subagent. The main thread coordinates and reviews; specialists write.
- If a task requires API contract changes, always update `shared/` via `shared-agent` before touching backend or client code.
- After each major delegation step completes (architector, shared-agent, specialist subagents), summarise what it did and flag any conflicts or open questions. During the mandatory review cycle, complete the full cycle without returning to the user mid-loop.
- If a specialist subagent reports a blocker, surface it to the user before continuing.
- If an architector plan exists, follow it exactly. Do not add or skip steps without surfacing the reason to the user.

## Mandatory review cycle

After **any** backend-agent, web-agent, or mobile-agent run that produces changes, and after any shared-agent run that produces **contract** changes (not wiki updates), execute this cycle inline — do not return to the user first.

**If shared-agent made contract changes:** call reviewer-agent first (before any downstream subagent starts). Pass: which files in `shared/` were modified and the migration note. If the reviewer returns findings, send shared-agent to fix them and re-review until clean. Once the reviewer reports a clean pass, re-invoke shared-agent with an explicit instruction to commit the contract changes. Only then proceed to backend / client subagents.

**After any backend-agent, web-agent, or mobile-agent run:**

1. Call **reviewer-agent**, passing: which subagents made changes, which files were modified per subagent, and a summary of what was implemented. If multiple subagents ran in parallel, combine all their changes into a single reviewer-agent call by default — this lets the reviewer catch cross-package inconsistencies. Exception: if the combined change set is large (more than ~30 files), call reviewer-agent separately per subagent to keep the context manageable.
2. If the reviewer returns findings (any severity), dispatch fix requests to each subagent separately based on which files they own. Include exact file:line references from the review.
3. After the fix, call **reviewer-agent** again to confirm issues are resolved.
4. Repeat steps 2–3 until the reviewer reports no remaining issues.
5. Once the reviewer reports a clean pass, call each developer subagent that made changes to `git commit` and `git push` in its own repository. When web-agent and mobile-agent ran in parallel, call each separately — they work in separate git repos.
6. Call **shared-agent** to update the wiki and append a LOG entry. Pass it explicitly: the feature/task name, which subagents ran, key decisions made during planning, and the list of files changed per package.
7. Only then return to the user and report the task complete.

Do not skip this cycle even when the task seems small or low-risk.

## Pre-return checklist

Before sending the final response to the user, confirm every applicable item is done:

- [ ] **Step 0:** Complexity verdict stated explicitly
- [ ] **Step 0.5:** Wiki index read, relevant pages fetched, wiki context block included in each specialist subagent prompt
- [ ] **Architector / Plan invoked** (if complex), plan presented to user, and user explicitly approved before delegation
- [ ] **shared-agent** ran first (if API contracts changed)
- [ ] **reviewer-agent** reviewed `shared/` changes and returned a clean pass (if shared-agent made contract changes)
- [ ] **shared-agent** committed contract changes after clean review pass (if shared-agent made contract changes)
- [ ] **Specialist subagent(s)** completed their work with no blockers
- [ ] **Fix loop** completed (reviewer re-ran after each fix until clean)
- [ ] **reviewer-agent** ran and returned a clean pass on all code changes
- [ ] **git commit + git push** completed by each developer subagent that made changes, in its own repo
- [ ] **shared-agent** ran to update wiki and append LOG entry
- [ ] User-facing summary written: what changed, commit hash, any open items

## Related Pages

- [[agent-system]] — roster of subagents and their scopes
- [[monorepo-structure]] — package layout the subagents map to
