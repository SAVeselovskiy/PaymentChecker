# CLAUDE.md — Project Orchestration

This is a multi-package monorepo with three sub-projects: **backend**, **website**, and **mobile**.
The `shared/` folder holds API contracts and types used by all three.

## Structure

```
/
├── backend/       — API server
├── website/       — Web frontend
├── mobile/        — Mobile app
├── shared/        — API contracts, schemas, shared types, wiki
└── .claude/agents/
    ├── architector.md
    ├── backend-agent.md
    ├── web-agent.md
    ├── mobile-agent.md
    ├── reviewer-agent.md
    └── shared-agent.md
```

## Orchestration model

**Claude (the main thread) orchestrates.** There is no `orchestrator` subagent — coordination happens inline. Claude assesses complexity, consults the wiki, dispatches specialist subagents in the right order, runs the review cycle, and triggers wiki updates.

The full playbook lives at [`shared/PaymentCheckerWiki/pages/architecture/orchestration-playbook.md`](shared/PaymentCheckerWiki/pages/architecture/orchestration-playbook.md). Read it when handling non-trivial work.

## Core rules

- **Claude (main thread)** drives orchestration. Specialist subagents do the writing.
- API contracts in `shared/` must be updated via the **shared-agent** before downstream packages implement them.
- All code changes go through **reviewer-agent** before git commit and push.
- After any significant work (new feature, architecture decision, deployment change, behavioral bug fix), invoke **shared-agent** to update the wiki and log the decision.

## Agent delegation map

| Task type                        | Sequence (driven by Claude in the main thread)                                  |
|----------------------------------|---------------------------------------------------------------------------------|
| Multi-package feature            | architector / Plan → shared-agent (contract) → reviewer-agent → shared-agent (commit) → specialist agents → reviewer-agent → shared-agent (wiki log) |
| New endpoint / schema change     | shared-agent → reviewer-agent → backend-agent + web-agent/mobile-agent → reviewer-agent → shared-agent (wiki log) |
| Backend-only bug                 | backend-agent → reviewer-agent → shared-agent (wiki log)                        |
| UI change with no API impact     | web-agent or mobile-agent → reviewer-agent → shared-agent (wiki log)            |
| Architecture / tech decision     | architector / Plan → shared-agent (ADR + wiki log)                              |
| Code review / quality gate       | reviewer-agent (safe to invoke directly — read-only, no side effects)           |
| Wiki update / knowledge log      | shared-agent (safe to invoke directly — docs only, no code changes)             |

## Go toolchain

Go lives at `/usr/local/go/bin/go` (not on PATH by default).
