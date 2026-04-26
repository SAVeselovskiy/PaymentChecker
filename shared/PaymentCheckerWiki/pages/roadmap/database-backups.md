---
title: Database Backups
category: roadmap
tags: [roadmap, deployment, storage, sqlite, backup, ops]
related: [[roadmap-overview]], [[storage]], [[schema]], [[docker]], [[environment]], [[build-deploy]]
updated: 2026-04-26
---

# Database Backups

Driven by an incident on 2026-04-26: while migrating the VPS deploy to the
new self-contained backend image, the production `spendings.db` inside the
docker named volume was overwritten with a stale test DB from the repo
root, and there was no backup to restore from. User data was lost.

> [!note] Not yet implemented
> No automated backup mechanism exists today. The DB only lives inside the
> docker volume `paymentchecker-backend_paymentchecker-data` on the VPS.

## Goals

- Survive the next bad migration / accidental volume mount swap / disk
  failure without losing more than ~24h of user data.
- Be cheap to operate (single-VPS philosophy, no extra paid services
  required by default).
- Backups are usable without the bot — restore should be a `cp` into the
  volume, not a custom tool.

## Constraints

- SQLite WAL is in use (assumed default for `modernc.org/sqlite`); a
  hot file copy can corrupt — backups must use `sqlite3 .backup` or
  `VACUUM INTO`, not `cp` of the live file.
- Bot runs as uid 1000 inside the container; backups run as root on the
  host or via `docker compose exec`.
- VPS storage is limited; backup retention has to be bounded.

## Proposed approach (v1 — local rotation)

1. Add a `scripts/backup.sh` to `PaymentChecker-backend/` that runs:
   ```sh
   docker compose exec -T bot sh -c \
     'sqlite3 /data/spendings.db ".backup /data/backup-$(date +%F).db"'
   docker cp paymentchecker:/data/backup-$(date +%F).db \
     /var/backups/paymentchecker/
   docker compose exec -T bot rm /data/backup-$(date +%F).db
   ```
2. Wire it to a systemd timer or cron entry on the VPS — daily at 04:00
   local. Keep last 14 days, then prune.
3. Document the restore path in [[build-deploy]]: stop the bot, copy a
   chosen `backup-YYYY-MM-DD.db` over the volume's `spendings.db`,
   start the bot.

Trade-off: backups live on the same VPS, so a host loss takes them too.

## Proposed approach (v2 — off-box copy)

After v1 is stable, push a copy off the VPS:

- **Cheapest:** `rclone` to a free-tier S3-compatible bucket (Cloudflare
  R2, Backblaze B2). Encrypt at rest with `rclone crypt`.
- **Simpler ops:** `scp`/`rsync` to a second machine you already own.
- **No-infra:** push the encrypted SQLite file as an artefact to a
  private Git repo (works for tiny DBs; not great long term).

Pick one before the user base grows past hobby scale.

## Open questions

- Is `modernc.org/sqlite` actually opening the DB in WAL mode? Check the
  storage layer; if it isn't, plain `cp` may be safe and simpler. Even
  so, prefer `.backup` for portability.
- Should backups be triggered from inside the bot process (e.g. a
  scheduled job alongside the weekly report), or stay external? Inside
  the process is more robust to compose typos but harder to run
  ad-hoc; external is simpler. Recommend external.
- Retention policy — daily for 14d is a guess; revisit after we know how
  big a typical DB grows.
- Pre-deploy hook: should `build.sh` take a backup before running
  `docker compose up -d` so a bad migration is recoverable?
  Recommendation: yes — even in v1 — guarded behind a "DB exists"
  check.

## Why this is a roadmap item, not "do it now"

The bot is currently usable and the data set is small. Locking in a
backup design before the standalone frontend deploy ([[frontend-standalone-deploy]])
ships is reasonable, but ordering is not blocking — both can be done
in parallel. Doing the v1 local rotation should take an afternoon;
v2 off-box is a follow-up.

## Related

- [[roadmap-overview]] — overall roadmap
- [[storage]] — current SQLite layer
- [[schema]] — what's actually in the DB
- [[docker]], [[build-deploy]], [[environment]] — deploy docs that need
  an "Operations / Backups" section once v1 lands
