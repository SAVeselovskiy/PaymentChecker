# Wiki Log

Append-only chronological record of all wiki operations.

---

## LOG 2026-04-21 — shared-agent assigned wiki ownership
**Participants:** user, Claude
**Context:** Discussing how shared-agent gets triggered and what its role should be
**Decision:** shared-agent now owns both API contracts and wiki maintenance; it is activated after any significant agent work to log decisions and update pages; a compact LOG format was defined
**Rationale:** Keeps knowledge up to date automatically without relying on the user to remember to document things
**Wiki:** [[agent-system]], wiki CLAUDE.md, root CLAUDE.md updated

---

## LOG 2026-04-25 — Web vertical slice shipped
**Participants:** user, orchestrator, architector (plan only), shared-agent, backend-agent, web-agent, reviewer-agent
**Context:** Added the first cross-package feature: HTTP API + Telegram Login Widget auth + Next.js web frontend.
**Decision:** Single Go binary serves both `/api/*` and the embedded static frontend. Shared OpenAPI v0.1.1 contract; `@paymentchecker/types` via pnpm workspace.
**Rationale:** Aligns with the project's minimalist single-VPS philosophy; reuses Telegram identity for auth.
**Wiki:** [[adr-004-http-api-and-telegram-login]], [[adr-005-static-export-embedded-in-go]], [[roadmap-overview]] updated, [[index.md]] updated.
**Commits:** backend `8a5616c`, frontend `5b211ba`, shared (initial contract) `e170c45`, shared (this commit) `16eb27d`.

---

## LOG 2026-04-25 — Web slice smoke test: env propagation + SPA routing
**Participants:** user, Claude (main thread), backend-agent (via main thread), shared-agent
**Context:** Smoke-testing the deployed Docker image after the vertical-slice ship; the Telegram widget didn't render and `/login/` served the dashboard.
**Decision:** Two fixes:
- Pass `NEXT_PUBLIC_*` Next.js build vars via `docker build --build-arg` and materialise them into `.env.production` before `pnpm build` (commits 2629fc9, d3168f8).
- Trim both leading AND trailing slashes before `fs.Stat` in the SPA fallback; accept directory paths whose `<path>/index.html` exists (commit 9885277).
**Rationale:** Next.js inlines `NEXT_PUBLIC_*` at build time, not runtime — `ENV` in the Dockerfile alone is fragile; `.env.production` is Next.js's deterministic env source. Go's `fs.FS` rejects trailing slashes as invalid paths, which silently broke every directory route in a `trailingSlash: true` static export.
**Wiki:** [[adr-005-static-export-embedded-in-go]] (amended with the SPA-routing + env-baking gotchas).
**Commits:** backend 2629fc9, d3168f8, 9885277.

---

## LOG 2026-04-26 — Production DB lost during VPS migration; backups roadmapped
**Participants:** user, Claude (main thread)
**Context:** During the standalone-backend rollout (see prior log entry), the new container hit `SQLITE_CANTOPEN` and then `SQLITE_READONLY_DIRECTORY` because the docker named volume `/data` was root-owned. While diagnosing, the user pointed at `PaymentChecker-backend/spendings.db` on the host as the "old DB". We copied that file into the volume — but it turned out to be a stale test DB from an early bare-binary run. The real production DB had always lived inside the named volume (`DB_PATH=/data/spendings.db` since the initial commit), and the copy overwrote it. No backup existed.
**Decision:**
- Accept the data loss for this deploy ("sacrifice old db for this time only").
- Add database backups as a roadmap item: v1 local rotation via `sqlite3 .backup` on a cron, v2 off-box copy to S3-compatible storage. Pre-deploy snapshot inside `build.sh` to land alongside v1 so a bad migration is recoverable.
**Rationale:** Single-VPS philosophy still applies — backups should be cheap and operable without extra paid services. The incident proved that even minor migrations can wipe data when there's nothing to fall back to.
**Wiki:** [[database-backups]] (created), [[roadmap-overview]] (added Operations section + tag), [[index.md]] (registered new page).

---

## LOG 2026-04-26 — Backend deploy decoupled from frontend
**Participants:** user, Claude (main thread)
**Context:** Frontend isn't production-ready, but the Telegram bot is. VPS deploy was failing because the backend Dockerfile pulled `PaymentChecker-frontend/` and `shared/` from the monorepo root, which didn't exist on the VPS. We also hit two latent bugs in `build.sh` while debugging — silent exits caused by `set -euo pipefail` interacting with `grep` returning no match in `read_env_var`, and a broken JWT_SECRET write path when the `.env` line was missing.
**Decision:**
- Backend Docker build context narrowed to `PaymentChecker-backend/` only. Dropped the `frontend-builder` stage; the embedded `internal/webfs/web/out/` ships only a placeholder `index.html`.
- `docker-compose.yml` no longer references sibling repos and no longer takes `NEXT_PUBLIC_TELEGRAM_BOT_NAME` build arg. `.env.example` and `build.sh` trimmed accordingly.
- `build.sh`: wrapped the `read_env_var` grep in `{ ... || true; }` so a missing line is "empty value" rather than a pipefail-induced silent exit; the JWT_SECRET auto-generate branch now appends the line when absent instead of silently no-op'ing through `sed`.
- Standalone frontend deploy added to the roadmap — see [[frontend-standalone-deploy]].
**Rationale:** Unblocks shipping the bot without waiting on the web UI; clean separation aligns with future hosting flexibility (separate container, Vercel, CDN). The `build.sh` fixes were already manifesting as "no output, script exits silently" failures during the VPS bring-up.
**Wiki:** [[frontend-standalone-deploy]] (created), [[roadmap-overview]] (updated — supersedes-callout on ADR-005 deploy model), [[index.md]] (updated). [[adr-005-static-export-embedded-in-go]] left as historical until a successor ADR is written.
**Commits:** backend `dd416f4` (build.sh pipefail/JWT_SECRET fixes); follow-up commit pending for the Dockerfile/compose split.

---

## LOG 2026-04-25 — Telegram auth UX options discussed
**Participants:** user, shared-agent
**Context:** Evaluating alternatives to the Telegram Login Widget to eliminate the third-party popup UX issue.
**Decision:** Deferred roadmap entry created for Option A (bot deep-link magic-token flow) as the recommended replacement; Option B (Mini App) noted as a future complement; Option C (keep Widget) is the do-nothing default.
**Rationale:** The deep-link flow hands off to the installed Telegram app, removes the third-party iframe, and eliminates the silent failure modes documented in ADR-005.
**Wiki:** [[telegram-auth-ux]] (created), [[roadmap-overview]] (updated), [[index.md]] (updated)

---

## INGEST 2026-04-21 — Initial wiki creation

- Source: codebase exploration of `/home/sergey/Projects/PaymentsChecker`
- Created:
  - [[overview]], [[monorepo-structure]], [[agent-system]]
  - [[backend-overview]], [[handlers]], [[parsing]], [[storage]], [[scheduler]]
  - [[add-spending]], [[reports]], [[free-text-input]], [[clear]]
  - [[schema]]
  - [[docker]], [[environment]], [[build-deploy]]
  - [[adr-001-sqlite]], [[adr-002-pure-go]], [[adr-003-long-polling]]
  - [[roadmap-overview]]
- Updated: n/a (initial creation)
- Notes: Wiki bootstrapped from direct codebase exploration. Backend is production-ready; website and mobile are placeholders.
