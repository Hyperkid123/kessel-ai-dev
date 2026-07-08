## Kessel Java SDK (`kessel-sdk-java`)

Client library for the Kessel Inventory and RBAC APIs.

### General

- Follow existing patterns in the codebase.
- Read the repo's `CLAUDE.md` / `AGENTS.md` and relevant `docs/*-guidelines.md` first.
- Ensure proper error handling — never silently ignore errors.
- Run tests and lint before committing. Fix any failures you introduce.
- Never skip pre-commit hooks with `--no-verify` unless explicitly requested.

### Generated vs hand-written code

**Never edit generated protobuf/gRPC files.** Regenerated from `buf.build/project-kessel/inventory-api` via `buf generate` / `make generate`.

Do not edit: files with `@Generated` / `@GrpcGenerated` annotations under `src/main/java` from buf.

**Hand-written code** — auth, gRPC credentials, `ClientBuilder`, RBAC helpers, examples, tests:
- `org.project_kessel.api.auth`, `common`, `console`, `grpc`, `inventory`, `rbac`

API changes belong in the upstream inventory-api proto repo — not in generated stubs.

### Architecture

- **API version**: `v1beta2` (unified inventory service). Prefer v1beta2 unless working with legacy code.
- **ClientBuilder**: Fluent builder for authenticated gRPC clients — follow existing usage.
- **Auth**: OAuth2 Client Credentials via OIDC discovery (Nimbus dependency).
- **Examples**: Runnable in `examples/` module — not published, separate from SDK artifact.

### Commands

Java/Maven multi-module project (`kessel-sdk` + `examples`).

- `./mvnw clean verify` — build, test, and validate (single pre-PR command)

Java is not pre-installed in this image. If local Maven is unavailable, note that CI will verify — do not block on missing JDK unless setup has been added to `setup.sh`.

### Dev environment

- Unit tests mock gRPC — should run without external services.
- `examples/` may need a live Kessel server — not CI tests.
