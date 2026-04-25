---
title: Telegram Auth UX — Replace Login Widget
category: roadmap
tags: [roadmap, auth, telegram, ux, deferred]
related: [[adr-004-http-api-and-telegram-login]], [[roadmap-overview]]
updated: 2026-04-25
---

# Telegram Auth UX — Replace Login Widget

> [!note] Not yet started — deferred
> No timeline. See the Implementation Outline section for the steps when this is picked up.

## Context

The website currently authenticates users via the official Telegram Login Widget — an iframe served by `oauth.telegram.org` and wired to `POST /api/auth/telegram` (see [[adr-004-http-api-and-telegram-login]]). The Widget is functional but has UX friction:

- On devices without an active `oauth.telegram.org` session, the Widget opens a third-party popup window for phone-number entry.
- The popup is visually inconsistent with the rest of the app (Telegram-branded UI, separate window context).
- Silent failure modes exist: a blank Widget appears on domain mismatch or when the build-time `NEXT_PUBLIC_TELEGRAM_BOT_USERNAME` env var is missing — documented in [[adr-005-static-export-embedded-in-go]] under Implementation Gotchas.

The goal is a flow that hands off directly to the installed Telegram app rather than routing through a browser popup.

---

## Options

### Option A — Bot deep-link "magic token" flow (recommended)

**Flow:**

1. User clicks "Login with Telegram" on the website.
2. Website calls `POST /api/auth/start`. The backend generates a short-lived random token (32 bytes hex, 5-minute TTL), stores `(token, user_id=NULL, expires_at)` in a new `pending_logins` table, and returns `{ token, deep_link }`.
3. Website opens `https://t.me/<bot_username>?start=<token>`. On mobile this deep-links to the Telegram app via the `tg://` URI scheme; on desktop it opens Telegram Desktop or Telegram Web as a fallback.
4. User taps Start in Telegram. The bot receives `/start <token>` with the user's Telegram `user_id`. The bot handler updates the row: `pending_logins.user_id = update.message.from.id`.
5. Website polls `GET /api/auth/check?token=<token>` (or listens via SSE). When the backend sees `user_id IS NOT NULL`, it issues the same JWT cookie as today and returns the `User` object.
6. Website redirects to `/`.

**Pros:**

- Matches the user's stated goal: opens the Telegram app, not a browser popup.
- Single entry point for any browser; native app handoff when the Telegram app is installed.
- No third-party iframe; no phone-number entry in the browser.

**Cons:**

- Introduces new backend state: the `pending_logins` table and a background sweeper (or lazy-delete on read).
- Requires a new bot command handler for `/start <token>`.
- Requires polling or SSE on the frontend for the 5-minute window.

**Estimated scope:** ~30 lines of bot-side handler, ~50 lines of HTTP handler, one new table, two new OpenAPI contract entries.

---

### Option B — Telegram Mini App (Web App)

The website runs inside Telegram via BotFather `/newapp` or `/setmenubutton`. `Telegram.WebApp.initData` provides a signed payload containing the user identity — implicit auth with zero clicks.

**Pros:**

- Best in-Telegram UX; zero auth friction for users who open the URL from within Telegram.

**Cons:**

- Only works when the user opens the URL from inside Telegram; does not cover the standalone-browser use case.
- Layers Telegram-specific APIs (`window.Telegram.WebApp`) into the shared codebase.

**Verdict:** best treated as a complement to Option A for an in-Telegram experience, not a replacement for the general browser flow. The two can coexist — the backend can accept either auth proof.

---

### Option C — Stay on the Widget

The current Widget already attempts a `tg://` deep link on mobile when Telegram is installed. The phone-number popup only appears for users who have no active `oauth.telegram.org` session and no installed app. Doing nothing is acceptable if mobile UX is good enough and desktop friction is tolerable.

**Pros:**

- No engineering work.

**Cons:**

- Desktop UX still routes through a third-party popup for unauthenticated users.
- Silent failure modes documented in ADR-005 continue to affect newcomers and local dev setups.

**Verdict:** the do-nothing default. Acceptable until the friction is measured to be a meaningful drop-off.

---

## Recommendation

Adopt **Option A** as the primary replacement for the Login Widget. Layer **Option B** on top later for an in-Telegram experience — the two flows coexist cleanly because the backend JWT-issuance step is the same regardless of which auth proof arrives. **Option C** is the current state and the implicit fallback if this work is not prioritized.

---

## Implementation Outline

High-level steps only — detailed design is deferred.

1. **Contract:** add `POST /api/auth/start` and `GET /api/auth/check` to `shared/api/openapi.yaml`. Bump the spec to v0.2.0 (additive — the existing `POST /api/auth/telegram` Widget endpoint stays for backward compatibility). Schemas:
   - `AuthStartResponse { token: string, deep_link: string, expires_at: string }`
   - `AuthCheckResponse { status: 'pending' | 'authorized' | 'expired', user?: User }`

2. **Storage:** add a `pending_logins` table:
   ```sql
   CREATE TABLE pending_logins (
     token      TEXT    PRIMARY KEY,
     user_id    INTEGER,
     expires_at INTEGER NOT NULL
   );
   ```

3. **Backend:**
   - HTTP handler for `POST /api/auth/start` — generate token, insert row, return `deep_link`.
   - HTTP handler for `GET /api/auth/check` — look up token, issue JWT cookie and return `User` when `user_id IS NOT NULL`, otherwise return `pending` or `expired`.
   - New bot command handler for `/start <token>` — parse token from payload, upsert `pending_logins.user_id`.
   - Background sweeper (or lazy-delete on read) to remove expired rows.

4. **Frontend:**
   - Replace `TelegramLoginButton` (or add an alternative path) — call `POST /api/auth/start`, render a "Continue in Telegram" button that opens the `deep_link`, and show a fallback QR code for desktop users.
   - Poll `GET /api/auth/check` every 1–2 seconds with exponential backoff.
   - Show clear progress states: "Waiting for confirmation in Telegram…", "Confirmed, signing you in…", "Link expired — try again."

5. **Wiki:** write ADR-006 documenting the migration; add a "Superseded by ADR-006 for primary flow" note to [[adr-004-http-api-and-telegram-login]].

---

## Open Questions

- Polling interval and total timeout — suggest 1.5 s interval, 5 min total matching token TTL.
- Whether to keep `POST /api/auth/telegram` (Widget endpoint) as a fallback for users who cannot or will not use the deep-link, or deprecate and eventually remove it.
- QR code rendering: server-side generated image vs. client-side library (e.g. `qrcode.react`).
- SSE or WebSocket as an alternative to polling — worth the added complexity for a 5-minute auth window?
- Mobile app (`PaymentChecker-mobile/`) auth model — does the deep-link flow apply as-is, or does the native app need a separate pattern (e.g. custom URL scheme handoff)?
