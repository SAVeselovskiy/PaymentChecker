---
title: Clear Data (/clear)
category: features
tags: [commands, data-deletion, confirmation]
related: [[handlers]], [[storage]], [[schema]]
updated: 2026-04-21
---

# Clear Data — `/clear`

Permanently deletes all spending data for the user. Protected by an inline keyboard confirmation step.

## User Flow

```
User: /clear

Bot: Are you sure? This will delete ALL your spending history.
     [Yes, delete everything] [Cancel]

User: (taps "Yes, delete everything")

Bot: All your data has been deleted.
```

## Behavior

1. `/clear` triggers `handleClear` which sends a confirmation keyboard
2. "Yes, delete everything" callback triggers `handleCallback` which calls `store.DeleteAllForUser`
3. "Cancel" callback dismisses the message with "Cancelled."
4. After deletion the user can start fresh — no trace of previous data remains

## Important Notes

- **Irreversible** — there is no soft-delete or recycle bin; the row is hard-deleted from SQLite
- Scoped to the individual user's `user_id` — does not affect other users
- The confirmation keyboard times out if the user doesn't respond; a new `/clear` is needed

## Related Pages

- [[handlers]] — `handleClear` and callback handling
- [[storage]] — `DeleteAllForUser` implementation
- [[schema]] — the `spendings` table rows being deleted
