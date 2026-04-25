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
