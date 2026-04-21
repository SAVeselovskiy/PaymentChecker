---
title: "ADR-002: Pure-Go SQLite Driver (No CGO)"
category: decisions
tags: [adr, sqlite, cgo, go, build]
related: [[adr-001-sqlite]], [[docker]], [[storage]]
updated: 2026-04-21
---

# ADR-002: Pure-Go SQLite Driver (No CGO)

**Status:** Accepted

## Context

Having chosen SQLite ([[adr-001-sqlite]]), we need a Go driver. The two main options are:

- `mattn/go-sqlite3` — CGO-based, most popular, requires a C compiler
- `modernc.org/sqlite` — pure Go (transpiled from SQLite's C source), no CGO

## Decision

Use **`modernc.org/sqlite`** (v1.48.2) with `CGO_ENABLED=0`.

## Rationale

- **Docker build simplicity** — no C toolchain needed in the builder image; `golang:1.25-alpine` is sufficient without `gcc`/`musl-dev`
- **Static binary** — `CGO_ENABLED=0` produces a fully statically linked binary that runs in a minimal `alpine:3.20` base without glibc
- **Cross-compilation** — pure-Go can cross-compile trivially; a CGO binary requires a cross-compiler toolchain
- **No functional difference** — for this workload (simple CRUD, low concurrency) there is no meaningful performance difference between the two drivers

## Trade-offs

| Pro | Con |
|-----|-----|
| Simpler Dockerfile (no C deps) | `modernc.org/sqlite` is less battle-tested than `mattn/go-sqlite3` |
| Statically linked binary | Slightly larger binary (transpiled C is verbose Go) |
| Easy cross-compilation | — |

## Consequences

- `CGO_ENABLED=0` is set in the Dockerfile build command
- The Go binary is statically linked and runs in pure Alpine without extra packages
- If switching to `mattn/go-sqlite3` in the future, the Dockerfile must add build dependencies and the binary will no longer be statically linked

## Related Pages

- [[docker]] — where `CGO_ENABLED=0` is set
- [[storage]] — imports `modernc.org/sqlite`
- [[adr-001-sqlite]] — the prior decision to use SQLite at all
