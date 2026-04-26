---
title: Standalone Frontend Deployment
category: roadmap
tags: [roadmap, deployment, website, frontend, docker]
related: [[roadmap-overview]], [[adr-005-static-export-embedded-in-go]], [[adr-004-http-api-and-telegram-login]], [[docker]], [[build-deploy]]
updated: 2026-04-26
---

# Standalone Frontend Deployment

The web UI in `PaymentChecker-frontend/` is not production-ready, while the
Telegram bot in `PaymentChecker-backend/` is. To unblock backend rollout, the
two are now deployed independently.

> [!note] Not yet started
> The backend split landed on 2026-04-26. The standalone frontend image and
> deploy pipeline are still TBD.

## Current state (2026-04-26)

- The backend Docker image is built from `PaymentChecker-backend/` alone
  (build context = `.`). The bot + HTTP API run unchanged.
- `internal/webfs` still embeds a `web/out/` directory via `//go:embed`, but
  the only file inside is a placeholder `index.html` shown at `/`. This keeps
  the embed valid without dragging the Next.js build into the backend image.
- `docker-compose.yml` no longer references `PaymentChecker-frontend/` or
  `shared/`, and no longer takes the `NEXT_PUBLIC_TELEGRAM_BOT_NAME` build
  arg. `.env.example` was trimmed to match.
- `build.sh` no longer requires `NEXT_PUBLIC_TELEGRAM_BOT_NAME`.

## Next step — frontend image

Add a self-contained Dockerfile under `PaymentChecker-frontend/` that:

1. Installs pnpm + frontend workspace deps from `shared/` and the package's
   own manifest.
2. Reads the API base URL from a build arg (e.g. `NEXT_PUBLIC_API_BASE_URL`)
   and bakes it into the static export, since `NEXT_PUBLIC_*` is build-time.
3. Emits the static export and serves it via a tiny static server (nginx
   alpine or `serve`), or uploads it to a static host.

Decide hosting target before building the image:
- **Same VPS, separate container** — nginx alpine on its own port; reverse
  proxy in front of both containers. Simplest to start.
- **Vercel / Netlify / Cloudflare Pages** — drop the Dockerfile, use the
  platform's native Next.js pipeline. Fastest to ship, removes ops surface.
- **Object storage + CDN** — push `out/` to S3/R2; serve via CDN. Cheapest
  at scale, more pipeline plumbing.

## Backend changes that follow

Once the frontend has its own origin:

- **CORS.** The backend's HTTP API must allow the frontend origin and send
  cookies cross-site. Today the embedded UI is same-origin, so CORS is a
  no-op. Plan to add an allow-list driven by an env var (e.g.
  `WEB_ORIGIN`) and `Access-Control-Allow-Credentials: true`.
- **Cookie attributes.** Auth cookies will need `SameSite=None; Secure`
  when the frontend lives on a different host. Verify the JWT cookie code
  in `internal/http/auth*` before flipping origins.
- **Telegram Login Widget domain.** `/setdomain` in BotFather must point at
  whatever public origin actually serves the widget — the frontend host,
  not the backend.
- **Backend public root.** Decide whether `/` on the backend should keep
  the placeholder, redirect to the frontend host, or 404. Currently keeps
  the placeholder.

## Open questions

- Do we want a new ADR superseding [[adr-005-static-export-embedded-in-go]]
  once the frontend image lands, or fold the new deploy model into a single
  combined ADR?
- Should `internal/webfs` be removed entirely once the placeholder is no
  longer wanted, or kept as a minimal "service is up" page?
- What's the canonical public domain for the frontend? Affects CORS, cookie
  domain, and BotFather config.

## Related

- [[roadmap-overview]] — overall roadmap
- [[adr-005-static-export-embedded-in-go]] — original embedded-static model
  (now superseded for production deploys)
- [[adr-004-http-api-and-telegram-login]] — auth model, will need CORS work
- [[docker]], [[build-deploy]] — deployment docs to update once the
  frontend image exists
