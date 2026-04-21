# CLAUDE.md — Project Orchestrator

This is a multi-package monorepo with three sub-projects: **backend**, **website**, and **mobile**.
The `shared/` folder holds API contracts and types used by all three.

## Structure

```
/
├── backend/       — API server
├── website/       — Web frontend
├── mobile/        — Mobile app
├── shared/        — API contracts, schemas, shared types
└── .claude/agents/
    ├── orchestrator.md
    ├── architector.md
    ├── backend-agent.md
    ├── web-agent.md
    ├── mobile-agent.md
    ├── reviewer-agent.md
    └── shared-agent.md
```

## Orchestration rules

- All tasks — single-package or multi-package — go through the **orchestrator**. It handles complexity assessment (Step 0), delegates to the right specialist agents, runs the review cycle, and triggers the wiki update. Never invoke a specialist agent directly, bypassing the orchestrator.
- API contracts in `shared/` must be updated via the **shared-agent** before downstream packages implement them.
- All code changes go through **reviewer-agent** before git commit and push.
- **After any significant work** (new feature, architecture decision, deployment change, behavioral bug fix), invoke **shared-agent** to update the wiki and log the decision.

## Agent delegation map

| Task type                        | Agent(s)                                                                        |
|----------------------------------|---------------------------------------------------------------------------------|
| Multi-package feature            | orchestrator → architector (plan) → shared-agent (contract) → reviewer-agent → shared-agent (commit) → specialist agents → reviewer-agent → shared-agent (wiki log) |
| New endpoint / schema change     | orchestrator → shared-agent → reviewer-agent → backend-agent + web-agent/mobile-agent → reviewer-agent → shared-agent (wiki log) |
| Backend-only bug                 | orchestrator → backend-agent → reviewer-agent → shared-agent (wiki log)         |
| UI change with no API impact     | orchestrator → web-agent or mobile-agent → reviewer-agent → shared-agent (wiki log) |
| Architecture / tech decision     | orchestrator → architector (plan) → shared-agent (ADR + wiki log)               |
| Code review / quality gate       | reviewer-agent (safe to invoke directly — read-only, no side effects)           |
| Wiki update / knowledge log      | shared-agent (safe to invoke directly — docs only, no code changes)             |

## Go toolchain

Go lives at `/usr/local/go/bin/go` (not on PATH by default).
