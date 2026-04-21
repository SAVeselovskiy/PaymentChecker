# PaymentChecker Wiki — LLM Schema

This is an LLM-maintained wiki for the **PaymentsChecker** monorepo: a Telegram spending-tracker bot with a Go backend, and planned website/mobile clients.

The wiki follows the pattern described at https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f.

---

## Directory Layout

```
PaymentCheckerWiki/
├── CLAUDE.md          ← this file: schema, operations, conventions
├── index.md           ← catalog of all pages (one-line summaries, by category)
├── log.md             ← append-only log of all ingest / query / lint passes
├── sources/           ← raw immutable source material (specs, docs, code excerpts)
└── pages/
    ├── architecture/  ← system design, monorepo structure, component relationships
    ├── backend/       ← Go bot internals (handlers, storage, scheduler, parsing)
    ├── features/      ← user-facing bot commands and interaction flows
    ├── data/          ← database schema, storage patterns, data model
    ├── deployment/    ← Docker, environment config, build & deploy process
    ├── decisions/     ← Architecture Decision Records (ADRs)
    └── roadmap/       ← planned work: website, mobile, API contracts
```

---

## Frontmatter Format

Every wiki page must begin with:

```yaml
---
title: Human-readable title
category: architecture | backend | features | data | deployment | decisions | roadmap
tags: [tag1, tag2]
related: [[page-a]], [[page-b]]
updated: YYYY-MM-DD
---
```

---

## Who maintains this wiki

The **shared-agent** is the designated maintainer. It is invoked automatically after significant development work or decisions to keep pages accurate and append log entries. Humans can also trigger any operation directly by asking Claude.

---

## Operations

### LOG

Use after any significant conversation, decision, or completed agent work. This is the most frequently used operation.

Append to `log.md`:
```
## LOG YYYY-MM-DD — <topic in ~5 words>
**Participants:** user, [agent names]
**Context:** one sentence — what was being worked on
**Decision:** what was decided or built
**Rationale:** why (one sentence; omit if obvious)
**Wiki:** [[pages-created-or-updated]]
```

Then create or update the relevant wiki pages to reflect the decision or change.

### INGEST

Use when adding new source material (code changes, design docs, meeting notes, etc.).

1. Place the raw source in `sources/` with a descriptive filename.
2. Read the source and identify what it touches (new feature, changed behavior, new ADR, etc.).
3. Create or update the relevant page(s) in `pages/`.
4. Update `index.md` — add any new pages, revise one-liners for updated pages.
5. Append a log entry to `log.md`:
   ```
   ## INGEST YYYY-MM-DD — <short description>
   - Source: sources/<filename>
   - Created: [[page-a]], [[page-b]]
   - Updated: [[page-c]]
   ```

### QUERY

Use to answer a question about the project using the wiki.

1. Search `index.md` for relevant categories.
2. Read the relevant pages and synthesize an answer with `[[page]]` citations.
3. If the answer would be broadly useful, save it as a new page in the most relevant category.
4. Append a log entry to `log.md`:
   ```
   ## QUERY YYYY-MM-DD — <question summary>
   - Pages consulted: [[page-a]], [[page-b]]
   - Filed result: [[page-c]] (if saved)
   ```

### LINT

Use periodically to check wiki health.

Check for:
- Pages missing from `index.md`
- Broken `[[wiki links]]`
- Contradictions between pages (flag with `> [!warning]` callout)
- Stale claims (e.g., version numbers, file paths that no longer exist)
- Orphan pages (no incoming links)
- Categories that have grown too large and should be split

Append a log entry to `log.md`:
```
## LINT YYYY-MM-DD
- Issues found: <N>
- Fixed: <list>
- Flagged for human review: <list>
```

---

## Cross-linking Conventions

- Always use Obsidian wiki links: `[[page-name]]` or `[[page-name|display text]]`
- Link on first mention of any entity that has its own page
- ADRs are referenced as `[[adr-NNN-slug]]`
- Never duplicate content — link instead

## Writing Style

- Write for a developer who is new to the project but experienced in general
- Prefer short paragraphs and bullet lists over dense prose
- Use code blocks for file paths, commands, SQL, Go snippets
- Mark uncertain/outdated content with `> [!warning] Needs verification`
- Mark planned/not-yet-built content with `> [!note] Not yet implemented`
