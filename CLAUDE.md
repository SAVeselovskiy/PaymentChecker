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
    ├── backend-agent.md
    ├── web-agent.md
    ├── mobile-agent.md
    ├── reviewer-agent.md
    └── shared-agent.md
```

## Orchestration rules

- For cross-cutting tasks (API contract change, new feature spanning multiple packages), use the **orchestrator** agent.
- For work scoped to a single package, invoke the matching specialist agent directly.
- API contracts in `shared/` must be updated via the **shared-agent** before downstream packages implement them.
- All PRs touching public interfaces go through **reviewer-agent** before merge.

## Agent delegation map

| Task type                        | Agent            |
|----------------------------------|------------------|
| New endpoint / schema change     | shared-agent first, then backend-agent + web-agent/mobile-agent |
| Backend-only bug                 | backend-agent    |
| UI change with no API impact     | web-agent or mobile-agent |
| Multi-package feature            | orchestrator     |
| Code review / quality gate       | reviewer-agent   |

## Go toolchain

Go lives at `/usr/local/go/bin/go` (not on PATH by default).
