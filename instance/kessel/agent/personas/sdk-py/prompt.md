## Kessel Python SDK (`kessel-sdk-py`)

Client library for the Kessel Inventory and RBAC APIs.

### General

- Follow existing patterns in the codebase.
- Read the repo's `CLAUDE.md` / `AGENTS.md` and relevant `docs/*-guidelines.md` first.
- Ensure proper error handling — never silently ignore errors.
- Run tests and lint before committing. Fix any failures you introduce.
- Never skip pre-commit hooks with `--no-verify` unless explicitly requested.

### Generated vs hand-written code

**Never edit generated protobuf/gRPC files.** Regenerated from `buf.build/project-kessel/inventory-api` via `buf generate` / `make generate`.

Do not edit: `*_pb2.py`, `*_pb2_grpc.py`

**Hand-written code** — auth, gRPC credentials, `ClientBuilder`, RBAC helpers, examples, tests:
- `src/kessel/auth/`, `src/kessel/grpc/`, `src/kessel/inventory/`, `src/kessel/rbac/`

API changes belong in the upstream inventory-api proto repo — not in generated stubs.

### Architecture

- **API version**: `v1beta2` (unified inventory service). Prefer v1beta2 unless working with legacy code.
- **ClientBuilder**: Fluent builder for authenticated gRPC clients — follow existing usage.
- **Auth**: OAuth2 Client Credentials via OIDC discovery.
- **Examples**: Runnable in `examples/` — not automated tests.

### Commands

Python 3.11+ (`pyproject.toml`). Default image has Python 3.12.

- `pytest` — run tests
- `black --check .` and `flake8` — lint (CI runs both)
- `python -m build` — verify package builds
- Use `pytest-asyncio` patterns for async tests

### Dev environment

- Unit tests mock gRPC — should run without external services.
- `examples/` may need a live Kessel server — not CI tests.
