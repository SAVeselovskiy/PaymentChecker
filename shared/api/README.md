# PaymentChecker API Contract

This directory contains the canonical OpenAPI 3.1 specification for the PaymentChecker HTTP API.
All downstream packages (backend Go handlers, TypeScript frontend) derive their types and validation
rules from `openapi.yaml`.

## Spec file

`openapi.yaml` — OpenAPI 3.1 definition for the web/mobile HTTP API.

## Regenerating TypeScript types

Types in `shared/types/api.ts` are generated automatically from the spec using
[openapi-typescript](https://openapi-ts.dev). Run from the `shared/` directory:

```bash
pnpm --filter @paymentchecker/types gen
```

This script is defined in `shared/types/package.json` and calls:

```bash
openapi-typescript ../api/openapi.yaml -o ./api.ts
```

Commit the generated `api.ts` — reviewers and the frontend can inspect types
without running codegen locally.

## Linting the spec

```bash
npx @redocly/cli lint api/openapi.yaml
```

Run this before any PR that touches `openapi.yaml`. The CI pipeline (once
added) should enforce this automatically.

## Versioning policy

- The spec version lives in `info.version` and follows **semver**.
- Every change must have a corresponding entry in `shared/CHANGELOG.md`.
- **Never change or remove an existing field shape without a deprecation period.**
  Add the new field alongside the old one, mark the old field `deprecated: true`,
  and list all referencing files in the migration note.
- Bump patch (`0.1.x`) for additive changes (new optional field, new endpoint).
- Bump minor (`0.x.0`) for new required fields or behavior changes that require
  downstream code updates.
- Bump major (`x.0.0`) only after the deprecation period has elapsed and all
  consumers have migrated.
