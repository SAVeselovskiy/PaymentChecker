---
title: Spending Reports
category: features
tags: [commands, reports, analytics, period]
related: [[handlers]], [[storage]], [[scheduler]], [[schema]]
updated: 2026-04-21
---

# Spending Reports

Five commands generate spending summaries. All group by category and show totals for a time window.

## Commands

| Command | Time window |
|---------|------------|
| `/today` | Current calendar day (midnight to now, in configured TZ) |
| `/week` | Current calendar week (Monday midnight to now) |
| `/month` | Current calendar month (1st midnight to now) |
| `/year` | Current calendar year (Jan 1 midnight to now) |
| `/stats YYYY-MM-DD YYYY-MM-DD` | Arbitrary date range (inclusive) |

## Example Output

```
📊 This week (Apr 14 – Apr 21):

Food         2 340
Transport      650
Coffee         420
──────────────────
Total        3 410
```

## How Reports Are Generated

1. Handler computes `from` and `to` as `time.Time` values in the user's configured timezone
2. Calls `store.SumByCategoryBetween(ctx, userID, from, to)`
3. Passes the `[]CategorySum` slice to `format.FormatPeriodReport`
4. Sends the rendered string back to the user

All timestamps are converted to UTC before hitting SQLite (stored as Unix seconds).

## `/stats` Format

```
/stats 2026-01-01 2026-03-31
```

Both dates are inclusive. The handler parses them with `time.Parse("2006-01-02", ...)` in the configured timezone. Invalid dates return a format error.

## Empty Reports

If no spendings exist for the period, the bot replies: "No spendings recorded for this period."

## Weekly Automatic Push

The [[scheduler]] pushes the equivalent of `/week` to all users automatically every Monday at 09:00 (configured TZ). Users with no spendings in the past 7 days receive no message.

## Related Pages

- [[storage]] — `SumByCategoryBetween` query
- [[scheduler]] — automatic Monday push
- [[handlers]] — handler implementation details
