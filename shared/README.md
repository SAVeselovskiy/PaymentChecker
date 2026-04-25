# shared/

API contracts, schemas, and types shared across backend, website, and mobile.

## Contents

| Path | Purpose |
|------|---------|
| `api/` | OpenAPI specs or Protobuf definitions |
| `types/` | Generated TypeScript types (`@paymentchecker/types`) |
| `constants/` | Cross-package enums and constants |
| `CHANGELOG.md` | Contract change history |

## Workspace Layout

`shared/` doubles as the **pnpm workspace root**. This means frontend tooling
can reference types from this directory as a proper workspace package rather
than via a file path or published registry.

- **`shared/types/`** is the `@paymentchecker/types` workspace package.
  `PaymentChecker-frontend/` declares `"@paymentchecker/types": "workspace:*"`
  in its `package.json` dependencies and imports generated types from there.
- **`shared/pnpm-workspace.yaml`** lists workspace members:
  - `types` — the types package in `shared/types/`
  - `../PaymentChecker-frontend` — the web frontend (relative path)

### Regenerating TypeScript types

Run from `shared/`:

```bash
pnpm --filter @paymentchecker/types gen
```

This calls `openapi-typescript ../api/openapi.yaml -o ./api.ts` and overwrites
`shared/types/api.ts`. Commit the result so reviewers and CI can inspect types
without running codegen.

## Rules

- All changes go through **shared-agent**.
- Breaking changes require a deprecation period — see agent instructions for the protocol.
- Downstream packages (backend, website, mobile) must be updated before the old contract is removed.
