---
title: Docker Setup
category: deployment
tags: [docker, docker-compose, deployment, containers]
related: [[environment]], [[build-deploy]], [[adr-002-pure-go]]
updated: 2026-04-21
---

# Docker Setup

The backend ships as a single Docker container defined in `backend/Dockerfile` and orchestrated with `backend/docker-compose.yml`.

## Dockerfile (Multi-stage)

**Stage 1 — Build:**
```dockerfile
FROM golang:1.25-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /usr/local/bin/paymentchecker ./cmd/bot
```

**Stage 2 — Runtime:**
```dockerfile
FROM alpine:3.20
COPY --from=builder /usr/local/bin/paymentchecker /usr/local/bin/paymentchecker
ENTRYPOINT ["/usr/local/bin/paymentchecker"]
```

The `CGO_ENABLED=0` flag and the pure-Go SQLite driver mean the binary is fully statically linked and runs in a minimal Alpine image with no libc dependencies. See [[adr-002-pure-go]].

## docker-compose.yml

```yaml
services:
  bot:
    build: .
    restart: unless-stopped
    env_file: .env
    volumes:
      - paymentchecker-data:/data

volumes:
  paymentchecker-data:
```

- **Restart policy:** `unless-stopped` — auto-restarts on crash or server reboot
- **Volume:** `paymentchecker-data` is a named Docker volume mounted at `/data`; the SQLite database lives at `/data/spendings.db`
- **Env:** loaded from `.env` file (see [[environment]])

## Running Locally

```bash
cd backend/
TELEGRAM_BOT_TOKEN=xxx /usr/local/go/bin/go run ./cmd/bot
```

## Running in Docker

```bash
cd backend/
cp .env.example .env   # fill in TELEGRAM_BOT_TOKEN
docker compose up -d --build
```

## Data Persistence

The SQLite database is stored in the named volume `paymentchecker-data`. It survives container restarts, image rebuilds, and `docker compose down` — but **not** `docker compose down -v` (which removes volumes).

## Related Pages

- [[environment]] — all env vars the container uses
- [[build-deploy]] — VPS deploy via `build.sh`
- [[adr-002-pure-go]] — why `CGO_ENABLED=0` works here
