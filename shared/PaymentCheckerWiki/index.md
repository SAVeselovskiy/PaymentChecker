---
title: Wiki Index
updated: 2026-04-25
---

# PaymentChecker Wiki — Index

Catalog of all pages, organized by category. One-line summary per page.

---

## Architecture

| Page | Summary |
|------|---------|
| [[overview]] | High-level system design: what PaymentsChecker is and how its parts fit together |
| [[monorepo-structure]] | Monorepo layout — packages, git submodule setup, shared layer |
| [[agent-system]] | Claude agent orchestration — roles, delegation rules, execution order |

## Backend

| Page | Summary |
|------|---------|
| [[backend-overview]] | Go bot entry point, App struct, startup sequence, package layout |
| [[handlers]] | All Telegram command handlers — what each command does and how it's wired |
| [[parsing]] | Regex-based input parsers for spending amounts and free-text format |
| [[storage]] | SQLite storage layer — Store struct, CRUD operations, query patterns |
| [[scheduler]] | Background weekly report scheduler — Monday 09:00 push logic |

**New packages (web vertical slice):**
- `internal/http/` — HTTP server, `ServeMux` routes, JWT middleware, auth handlers
- `internal/webfs/` — `//go:embed web/out` FS; SPA fallback handler serving the Next.js static export

## Features

| Page | Summary |
|------|---------|
| [[add-spending]] | `/add` command — guided two-step flow for logging a spending entry |
| [[reports]] | Period report commands — `/today`, `/week`, `/month`, `/year`, `/stats` |
| [[free-text-input]] | Legacy free-text format `Category: amount` for quick entry |
| [[clear]] | `/clear` command — delete all user data with confirmation step |

## Data

| Page | Summary |
|------|---------|
| [[schema]] | SQLite schema — `spendings` table DDL, index, field semantics |

## Deployment

| Page | Summary |
|------|---------|
| [[docker]] | Docker and Docker Compose setup — image layers, volume, restart policy |
| [[environment]] | All environment variables — required vs optional, defaults, purpose |
| [[build-deploy]] | Build script and VPS deploy process — `build.sh` walkthrough |

## Decisions

| Page | Summary |
|------|---------|
| [[adr-001-sqlite]] | ADR-001: SQLite chosen as the database — rationale and trade-offs |
| [[adr-002-pure-go]] | ADR-002: Pure-Go SQLite driver (no CGO) — portability rationale |
| [[adr-003-long-polling]] | ADR-003: Long polling over webhooks — simplicity for self-hosted deploy |
| [[adr-004-http-api-and-telegram-login]] | ADR-004: HTTP API surface and Telegram Login Widget auth (JWT cookie) |
| [[adr-005-static-export-embedded-in-go]] | ADR-005: Next.js static export embedded in Go binary — single-binary deploy |

## Roadmap

| Page | Summary |
|------|---------|
| [[roadmap-overview]] | Planned work: website, mobile app, HTTP API, shared contracts |
