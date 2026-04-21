---
title: "ADR-003: Long Polling over Webhooks"
category: decisions
tags: [adr, telegram, long-polling, webhooks, networking]
related: [[backend-overview]], [[docker]]
updated: 2026-04-21
---

# ADR-003: Long Polling over Webhooks

**Status:** Accepted

## Context

The Telegram Bot API supports two update delivery methods:

- **Long polling** — the bot repeatedly calls `getUpdates`; Telegram holds the connection open until messages arrive
- **Webhooks** — Telegram pushes updates to a public HTTPS endpoint on the bot's server

## Decision

Use **long polling** via `go-telegram/bot`'s built-in poller.

## Rationale

- **No public HTTPS endpoint required** — long polling works behind NAT, firewalls, and without a TLS certificate or reverse proxy
- **Simpler VPS setup** — `docker compose up` is the entire deploy; no Nginx/Caddy, no Let's Encrypt, no domain name required
- **Fits the scale** — a personal spending tracker has low message volume; the ~1 second latency of long polling is imperceptible to users
- **Easier local development** — works on a laptop without exposing a port to the internet

## Trade-offs

| Pro | Con |
|-----|-----|
| No HTTPS/domain/reverse proxy needed | Slightly higher latency than webhooks (~1s) |
| Works anywhere (NAT, local dev) | Bot must actively poll; extra outbound connection |
| Simpler infrastructure | Cannot receive updates if the process is down (vs. webhook queuing) |

## Consequences

- The bot process must stay running to receive messages (`restart: unless-stopped` in Docker covers crashes)
- If the process is down for an extended period, missed messages are delivered once polling resumes (Telegram queues them)
- If the app ever needs sub-second latency or a multi-replica deployment, webhooks would be the right migration

## Related Pages

- [[backend-overview]] — where `go-telegram/bot` is initialized with long polling
- [[docker]] — `restart: unless-stopped` ensures the poller stays alive
