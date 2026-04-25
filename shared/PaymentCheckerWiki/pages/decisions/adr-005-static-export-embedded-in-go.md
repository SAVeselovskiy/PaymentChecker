---
title: ADR-005 — Next.js static export embedded in Go binary
category: decisions
tags: [adr, frontend, nextjs, deployment, embed]
related: [[adr-001-sqlite]], [[adr-002-pure-go]], [[adr-003-long-polling]], [[adr-004-http-api-and-telegram-login]]
updated: 2026-04-25
---

# ADR-005 — Next.js Static Export Embedded in Go Binary

## Status

Accepted

## Date

2026-04-25

## Context

A web frontend was added using Next.js 15 App Router with TypeScript. Two production deploy models were evaluated:

1. **`next start` runtime** — ship Next.js with a Node.js runtime in production; run it as a separate container or process alongside the Go backend.
2. **`output: 'export'` + `//go:embed`** — build Next.js to a static directory and have the Go backend serve those files directly; no Node.js in production.

The project runs on a single VPS with `docker compose`. Adding a Node.js runtime container would have meant two containers, two origins, CORS configuration, and cookie domain management.

## Decision

**Use Next.js static export embedded in the Go binary.**

- `next.config.ts` sets `output: 'export'`. `pnpm build` produces a static site in `PaymentChecker-frontend/out/`.
- A multi-stage Dockerfile builds the frontend first, then copies `out/` into `internal/webfs/web/out/` before `go build`.
- The Go backend embeds the directory with `//go:embed web/out` in `internal/webfs/`.
- The HTTP server routes:
  - `/api/*` → Go handlers (see [[adr-004-http-api-and-telegram-login]])
  - `/` (and everything else) → SPA fallback serving `index.html` from the embedded FS

One container, one binary, one origin — no CORS headers needed, no cookie domain configuration, no Node.js runtime in production.

## Consequences

**Aligned with existing minimalism decisions** — this follows the same philosophy as [[adr-001-sqlite]] (SQLite instead of Postgres), [[adr-002-pure-go]] (no CGO), and [[adr-003-long-polling]] (no webhook server). The entire stack deploys with `docker compose up`.

**Next.js server features are unavailable in production:**

- No SSR or React Server Component server-side data fetching
- No `middleware.ts` (which runs in the Node.js Edge Runtime)
- Auth redirects cannot happen server-side; they are handled in a client `<AuthGate>` component that checks the `/api/me` endpoint and redirects unauthenticated users to `/login`

These limitations are acceptable for the current feature set (dashboard + login page).

**Image size:** ~30 MB final Docker image, versus ~150 MB with a separate Node.js runtime. This matters for a single-VPS deployment.

**Frontend development is unaffected:** `pnpm dev` runs on `:3000` with `next.config.ts` rewrites proxying `/api/*` to the backend on `:8080`. Static export is only used at build time.

## Alternatives Considered

**Vite SPA** — strictly leaner bundle and simpler config, but would lose Next.js App Router conventions, the `@next/eslint-plugin-next` lint rules, and the Tailwind/TypeScript integration that the project was initialized with. Not worth the migration cost.

**`next start` runtime** — rejected for operational cost: a separate container, CORS configuration, cookie `Domain` alignment between the Go API origin and the Node origin, and Node.js version management on the VPS. The single-binary philosophy wins here.
