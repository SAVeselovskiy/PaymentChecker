---
title: Free-text Input
category: features
tags: [input, legacy, free-text, plain-text]
related: [[add-spending]], [[parsing]], [[handlers]]
updated: 2026-04-21
---

# Free-text Input

A legacy, quick-entry method. Users type a plain message in `Category: amount` format and the bot logs it immediately — no keyboard, no confirmation steps.

## Format

```
Food: 450
Transport: 120
Coffee: 3.50
Groceries 1200
```

Both `Category: amount` (with colon) and `Category amount` (with space) are accepted. Category names are trimmed and title-cased on save.

## Behavior

1. A message that doesn't start with `/` is caught by the `handleText` fallback handler
2. `ParseSpending` attempts to parse it as `Category: amount`
3. If parsing succeeds: spending is saved, bot confirms "Saved: {Category} — {Amount}"
4. If parsing fails: bot replies with a usage hint (does **not** return an error to the user, just a helpful message)

## Comparison with `/add`

| | `/add` | Free-text |
|---|--------|-----------|
| Steps | 2 (amount + category tap) | 1 (single message) |
| Category selection | Inline keyboard with recent + new | Typed inline |
| Confirmation | Yes | Yes (simple text) |
| Error feedback | Amount parse error shown | Parse failure shows hint |

Free-text is faster for power users; `/add` is more guided and prevents typos in category names.

## Related Pages

- [[add-spending]] — the guided alternative
- [[parsing]] — `ParseSpending` parser details
- [[handlers]] — `handleText` implementation
