# Shared Contract Changelog

## 0.1.1 — 2026-04-25

### Changed
- `POST /api/auth/telegram`: documented `auth_date` freshness window as 24 hours (was incorrectly 1 hour). Matches actual backend behaviour.
- `Set-Cookie` header response on `POST /api/auth/telegram`: added `Max-Age=604800` (7-day session lifetime) to example and description.

## 0.1.0 — 2026-04-25

Initial HTTP API surface for the web vertical slice.

### Added

- `POST /api/auth/telegram` — Telegram Login Widget exchange → JWT cookie
- `POST /api/auth/logout` — clear session (sets expired cookie)
- `GET /api/me` — current authenticated user (requires session cookie)
- `GET /api/spendings/today` — today's spending breakdown by category (requires session cookie)
- Schemas: `User`, `CategorySum`, `TodayBreakdown`, `TelegramAuthPayload`, `Error`
- Cookie-based session auth (`session` cookie, HttpOnly + Secure + SameSite=Lax)
- `shared/types/api.ts` — TypeScript types generated from the OpenAPI spec
- `shared/pnpm-workspace.yaml` — pnpm workspace root
- `shared/types/package.json` — `@paymentchecker/types` workspace package

### Migration notes

New file — no downstream migration required for existing consumers.

- **Backend:** implement four HTTP handlers per spec; mirror schemas in
  `internal/http/types.go`. HMAC verification for `POST /api/auth/telegram`
  must use `TELEGRAM_BOT_TOKEN` (or equivalent env var). Day boundaries for
  `/api/spendings/today` must use `cfg.Location` (same as the bot `/today`
  command) to ensure identical results.
- **Frontend:** install `@paymentchecker/types` from the pnpm workspace
  (`"@paymentchecker/types": "workspace:*"` in `package.json`); import
  generated types from that package. Do not hand-author API types.
- **Mobile:** no immediate action — the spec is available for future use.
