---
title: Roadmap Overview
category: roadmap
tags: [roadmap, planned, website, mobile, api, deployment, ops]
related: [[overview]], [[monorepo-structure]], [[agent-system]], [[adr-004-http-api-and-telegram-login]], [[adr-005-static-export-embedded-in-go]], [[telegram-auth-ux]], [[frontend-standalone-deploy]], [[database-backups]]
updated: 2026-04-26
---

# Roadmap Overview

The backend Telegram bot is production-ready. The first cross-package vertical slice has shipped: HTTP API + Telegram Login Widget auth + Next.js web frontend. Mobile remains TBD.

## Component Status

### 1. HTTP API (Backend Extension)

**Status: Vertical slice shipped (partial).**

The backend now exposes an HTTP server alongside the Telegram bot in the same Go process (see [[adr-004-http-api-and-telegram-login]]). The following endpoints are implemented and covered by the OpenAPI 0.1.1 contract in `shared/api/openapi.yaml`:

| Endpoint | Status |
|----------|--------|
| `POST /api/auth/telegram` | Implemented |
| `POST /api/auth/logout` | Implemented |
| `GET /api/me` | Implemented |
| `GET /api/spendings/today` | Implemented |
| `POST /api/spendings` (add spending) | Not yet implemented |
| `PUT /api/spendings/{id}` (edit) | Not yet implemented |
| `DELETE /api/spendings/{id}` (delete) | Not yet implemented |
| `GET /api/spendings/week` | Not yet implemented |
| `GET /api/spendings/month` | Not yet implemented |
| `GET /api/spendings/year` | Not yet implemented |

The contract lives in `shared/api/` and is versioned; downstream packages consume `@paymentchecker/types` from the pnpm workspace.

### 2. Website (`PaymentChecker-frontend/`)

**Status: Vertical slice shipped, but not production-ready. Deployment decoupled from the backend.**

- Stack: Next.js 15 App Router + TypeScript + Tailwind CSS + Recharts + TanStack Query
- Shipped pages:
  - `/login` — Telegram Login Widget, redirects to `/` on success
  - `/` — today's spending breakdown by category (bar chart + list)
- Auth: client-side `<AuthGate>` component checks `/api/me`; unauthenticated users are redirected to `/login`
- Future: history view, week/month/year charts, settings page, add-spending UI

> [!warning] Deploy model superseded
> [[adr-005-static-export-embedded-in-go]] originally embedded the Next.js
> static export inside the Go binary. As of 2026-04-26 the backend Docker
> image ships with a placeholder page only and the frontend is deployed
> independently — see [[frontend-standalone-deploy]]. ADR-005 is kept for
> historical context; a follow-up ADR will record the new deploy model
> once the standalone frontend image lands.

### Auth UX — Telegram app handoff

The website currently uses the Telegram Login Widget. The roadmap is to replace it with a bot deep-link flow that hands off to the installed Telegram app, avoiding the third-party phone-entry popup.

> [!note] Not yet started — deferred
> See [[telegram-auth-ux]] for options, recommendation, and implementation outline.

### 3. Mobile App (`PaymentChecker-mobile/`)

- Current state: placeholder (`README.md` only, no code)
- Planned: mirrors website features; native platform feel
- Stack: TBD (React Native or Flutter likely)
- Will consume the same HTTP API as the website

> [!note] Not yet started
> No mobile design or implementation work has been done.

### 4. Operations — Database Backups

**Status: Not yet implemented. Tracked in [[database-backups]].**

No automated backup mechanism exists for `spendings.db`. On 2026-04-26 the
production DB was lost during the VPS migration to the standalone-backend
deploy. Plan: daily on-host `sqlite3 .backup` rotation (v1), then off-box
copy to S3-compatible storage (v2). Pre-deploy snapshot in `build.sh`
should land alongside v1 so a bad migration is recoverable.

### 5. Shared Contracts (`shared/`)

- `shared/api/openapi.yaml` — OpenAPI 3.1 spec (current: v0.1.1)
- `shared/types/api.ts` — TypeScript types, generated from the spec
- `shared/types/package.json` — `@paymentchecker/types` workspace package
- Future: add spending mutation endpoints; consider Protobuf for mobile

## Development Order

```
HTTP API design (shared-agent)        ✓ done (v0.1.1)
    └─ Backend HTTP API (backend-agent)
        ├─ Auth + today endpoints     ✓ done (vertical slice)
        └─ CRUD + history endpoints   ← next
    └─ Website (web-agent)
        ├─ Login + today view         ✓ done (vertical slice)
        └─ History, charts, settings  ← next
    └─ Mobile app (mobile-agent)      ← TBD
```

## Related Pages

- [[monorepo-structure]] — current package layout
- [[agent-system]] — how agents handle cross-cutting features
- [[overview]] — current system architecture
- [[adr-004-http-api-and-telegram-login]] — HTTP API and auth decisions
- [[adr-005-static-export-embedded-in-go]] — frontend deployment model
