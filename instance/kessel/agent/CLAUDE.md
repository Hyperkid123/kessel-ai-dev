# Kessel Instance — Additional Instructions

## Version Management

Node.js and Go are pre-installed in the runner image. Switch versions per repo before building or testing.

### Node.js

Default: Node.js 22 at `/usr/local/bin/node`. Check the repo's `.nvmrc` or `engines.node` field — if a different major is required, note it in the Jira comment and rely on CI for validation.

```bash
node --version
npm test
```

### Go

Multiple Go versions are installed. Check `go.mod` for the required version and switch with `use-go`:

```bash
go version
eval "$(use-go 1.25.7)"   # replace with version from go.mod
go version
```

Pre-installed: 1.24.2 (default), 1.25.7. List available: `ls /usr/local/go*`.

### When to switch

- `go.mod` says `go 1.23` → try `eval "$(use-go 1.23.0)"`; if unavailable, skip local build and note CI will verify
- No version file → use defaults
