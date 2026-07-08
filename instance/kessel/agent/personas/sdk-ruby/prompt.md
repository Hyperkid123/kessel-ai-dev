## Kessel Ruby SDK (`kessel-sdk-ruby`)

Client library for the Kessel Inventory and RBAC APIs.

### General

- Follow existing patterns in the codebase.
- Read the repo's `CLAUDE.md` / `AGENTS.md` and relevant `docs/*-guidelines.md` first.
- Ensure proper error handling — never silently ignore errors.
- Run tests and lint before committing. Fix any failures you introduce.
- Never skip pre-commit hooks with `--no-verify` unless explicitly requested.

### Generated vs hand-written code

**Never edit generated protobuf/gRPC files.** Regenerated from `buf.build/project-kessel/inventory-api` via `buf generate` / `make generate`.

Do not edit: `*_pb.rb`, `*_services_pb.rb`

**Hand-written code** — auth, gRPC credentials, `ClientBuilder`, RBAC helpers, examples, tests.

API changes belong in the upstream inventory-api proto repo — not in generated stubs.

### Architecture

- **API version**: `v1beta2` (unified inventory service). Prefer v1beta2 unless working with legacy code.
- **ClientBuilder**: Fluent builder for authenticated gRPC clients — follow existing usage.
- **Auth**: OAuth2 Client Credentials via OIDC discovery.
- **Examples**: Runnable in `examples/` — not automated tests.

### Commands

Ruby/Bundler is not pre-installed in this image. If local tooling is unavailable, note that CI will verify.

- `bundle install` — install dependencies
- `bundle exec rspec` — run tests (`COVERAGE=1` for coverage)
- `bundle exec rubocop` — lint
- `bundle exec bundler-audit` — security audit
- Maintain RBS type signatures in `sig/` when changing public APIs

### Dev environment

- Unit tests mock gRPC — should run without external services.
- `examples/` may need a live Kessel server — not CI tests.
