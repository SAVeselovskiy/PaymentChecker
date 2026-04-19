# shared/

API contracts, schemas, and types shared across backend, website, and mobile.

## Contents

| Path | Purpose |
|------|---------|
| `api/` | OpenAPI specs or Protobuf definitions |
| `types/` | Language-agnostic type definitions (JSON Schema) or shared TS types |
| `constants/` | Cross-package enums and constants |
| `CHANGELOG.md` | Contract change history |

## Rules

- All changes go through **shared-agent**.
- Breaking changes require a deprecation period — see agent instructions for the protocol.
- Downstream packages (backend, website, mobile) must be updated before the old contract is removed.
