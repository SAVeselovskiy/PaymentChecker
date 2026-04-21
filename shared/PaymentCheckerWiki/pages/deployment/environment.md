---
title: Environment Variables
category: deployment
tags: [config, environment, env-vars]
related: [[docker]], [[build-deploy]], [[schema]]
updated: 2026-04-21
---

# Environment Variables

All configuration is via environment variables, loaded at startup from `internal/config/config.go`. A `.env.example` template is provided.

## Variables

| Variable | Required | Default | Purpose |
|----------|----------|---------|---------|
| `TELEGRAM_BOT_TOKEN` | **Yes** | — | Telegram bot token from @BotFather. Without this the bot cannot start. |
| `DB_PATH` | No | `spendings.db` | Path to the SQLite database file. In Docker: set to `/data/spendings.db` via the volume mount. |
| `TZ` | No | `UTC` | IANA timezone name (e.g., `Europe/Moscow`). Controls: day/week/month/year report boundaries, and the Monday 09:00 scheduler trigger. |

## .env File

For Docker Compose, variables are loaded from a `.env` file in `backend/`:

```bash
# .env
TELEGRAM_BOT_TOKEN=123456789:ABCdef...
DB_PATH=/data/spendings.db
TZ=Europe/Moscow
```

Copy from the template:
```bash
cp backend/.env.example backend/.env
```

## Obtaining a Bot Token

1. Open Telegram, find `@BotFather`
2. Send `/newbot` and follow instructions
3. Copy the token to `TELEGRAM_BOT_TOKEN`

## Timezone Notes

- All timestamps are **stored as UTC** in SQLite regardless of `TZ`
- `TZ` only affects the **computation of time window boundaries** (what counts as "today", "this week", etc.) and the scheduled push time
- If `TZ` is invalid, the app falls back to UTC and logs a warning

## Related Pages

- [[docker]] — how `.env` is passed to the container
- [[build-deploy]] — deploy script that validates env config
- [[scheduler]] — how `TZ` affects the Monday push
- [[schema]] — `DB_PATH` targets the `spendings` table
