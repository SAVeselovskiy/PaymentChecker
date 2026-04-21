---
title: Monorepo Structure
category: architecture
tags: [monorepo, structure, git, submodules]
related: [[overview]], [[agent-system]]
updated: 2026-04-21
---

# Monorepo Structure

The repository root is a **workspace** that contains three sub-projects as separate git repos, plus a shared layer.

## Directory Layout

```
PaymentsChecker/          ← workspace root (this repo)
├── backend/              ← Go Telegram bot (separate git repo, submodule-style)
├── website/              ← Web frontend (separate git repo, placeholder)
├── mobile/               ← Mobile app (separate git repo, placeholder)
├── shared/               ← API contracts, types, wiki (tracked in root repo)
│   ├── README.md
│   └── PaymentCheckerWiki/   ← this Obsidian wiki
├── .claude/
│   └── agents/           ← Claude agent instruction files
├── CLAUDE.md             ← monorepo orchestration rules for Claude
└── .gitignore            ← excludes backend/, website/, mobile/
```

## Sub-project Status

| Package | Status | Language | Purpose |
|---------|--------|----------|---------|
| `backend/` | Production-ready | Go 1.25 | Telegram bot, SQLite storage |
| `website/` | Placeholder | TBD | Web frontend for spending data |
| `mobile/` | Placeholder | TBD | iOS/Android app |
| `shared/` | Framework only | — | API contracts, types, wiki |

## Git Topology

Each of `backend/`, `website/`, and `mobile/` is an **independent git repository**. The root `.gitignore` excludes them so git doesn't try to track them as nested repos. This is a manual submodule-like setup — not `git submodule`.

`shared/` is tracked in the root repo (not excluded).

## Shared Layer

`shared/` is intended to hold:

- `api/` — OpenAPI specs or Protobuf definitions
- `types/` — language-agnostic type definitions (JSON Schema or TypeScript)
- `constants/` — cross-package enums and constants
- `CHANGELOG.md` — contract change history

> [!note] Not yet implemented
> No active API contracts exist yet. The backend exposes no HTTP API — it only talks to Telegram. Contracts will be needed once website/mobile clients are built.

## Go Toolchain Note

Go is installed at `/usr/local/go/bin/go` and is **not on PATH** by default. All Go commands must use the full path:

```bash
/usr/local/go/bin/go test ./...
/usr/local/go/bin/go build ./cmd/bot
```

## Related Pages

- [[overview]] — system design
- [[agent-system]] — how Claude agents map to packages
- [[roadmap-overview]] — future website, mobile, and API contracts
