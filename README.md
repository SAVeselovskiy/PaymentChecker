# PaymentsChecker — Agent Orchestration System

Multi-package monorepo with an AI agent orchestration layer. All development tasks — from single bug fixes to cross-platform features — are routed through a structured agent pipeline that enforces contract-first design, mandatory code review, and automatic knowledge logging.

---

## Repository Structure

```
/
├── backend/                  — Go API server (own git repo)
├── website/                  — Web frontend (own git repo)
├── mobile/                   — Mobile app (own git repo)
├── shared/                   — API contracts, schemas, shared types + wiki
│   └── PaymentCheckerWiki/   — Obsidian knowledge vault
└── .claude/agents/           — Agent instruction files
    ├── orchestrator.md
    ├── architector.md
    ├── backend-agent.md
    ├── web-agent.md
    ├── mobile-agent.md
    ├── reviewer-agent.md
    └── shared-agent.md
```

> `backend/`, `website/`, and `mobile/` are each **separate git repositories**. `shared/` is tracked in the root repo.

---

## Agent Roster

| Agent | Role | Writes to | Can read |
|---|---|---|---|
| **orchestrator** | Routes all tasks, enforces the pipeline | — (no file writes) | everything |
| **architector** | Produces implementation plans for complex tasks | — (no file writes) | everything |
| **shared-agent** | API contracts + wiki knowledge base | `shared/` only | everywhere |
| **backend-agent** | Go API server implementation | `backend/` only | `shared/` (read-only) |
| **web-agent** | Web frontend implementation | `website/` only | `shared/` (read-only) |
| **mobile-agent** | Mobile app implementation | `mobile/` only | `shared/` (read-only) |
| **reviewer-agent** | Read-only quality gate | — (no file writes) | everything |

---

## Execution Flows

### Step 0 — Complexity Check (always first)

Every task begins here. The orchestrator classifies the task before any delegation.

```
                        ┌─────────────────────────────────┐
                        │           User request           │
                        └─────────────────┬───────────────┘
                                          │
                                          ▼
                              ┌───────────────────────┐
                              │      orchestrator      │
                              │   Step 0: classify     │
                              └───────────┬───────────┘
                                          │
               ┌──────────────────────────┴──────────────────────────┐
               │                                                       │
               ▼                                                       ▼
     ┌──────────────────┐                                   ┌──────────────────┐
     │     COMPLEX      │                                   │     SIMPLE       │
     │                  │                                   │                  │
     │ • Multi-package  │                                   │ • Single-package │
     │ • New/changed    │                                   │ • Clear scope    │
     │   API contract   │                                   │ • No design      │
     │ • New arch.      │                                   │   decisions      │
     │   pattern        │                                   │                  │
     │ • Non-obvious    │                                   │ Also: routine    │
     │   approach       │                                   │ endpoint adding  │
     │                  │                                   │ (exception)      │
     └────────┬─────────┘                                   └────────┬─────────┘
              │                                                       │
              ▼                                                       ▼
      invoke architector                               skip architector, go to
      (planning phase)                                 delegation step 2+
```

---

### Flow A — Complex Multi-Package Feature

Example: *"Add real-time payment notifications to backend, web, and mobile"*

```
  User
   │
   │  request
   ▼
┌──────────────┐
│ orchestrator │  Step 0: COMPLEX
└──────┬───────┘
       │
       │ spawn
       ▼
┌──────────────┐
│  architector │  explores codebase → produces structured plan
│  (opus)      │  (approach, steps per agent, open questions)
└──────┬───────┘
       │
       │ plan returned verbatim
       ▼
┌──────────────┐
│ orchestrator │──────────────────────────────────► User
└──────┬───────┘  presents plan, waits for approval
       │
       │ user approves
       ▼
┌──────────────┐
│ shared-agent │  defines types + endpoint in shared/api/ and shared/types/
│              │  updates shared/CHANGELOG.md
│              │  does NOT commit — returns migration note
└──────┬───────┘
       │
       │ spawn (contract review)
       ▼
┌──────────────┐   issues?
│reviewer-agent│──────────────► shared-agent (fix) ──► reviewer-agent
│              │                      ▲                        │
└──────┬───────┘                      └────────────────────────┘
       │ clean pass                        (loop until clean)
       │
       │ orchestrator re-invokes shared-agent
       ▼
┌──────────────┐
│ shared-agent │  commits contract changes (git commit + push, shared/ repo)
└──────┬───────┘
       │
       │ spawn in parallel
       ├─────────────────────┬─────────────────────┐
       ▼                     ▼                     ▼
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│backend-agent│     │  web-agent   │     │ mobile-agent │
│             │     │              │     │              │
│ implements  │     │ implements   │     │ implements   │
│ endpoint,   │     │ UI + API     │     │ UI + API     │
│ DB, tests   │     │ client       │     │ client       │
│ no commit   │     │ no commit    │     │ no commit    │
└──────┬──────┘     └──────┬───────┘     └──────┬───────┘
       └────────────────────┴────────────────────┘
                            │
                            │ spawn (combined code review, all 3 agents)
                            │ (split per agent if > ~30 files total)
                            ▼
                  ┌──────────────────┐   issues?
                  │  reviewer-agent  │──────────────► owning agent (fix)
                  │                  │                      │
                  └────────┬─────────┘◄─────────────────────┘
                           │ clean pass     (loop until clean)
                           │
                           │ orchestrator invokes each separately
                           ├──────────────────────────────────────────┐
                           ▼                                          ▼
                ┌──────────────────┐                       ┌──────────────────┐
                │  backend-agent   │                       │  web-agent  /    │
                │  git commit+push │                       │  mobile-agent    │
                │  (backend/ repo) │                       │  git commit+push │
                └──────────────────┘                       │  (own repos)     │
                                                           └──────────────────┘
                           │
                           │ spawn
                           ▼
                  ┌──────────────────┐
                  │  shared-agent    │  updates wiki pages
                  │                  │  appends LOG entry to log.md
                  │                  │  commits immediately (wiki repo)
                  └────────┬─────────┘
                           │
                           ▼
                          User  ◄──── orchestrator: summary, commit hashes, open items
```

---

### Flow B — Simple Single-Package Task

Example: *"Fix pagination bug in the backend payment list endpoint"*

```
  User
   │
   │  request
   ▼
┌──────────────┐
│ orchestrator │  Step 0: SIMPLE — single-package, no API change, clear scope
└──────┬───────┘
       │
       │ spawn
       ▼
┌──────────────┐
│backend-agent │  reads shared/ for context, implements fix, runs tests
│              │  does NOT commit
└──────┬───────┘
       │
       │ spawn
       ▼
┌──────────────┐   issues?
│reviewer-agent│──────────────► backend-agent (fix) ──► reviewer-agent
│              │                      ▲                        │
└──────┬───────┘                      └────────────────────────┘
       │ clean pass
       │
       │ orchestrator invokes
       ▼
┌──────────────┐
│backend-agent │  git commit + push  (backend/ repo)
└──────┬───────┘
       │
       │ spawn
       ▼
┌──────────────┐
│ shared-agent │  updates wiki, appends LOG entry, commits
└──────┬───────┘
       │
       ▼
      User
```

---

### Flow C — Routine Endpoint Addition (Exception Path)

Example: *"Add GET /payments/export endpoint (follows existing endpoint pattern)"*

This is multi-package and has an API contract change, but qualifies as a **routine endpoint addition** — the exception block kicks in, architector is skipped.

```
  User
   │
   ▼
┌──────────────┐
│ orchestrator │  Step 0: SIMPLE via exception — routine endpoint, established pattern
└──────┬───────┘  (multi-package + new contract technically met, but exception applies)
       │
       ▼
┌──────────────┐
│ shared-agent │  defines endpoint + types in shared/
│              │  does NOT commit
└──────┬───────┘
       │
       ▼
┌──────────────┐
│reviewer-agent│  reviews contract ──► fix loop if needed ──► clean pass
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ shared-agent │  commits contract
└──────┬───────┘
       │
       ├──────────────────────┬──────────────────────┐
       ▼                      ▼                      ▼
┌─────────────┐      ┌──────────────┐      ┌──────────────┐
│backend-agent│      │  web-agent   │      │ mobile-agent │
└──────┬──────┘      └──────┬───────┘      └──────┬───────┘
       └────────────────────┴────────────────────┘
                            │
                            ▼
                  ┌──────────────────┐
                  │  reviewer-agent  │  fix loop if needed
                  └────────┬─────────┘
                           │ clean pass
                           ▼
                  each agent: git commit + push (own repo)
                           │
                           ▼
                  shared-agent: wiki update + LOG
                           │
                           ▼
                          User
```

---

### Flow D — Architecture / Tech Decision (No Code)

Example: *"Decide on authentication strategy: JWT vs sessions"*

```
  User
   │
   ▼
┌──────────────┐
│ orchestrator │  Step 0: COMPLEX — non-obvious approach, meaningful trade-offs
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  architector │  researches options, presents trade-offs, recommends one
└──────┬───────┘
       │ plan returned
       ▼
┌──────────────┐
│ orchestrator │──────────────────────────────────► User (approves decision)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ shared-agent │  writes ADR to wiki, updates relevant pages, appends LOG
│              │  commits immediately (no contract change, wiki only)
└──────┬───────┘
       │
       ▼
      User
```

> No reviewer-agent in this flow — nothing to review when no code or contract changed.

---

## Mandatory Review Cycle (Detail)

The review cycle runs automatically after any code or contract change. The orchestrator never returns to the user mid-cycle.

```
                    ┌─────────────────────────────────────┐
                    │        MANDATORY REVIEW CYCLE        │
                    └─────────────────┬───────────────────┘
                                      │
              ┌───────────────────────┴───────────────────────┐
              │                                               │
              ▼                                               ▼
   After shared-agent contract                   After backend/web/mobile
   changes (before specialist agents)            agent code changes
              │                                               │
              ▼                                               ▼
   ┌──────────────────────┐                  ┌──────────────────────────────┐
   │    reviewer-agent    │                  │       reviewer-agent         │
   │  reviews shared/     │                  │  reviews all changed code    │
   │  contract files      │                  │                              │
   │  only                │                  │  combined call by default    │
   └──────────┬───────────┘                  │  (split if > ~30 files)     │
              │                              └──────────────┬───────────────┘
              │                                             │
     ┌────────┴─────────┐                        ┌─────────┴──────────┐
     │                  │                        │                    │
   issues?           clean                    issues?              clean
     │                  │                        │                    │
     ▼                  ▼                        ▼                    ▼
shared-agent        shared-agent          owning agent(s)     each developer
 fixes it           commits               fix their files      agent commits
     │              contract                    │              + pushes in
     │                  │                       │              own repo
     └──► re-review ◄───┘              └──► re-review              │
          (loop)                             (loop)                 │
                                                                    ▼
                                                            shared-agent
                                                            wiki update
                                                            + LOG entry
```

---

## Task Type Quick Reference

| Task | Architector | shared-agent (contract) | backend | web | mobile | reviewer calls |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| Multi-package feature | ✓ | ✓ | ✓ | ✓ | ✓ | 2 (contract + code) |
| New endpoint (routine) | — | ✓ | ✓ | ✓ | ✓ | 2 (contract + code) |
| Backend-only bug | — | — | ✓ | — | — | 1 (code) |
| Web UI change | — | — | — | ✓ | — | 1 (code) |
| Mobile UI change | — | — | — | — | ✓ | 1 (code) |
| Architecture decision | ✓ | — (ADR only) | — | — | — | — |
| Wiki / knowledge log | — | ✓ (wiki only) | — | — | — | — |

> shared-agent always runs last in every flow to update the wiki and append a LOG entry.

---

## Review Checklist (What reviewer-agent checks)

| Category | Checks |
|---|---|
| **Correctness** | Logic vs requirements, edge cases, error paths, off-by-one |
| **API contracts** | Backward-compatible changes in `shared/`, client code matches contract |
| **Security** | No hardcoded secrets, input validation at boundaries, no injection vectors |
| **Naming** | All variable names ≥ 3 chars (exceptions: `ctx` and `ok` in Go only) |
| **Quality** | No dead code, consistent naming, tests cover new behaviour |
| **Godoc (Go only)** | Every new/changed exported identifier has an accurate Godoc comment |

Findings are reported as **critical / major / minor** with exact `file:line` references.
