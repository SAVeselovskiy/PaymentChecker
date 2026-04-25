---
title: ADR-004 — HTTP API surface and Telegram Login Widget auth
category: decisions
tags: [adr, api, auth, telegram, jwt]
related: [[adr-001-sqlite]], [[adr-002-pure-go]], [[adr-003-long-polling]], [[backend-overview]]
updated: 2026-04-25
---

# ADR-004 — HTTP API Surface and Telegram Login Widget Auth

## Status

Accepted

## Date

2026-04-25

## Context

The PaymentChecker backend was Telegram-only: all user interaction happened through bot commands, and no HTTP interface existed. To enable a website (and later a mobile app), the backend needed an HTTP API and a way to authenticate users without introducing a separate identity system. Users already exist as Telegram identities — every stored spending record is keyed by `telegram_id`.

Introducing a new identity provider (email/password, OAuth with Google, etc.) would have added account-management surface area and required users to maintain a second credential.

## Decision

**Add an HTTP server alongside the bot in the same Go process under `errgroup`.**

The HTTP server and the Telegram long-polling loop run as two goroutines coordinated by `errgroup`. Either failing shuts down the process cleanly.

**Expose 4 endpoints, defined in `shared/api/openapi.yaml` v0.1.1:**

| Method | Path | Auth required |
|--------|------|---------------|
| `POST` | `/api/auth/telegram` | No |
| `POST` | `/api/auth/logout` | No |
| `GET` | `/api/me` | Yes (session cookie) |
| `GET` | `/api/spendings/today` | Yes (session cookie) |

**Auth: Telegram Login Widget.**

The frontend embeds the Telegram Login Widget. On successful widget interaction, the browser receives a signed payload from Telegram containing `id`, `first_name`, `last_name`, `username`, `photo_url`, and `auth_date`. The backend verifies this payload via `HMAC_SHA256(data_check_string, SHA256(bot_token))` — the same mechanism described in the Telegram Bot API docs. Payloads older than 24 hours are rejected. On success, the backend issues a 7-day JWT in a cookie:

```
Set-Cookie: session=<jwt>; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=604800
```

**Use stdlib `net/http.ServeMux` (Go 1.22+ method+path patterns) — no third-party router.**

Go 1.22 introduced `METHOD /path` patterns to `ServeMux`, making a library like `chi` unnecessary for 4 routes.

**Persist Telegram profile fields in a new `users` table.**

A minimal `users` table stores `telegram_id`, `first_name`, `last_name`, `username`, `photo_url`, and `created_at`. This is the minimum needed for `GET /api/me` to return contract-conforming data. Users are upserted on first login.

## Consequences

- **Single binary still** — no new deploy artifacts. The port count goes from 0 (bot only, no HTTP) to 1 (`HTTP_PORT`, default `8080`).
- **`JWT_SECRET` becomes a required env var** (minimum 32 bytes). `config.Load` fails fast if absent or too short.
- **`BOT_TOKEN` is now dual-purpose** — Telegram long-polling and HMAC verification for Login Widget payloads.
- **Domain registration required** — Telegram Login Widget requires a domain registered via BotFather `/setdomain`. Development workflow uses a Cloudflare Tunnel to expose localhost under a real domain; production uses the real VPS domain.
- **`users` table added** — first login persists the user's Telegram profile. No GDPR concern beyond what the bot already stored (it already had `telegram_id` in the `spendings` table).
- **Day-boundary consistency** — `/api/spendings/today` uses `cfg.Location` (the same `time.Location` as the bot's `/today` command) so results are identical between the bot and the API.

## Alternatives Considered

**chi router** — rejected. Go 1.22 stdlib patterns are sufficient for 4 routes. Adding a dependency purely for routing is not justified at this scale.

**Server-side session store** — rejected. Stateless JWT in an HttpOnly cookie is simpler, survives backend restarts without session drain, and requires no additional storage. The 7-day TTL is an acceptable tradeoff.

**Magic-link auth** — rejected. Reusing the existing Telegram identity is more natural (users are already in Telegram), requires no email infrastructure, and reduces account-management surface to zero.
