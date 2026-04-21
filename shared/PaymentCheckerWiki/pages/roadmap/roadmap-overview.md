---
title: Roadmap Overview
category: roadmap
tags: [roadmap, planned, website, mobile, api]
related: [[overview]], [[monorepo-structure]], [[agent-system]]
updated: 2026-04-21
---

# Roadmap Overview

The backend Telegram bot is production-ready. The planned next phases are a web frontend, a mobile app, and the HTTP API layer that connects them.

## Planned Components

### 1. HTTP API (Backend Extension)

Before website or mobile can be built, the backend needs to expose an HTTP API.

- Currently: bot talks only to Telegram (no HTTP endpoints)
- Needed: REST or GraphQL API for querying spendings, adding entries, managing account
- API contracts will live in `shared/api/` (OpenAPI specs or Protobuf)
- The [[agent-system]] requires `shared-agent` to define contracts before `backend-agent` implements them

> [!note] Not yet started
> No API design work has been done. This is the prerequisite for everything below.

### 2. Website (`website/`)

A web frontend for visualizing and managing spendings.

- Current state: placeholder (`README.md` only, no code)
- Planned: charts/graphs, category breakdown, date filters, spending history
- Stack: TBD (likely React/Next.js based on project conventions)
- Will consume the HTTP API once built

### 3. Mobile App (`mobile/`)

iOS and Android app for on-the-go spending entry and review.

- Current state: placeholder (`README.md` only, no code)
- Planned: mirrors website features; native platform feel
- Stack: TBD (React Native or Flutter likely)
- Will consume the same HTTP API as the website

### 4. Shared Contracts (`shared/`)

- `shared/api/` — OpenAPI specs or Protobuf for the HTTP API
- `shared/types/` — TypeScript types or JSON Schema for frontend use
- `shared/constants/` — shared enums (categories, currencies, etc.)

## Development Order

```
HTTP API design (shared-agent)
    └─ Backend HTTP API (backend-agent)
        ├─ Website (web-agent)
        └─ Mobile app (mobile-agent)
```

## Related Pages

- [[monorepo-structure]] — current package layout
- [[agent-system]] — how agents handle cross-cutting features
- [[overview]] — current system (bot-only)
