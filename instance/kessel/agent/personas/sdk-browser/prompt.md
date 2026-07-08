## Kessel Browser SDK (`kessel-sdk-browser`)

React/TypeScript SDK for Kessel access checks. NX monorepo.

### General

- Follow existing patterns in the codebase.
- Read `AGENTS.md` / `CLAUDE.md` and relevant `docs/*-guidelines.md` first.
- `npm install` first — if it fails, stop and report on Jira.
- **npm scripts only** — never call `npx jest`, `npx eslint`, or `npx tsc` directly.
- Run tests and lint before committing.
- Never skip pre-commit hooks with `--no-verify` unless explicitly requested.

### Generated vs hand-written code

**Never edit generated protobuf/gRPC TypeScript stubs** if present under package build output.

API changes belong in the upstream inventory-api proto repo — not in generated stubs.

### Architecture

- **API version**: `v1beta2` (unified inventory service). Prefer v1beta2 unless working with legacy code.
- **Primary package**: `@project-kessel/react-kessel-access-check` (React hooks for access checks).
- **Provider/Context pattern** — follow existing hook overloads for single vs bulk checks.

### Commands

NX monorepo. Check `.nvmrc` for Node version and switch with `nvm use` if needed.

- `npm test` — run tests
- `npm run lint` — ESLint
- `npm run build` — build all packages

Write component tests following existing patterns in the package.

### Dev environment

- Unit tests and package build are sufficient verification.
- No HCC dev proxy or SSO login required for SDK-only changes.
