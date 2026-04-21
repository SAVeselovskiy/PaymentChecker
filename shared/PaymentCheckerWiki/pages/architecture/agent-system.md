---
title: Agent System
category: architecture
tags: [claude, agents, orchestration, development-workflow]
related: [[monorepo-structure]], [[overview]]
updated: 2026-04-21
---

# Agent System

Development on this monorepo is assisted by a set of Claude AI agents, each scoped to a specific package or concern. Agents are defined as instruction files in `.claude/agents/`.

## Agent Roster

| Agent | Scope | When to invoke |
|-------|-------|---------------|
| **orchestrator** | Cross-package coordination | Multi-package features; sequences work across specialists |
| **architector** | Design and planning | Complex tasks spanning >1 package, new API patterns, breaking changes |
| **shared-agent** | `shared/` API contracts | Any change to schemas, types, or constants — must run BEFORE backend/clients |
| **backend-agent** | `backend/` | Go code: handlers, storage, scheduler, tests |
| **web-agent** | `website/` | Web frontend: components, pages, API integration |
| **mobile-agent** | `mobile/` | Mobile app: screens, navigation, platform code |
| **reviewer-agent** | Code review | Mandatory after any code change; gates all merges |

## Delegation Rules

- Never write code in **orchestrator** — it only coordinates specialists
- **shared-agent** always runs first when an API contract is changing
- **reviewer-agent** is mandatory after every code change, no exceptions
- For a backend-only bug: invoke **backend-agent** directly (no orchestrator needed)
- For a UI change with no API impact: invoke **web-agent** or **mobile-agent** directly

## Execution Order for a New Feature

```
1. architector        ← design & planning (if complex)
2. shared-agent       ← update API contracts
3. backend-agent      ← server-side implementation
4. web-agent          ← web client (parallel with mobile-agent)
   mobile-agent       ← mobile client (parallel with web-agent)
5. reviewer-agent     ← code review; fix loop until clean
```

## Agent Files Location

```
.claude/agents/
├── orchestrator.md
├── backend-agent.md
├── web-agent.md
├── mobile-agent.md
├── reviewer-agent.md
└── shared-agent.md
```

The monorepo-level `CLAUDE.md` (at the repo root) defines the delegation map and orchestration rules that govern all agents.

## Related Pages

- [[monorepo-structure]] — package layout the agents map to
- [[overview]] — system context
