---
title: System Overview
category: architecture
tags: [system-design, telegram-bot, spending-tracker]
related: [[monorepo-structure]], [[backend-overview]], [[roadmap-overview]]
updated: 2026-04-21
---

# System Overview

PaymentsChecker is a **personal spending tracker** delivered as a Telegram bot. Users log expenses via chat commands; the bot stores them in SQLite and pushes weekly summaries automatically.

## What It Is

- A Telegram bot users interact with directly in chat
- Each Telegram user has an isolated spending ledger
- Supports manual entry (guided or free-text) and automatic weekly reports
- Self-hosted: runs as a single Docker container on a VPS

## Components

```
┌──────────────────────────────────────────┐
│              Telegram Cloud               │
│  (message routing, bot API, long poll)    │
└─────────────────────┬────────────────────┘
                      │ HTTPS long polling
┌─────────────────────▼────────────────────┐
│           Backend (Go, Docker)            │
│                                           │
│  ┌─────────────┐    ┌──────────────────┐ │
│  │  Bot logic  │    │  Weekly scheduler│ │
│  │ (handlers,  │    │  (goroutine,     │ │
│  │  parsers,   │    │   Mon 09:00)     │ │
│  │  formatter) │    └──────────────────┘ │
│  └──────┬──────┘                         │
│         │                                │
│  ┌──────▼──────────────────────────────┐ │
│  │       SQLite (spendings.db)         │ │
│  └─────────────────────────────────────┘ │
└──────────────────────────────────────────┘

┌──────────────────────────────────────────┐
│  Website (planned)   Mobile (planned)    │
│  — web frontend —    — iOS / Android —   │
│  No code yet         No code yet         │
└──────────────────────────────────────────┘
```

## Data Flow

1. User sends a message or command to the bot in Telegram
2. Backend fetches it via long polling from the Telegram Bot API
3. Handler parses the message, performs the requested operation (store, query, delete)
4. Response is sent back via Telegram API
5. Weekly scheduler independently pushes a report every Monday at 09:00 (configurable TZ)

## Key Design Choices

- **Single binary, single SQLite file** — zero infrastructure beyond Docker
- **No HTTP API yet** — backend talks only to Telegram; website/mobile are future work
- **Per-user data isolation** — all queries filter by `user_id` from the Telegram update
- **Timezone-aware boundaries** — day/week/month/year boundaries use a configurable `TZ` env var

## Related Pages

- [[monorepo-structure]] — how the codebase is organized
- [[backend-overview]] — internals of the Go bot
- [[deployment/docker]] — how it's deployed
- [[roadmap-overview]] — planned website and mobile clients
