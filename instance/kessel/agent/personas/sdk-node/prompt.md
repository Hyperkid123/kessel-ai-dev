## Kessel Node SDK (`kessel-sdk-node`)

TypeScript client library for the Kessel Inventory and RBAC APIs. Published as `@project-kessel/kessel-sdk`.

### General

- Follow existing patterns in the codebase.
- Read `AGENTS.md` / `CLAUDE.md` and relevant `docs/*-guidelines.md` first.
- `npm install` first — if it fails, stop and report on Jira.
- **npm scripts only** — never call `npx jest`, `npx eslint`, or `npx tsc` directly.
- Run tests and lint before committing.
- Never skip pre-commit hooks with `--no-verify` unless explicitly requested.

### Generated vs hand-written code

**Never edit generated protobuf/gRPC TypeScript stubs.** Regenerated from `buf.build/project-kessel/inventory-api`.

Do not edit: generated `.ts` stubs under `src/kessel/inventory/v*/` (except hand-written `index.ts`).

**Hand-written code**:
- `src/kessel/auth/`, `src/kessel/console`, `src/kessel/grpc/`, `src/kessel/inventory/index.ts`, `src/kessel/rbac/`, `src/promisify.ts`

API changes belong in the upstream inventory-api proto repo — not in generated stubs.

### Architecture

- **API version**: `v1beta2` (unified inventory service). Prefer v1beta2 unless working with legacy code.
- **ClientBuilder**: Fluent builder for authenticated gRPC clients — follow existing usage.
- **Auth**: OAuth2 Client Credentials via OIDC discovery (`oauth4webapi`).
- **Examples**: Runnable in `examples/` — require a live server, not CI tests.

### Commands

Node >= 20. Check `.nvmrc` or `engines.node` and switch with `nvm use` if needed.

- `npm test` — Jest unit tests
- `npm run lint:check` — ESLint (use `npm run lint` to auto-fix)
- `npm run type-check` — TypeScript validation
- `npm run build` — CJS + ESM + type declarations
- `npm run prettier:check` — formatting

### Dev environment

- Unit tests mock gRPC — should run without external services.
- `examples/` require a live Kessel server — not CI tests.
