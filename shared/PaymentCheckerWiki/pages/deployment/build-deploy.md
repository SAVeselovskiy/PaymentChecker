---
title: Build and Deploy
category: deployment
tags: [deployment, vps, build, ci]
related: [[docker]], [[environment]]
updated: 2026-04-21
---

# Build and Deploy

The project ships via Docker. There are two deployment paths: manual Docker Compose and a one-shot VPS script.

## Local / Manual Deploy

```bash
cd backend/
cp .env.example .env    # fill in TELEGRAM_BOT_TOKEN and optionally TZ, DB_PATH
docker compose up -d --build
```

To view logs:
```bash
docker compose logs -f
```

To stop:
```bash
docker compose down
# WARNING: 'docker compose down -v' removes the data volume (all spendings deleted)
```

## VPS One-shot Deploy — `build.sh`

`backend/build.sh` is a self-contained shell script intended for a fresh VPS:

1. Installs Docker (if not present)
2. Validates the `.env` file exists and `TELEGRAM_BOT_TOKEN` is set
3. Runs `docker compose up -d --build`

Usage:
```bash
scp backend/build.sh user@vps:~/
scp backend/.env user@vps:~/   # after filling in secrets
ssh user@vps
chmod +x build.sh && ./build.sh
```

## Build Flags

The Dockerfile uses optimized build flags:
```bash
CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /usr/local/bin/paymentchecker ./cmd/bot
```

| Flag | Effect |
|------|--------|
| `CGO_ENABLED=0` | Disables CGO; statically linked binary |
| `-trimpath` | Removes local path prefixes from the binary (reproducible builds) |
| `-ldflags="-s -w"` | Strips debug info and DWARF — reduces binary size |

## Updating a Running Instance

```bash
cd backend/
git pull                        # pull latest code
docker compose up -d --build    # rebuild and restart
```

Docker Compose replaces the container while keeping the named volume intact — no data loss.

## Related Pages

- [[docker]] — Dockerfile and compose configuration
- [[environment]] — required env vars before deploying
