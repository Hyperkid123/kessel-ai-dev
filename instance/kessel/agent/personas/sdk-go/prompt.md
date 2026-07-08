## Kessel Go SDK (`kessel-sdk-go`)

Client library for the Kessel Inventory and RBAC APIs.

### General

- Follow existing patterns in the codebase.
- Read the repo's `CLAUDE.md` / `AGENTS.md` and relevant `docs/*-guidelines.md` first.
- Ensure proper error handling — never silently ignore errors.
- Use the LSP tool to check for type errors and trace code paths.
- Run tests and lint before committing. Fix any failures you introduce.
- Never skip pre-commit hooks with `--no-verify` unless explicitly requested.

### Generated vs hand-written code

**Never edit generated protobuf/gRPC files.** Regenerated from `buf.build/project-kessel/inventory-api` via `buf generate` / `make generate`.

Do not edit: `*.pb.go`, `*_grpc.pb.go`

**Hand-written code** — auth, gRPC credentials, `ClientBuilder`, RBAC helpers, examples, tests:
- `kessel/auth/`, `kessel/grpc/`, `kessel/inventory/internal/builder/`, `kessel/rbac/`

API changes belong in the upstream inventory-api proto repo — not in generated stubs.

### Architecture

- **API version**: `v1beta2` (unified inventory service). Prefer v1beta2 unless working with legacy code.
- **ClientBuilder**: Fluent builder for authenticated gRPC clients — follow existing usage.
- **Auth**: OAuth2 Client Credentials via OIDC discovery.
- **Examples**: Runnable in `examples/` — not automated tests.

### Commands

Check `go.mod` for Go version; switch with `eval "$(use-go <version>)"` if needed.

- `make test` — `go test -v ./kessel/...`
- `make lint` — golangci-lint (via Docker/Podman)
- `make fmt` / `make mod-tidy` — formatting and module hygiene
- `make build` — compile example binaries

CI requires both lint and test workflows to pass. Table-driven tests. Explicit error handling — no silent `_` on errors.

### Dev environment

- Unit tests mock gRPC — should run without external services.
- `examples/` may need a live Kessel server — not CI tests.
