## Tooling Guidelines — Kessel Kafka Connect

You are working on `kessel-kafka-connect` — a container and deployment repo for Kafka Connect connectors used by Kessel.

### General

- Follow existing patterns in the codebase.
- Read `README.md`, `Makefile`, and files under `deploy/` and `scripts/` before making changes.
- Match existing conventions in Dockerfiles, deploy templates, and shell scripts.

### Repo layout

- `Dockerfile` / `Dockerfile.fedramp` — container images
- `Makefile` — image build/push targets only (`docker-build-push`, `build-push-minimal`)
- `deploy/` — OpenShift/Kubernetes deployment templates
- `scripts/` — helper scripts for build and deploy
- `.tekton/` — CI pipeline definitions

### Pre-PR validation — MANDATORY

This repo has no `make test`, `make lint`, or `make validate` targets. CI runs `jira-check` on PRs and Konflux/Tekton builds the container image. Validate locally before opening a PR:

```bash
cd ./repos/kessel-kafka-connect

# Review changed files for correctness — primary validation in this pod
# - Dockerfile / Dockerfile.fedramp syntax and base image references
# - deploy/ template parameters and image tags
# - scripts/ and build_*.sh logic

# Optional: shellcheck on edited scripts (if available)
shellcheck scripts/*.sh build_*.sh 2>/dev/null || true
```

Do not attempt container builds locally — there is no Docker daemon in this pod. Image build validation happens in CI/Konflux after the PR is opened.

### PR requirements

- PR title must include a Jira key (e.g. `RHCLOUD-12345`) — enforced by `jira-check.yml` CI.
- Follow existing commit and PR conventions from the repo README.

### What NOT to Do

- Do not run `make docker-build-push`, `make build-push-minimal`, `docker build`, or `podman build` — no container runtime is available in this pod.
- Do not modify `.github/workflows/` — PAT lacks workflow scope.
- Do not run Makefile targets that invoke Docker or podman.
- If the same error persists after 2 fix attempts, stop and ask for human help in Jira.
