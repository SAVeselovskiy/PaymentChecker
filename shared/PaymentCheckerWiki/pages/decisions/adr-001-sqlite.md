---
title: "ADR-001: SQLite as Database"
category: decisions
tags: [adr, sqlite, database, architecture-decision]
related: [[schema]], [[storage]], [[adr-002-pure-go]]
updated: 2026-04-21
---

# ADR-001: SQLite as Database

**Status:** Accepted

## Context

The app needs to persist spending entries for multiple Telegram users. The backend is a single self-hosted process on a VPS with no managed database infrastructure.

## Decision

Use **SQLite** as the sole data store, with the database as a single file on disk.

## Rationale

- **Zero infrastructure** — no separate database process, no connection pooling, no cloud service
- **Single binary deploy** — the entire app (binary + DB file) lives on one VPS; `docker compose up` is the full deployment
- **Low write concurrency** — a personal spending tracker has negligible concurrent writes; SQLite's single-writer model is not a constraint
- **Sufficient query power** — the only non-trivial query is `SUM ... GROUP BY category WHERE user_id = ? AND created_at BETWEEN ? AND ?`, which SQLite handles with ease
- **Data locality** — the named Docker volume gives persistence; backups are a single file copy

## Trade-offs

| Pro | Con |
|-----|-----|
| No infra to manage | Not suitable if multiple write-heavy processes needed |
| Easy backup (one file) | No built-in replication or HA |
| Embedded — no network latency | Schema migrations require manual tooling |

## Consequences

- All data lives in `spendings.db` (or `DB_PATH`)
- Schema migrations have no framework — must be done manually (see [[schema]])
- If a future HTTP API needs high write concurrency, reconsider (PostgreSQL would be the natural migration target)

## Related Pages

- [[schema]] — the actual table definition
- [[storage]] — the Go storage layer built on SQLite
- [[adr-002-pure-go]] — follow-up decision on which SQLite driver to use
