---
title: Add Spending (/add)
category: features
tags: [commands, spending-entry, guided-flow, inline-keyboard]
related: [[handlers]], [[parsing]], [[free-text-input]], [[schema]]
updated: 2026-04-21
---

# Add Spending — `/add`

The primary way to log a spending entry. A two-step guided flow using an inline keyboard for category selection.

## User Flow

```
User: /add 450

Bot: Got it — 450. Which category?
     [Food] [Transport] [Coffee] [Other] [+ New category]

User: (taps "Food")

Bot: Saved: Food — 450
```

## Step-by-step

1. **User sends `/add <amount>`** (e.g., `/add 450` or `/add 45.50`)
2. `handleAdd` calls `ParseAmount` to extract the float
3. If parsing fails, bot replies with a format hint and stops
4. Amount is stored in the pending session map (`state.go`) keyed by `user_id`
5. Bot fetches the user's most recent categories from the DB and sends them as an inline keyboard
6. User taps a category button (or "New category" to type one)
7. `handleCallback` reads the pending session, creates the DB entry, clears session state
8. Bot confirms: "Saved: {Category} — {Amount}"

## Pending Session State

Sessions live in a `sync.Map` in `internal/bot/state.go`. Each entry is keyed by `user_id` (int64) and holds the pending amount. Sessions are cleared after completion or if the user starts a new `/add` before completing the previous one.

## Amount Formats Accepted

```
/add 450
/add 45.50
/add 45,50
/add 1200
```

See [[parsing]] for the full `ParseAmount` spec.

## Alternative: Free-text

For quicker entry without the keyboard flow, users can send plain text in `Category: amount` format. See [[free-text-input]].

## Related Pages

- [[handlers]] — `handleAdd` and `handleCallback` implementation
- [[parsing]] — `ParseAmount` parser
- [[free-text-input]] — alternative entry method
- [[schema]] — the `spendings` table row created
