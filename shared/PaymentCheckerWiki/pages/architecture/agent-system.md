---
title: Agent System
category: architecture
tags: [claude, agents, orchestration, development-workflow]
related: [[monorepo-structure]], [[overview]], [[orchestration-playbook]]
updated: 2026-04-25
---

# Agent System

Development on this monorepo is assisted by a set of Claude AI subagents, each scoped to a specific package or concern. Subagents are defined as instruction files in `.claude/agents/`.

**Orchestration is performed by Claude in the main thread**, not by a subagent. See [[orchestration-playbook]] for the step-by-step coordination procedure.

## Subagent Roster

| Subagent | Scope | When to invoke |
|----------|-------|----------------|
| **architector** | Design and planning | Complex tasks spanning >1 package, new API patterns, breaking changes (the built-in `Plan` agent is an alternative) |
| **shared-agent** | `shared/` — contracts + wiki | API changes (runs BEFORE clients); also activated AFTER significant work to log decisions and update wiki |
| **backend-agent** | `backend/` | Go code: handlers, storage, scheduler, tests |
| **web-agent** | `website/` | Web frontend: components, pages, API integration |
| **mobile-agent** | `mobile/` | Mobile app: screens, navigation, platform code |
| **reviewer-agent** | Code review | Mandatory after any code change; gates all merges |

## Delegation Rules

- The main thread (Claude) coordinates; subagents write code.
- **shared-agent** runs first when an API contract is changing, and runs last after any significant work to log decisions and update the wiki.
- **reviewer-agent** is mandatory after every code change, no exceptions.
- For a backend-only bug: invoke **backend-agent** directly.
- For a UI change with no API impact: invoke **web-agent** or **mobile-agent** directly.

## Execution Order for a New Feature

```
1. architector        ← design & planning (if complex)
2. shared-agent       ← update API contracts
3. backend-agent      ← server-side implementation
4. web-agent          ← web client (parallel with mobile-agent)
   mobile-agent       ← mobile client (parallel with web-agent)
5. reviewer-agent     ← code review; fix loop until clean
6. shared-agent       ← wiki + LOG entry
```

## Subagent Files Location

```
.claude/agents/
├── architector.md
├── backend-agent.md
├── web-agent.md
├── mobile-agent.md
├── reviewer-agent.md
└── shared-agent.md
```

The monorepo-level `CLAUDE.md` (at the repo root) names the playbook the main thread follows. The detailed step-by-step coordination procedure lives at [[orchestration-playbook]].

> [!note] No orchestrator subagent
> An earlier design defined an `orchestrator` subagent that delegated to specialists. In practice the runtime did not reliably grant the `Agent` tool to a subagent, so the orchestrator could not actually spawn other agents and looped on self-doubt. Coordination is now performed inline by the main thread; the playbook is preserved at [[orchestration-playbook]].

## Related Pages

- [[orchestration-playbook]] — step-by-step procedure the main thread follows
- [[monorepo-structure]] — package layout the subagents map to
- [[overview]] — system context
