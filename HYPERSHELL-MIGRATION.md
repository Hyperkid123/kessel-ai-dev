# HyperShell dispatcher migration draft: Kessel and Gateway

**Status: preparation only.** This branch prepares dispatcher profiles for the two existing Kessel deployments. It does not enroll this repository in the dispatcher, publish or promote an image, change app-interface/GitOps, deploy anything, or authorize a cutover. No live resources were changed.

This work owns only `project-kessel/kessel-ai-dev`. Rehor core is unchanged.

## Scope and invariants

- Migrate both existing instances independently: `instance/kessel/agent/` (`devbot-kessel`, label `hcc-ai-kessel`) and `instance/gateway/agent/` (`devbot-gateway`, label `hcc-gateway-ai`). They share this repository and the shared proxy/memory-server, but keep separate instance IDs, Jira labels, boards, config paths, repo allowlists, MCP configuration, env presets, personas, and schedules.
- Preserve the deployed identities. The source of the live values is the app-interface SaaS file `data/services/insights/kessel/kessel-ai-dev.yml` plus the legacy templates' effective defaults. The values copied into `env` are non-secret identity/routing values only.
- Leave the legacy deploy templates untouched: `deploy/template.yaml`, `deploy/gateway-template.yaml`, their KEDA `ScaledObject`s, NetworkPolicies, PostgreSQL sidecar, and image parameters. This draft does not remove or rewrite the legacy path.
- Do not run dispatcher and legacy polling against the same instance at the same time.
- No secret values are committed or logged. Slack webhooks, Jira credentials, Git credentials, SSO credentials, and provider tokens remain deployment-owned.

## Deployed identity parity

| Setting | `instance/kessel` | `instance/gateway` |
| --- | --- | --- |
| Legacy deployment | `devbot-kessel` (`deploy/template.yaml`) | `devbot-gateway` (`deploy/gateway-template.yaml`) |
| `BOT_INSTANCE_ID` | `Oisín na Suthain-Shíor-Oibre` | `Matúš Čák Trenčiansky, Pán Siedmich Brán` |
| `BOT_LABEL` | `hcc-ai-kessel` | `hcc-gateway-ai` |
| Jira board | `2379` (`BOT_BOARD_ID`) | `10850` (`BOT_BOARD_ID`) |
| Backlog intake | `BOT_INCLUDE_BACKLOG=true` | `BOT_INCLUDE_BACKLOG=true` |
| Config path | `instance/kessel` | `instance/gateway` |
| Workflow / source | `jira-sprint` / Jira | `jira-sprint` / Jira |
| Env presets | `node`, `go`, `slack`, `container-scan` | `slack` |
| Legacy schedule (UTC) | daily `03:30`–`00:00` | Mon–Fri `07:00`–`00:00` |
| Repo allowlist | `project-repos.json` (12 GitHub repos) | `project-repos.json` (`app-interface` on GitLab) |
| MCP | `mcp-atlassian` via deployment `JIRA_MCP_URL` | `mcp-atlassian` via deployment `JIRA_MCP_URL` |

The existing `project-repos.json`, `mcp.json`, personas, and `CLAUDE.md` content are preserved unchanged. The legacy Slack webhooks and other secrets are not copied into the repository.

## OpenCode routing decision

Both profiles select:

```yaml
runtime: opencode
provider: vertex
model: claude-opus-4-6
```

Evidence for this route:

- Both legacy templates keep the existing Vertex proxy path (`ANTHROPIC_VERTEX_BASE_URL=http://devbot-proxy:8443`, `EXECUTOR_ADDR=devbot-proxy:9090`) and the app-interface deployment sets `VERTEX_ALLOWED_MODELS=claude-sonnet-4-6,claude-opus-4-6,claude-haiku-4-5`.
- The pinned dev-bot `config.json` uses `claude.model: claude-opus-4-6`, so this draft pins the same model explicitly instead of relying on a workflow tier or a provider default.
- Upstream Rehor documents OpenCode `vertex` as Claude through the existing auth proxy and lists `claude-sonnet-4-6`, `claude-opus-4-6`, and `claude-haiku-4-5@20251001` as packaged Vertex models (`docs/models.md`).
- The Console Next draft uses `rehor-openai` because its deployment supplies the OpenAI model gateway. That route is not configured for these Kessel deployments, so copying it would change routing and likely fail the gateway allowlist.

`BOT_EXECUTION_ENGINE=coordinator` is set in the profile environment because dispatcher sandbox runs reject the default Python engine. The remaining `env` entries are non-secret `BOT_*` values accepted by the dispatcher profile allowlist.

## Preflights

The upstream `jira-sprint` workflow default set is `gh-pr-status` and `gl-mr-status`. Both profiles declare those plus the required-input `jira-sprint` preflight:

```yaml
preflights:
  - gh-pr-status
  - gl-mr-status
  - id: jira-sprint
    with:
      label: hcc-ai-kessel   # hcc-gateway-ai for the gateway profile
      include_backlog: true
```

All three ids are registered TypeScript preflights in the pinned Rehor revision and in current upstream `master`. No core preflight is missing for this workflow.

**Parity limitation (tracked, not invented):** the deployed Python runner receives `BOT_JIRA_EMAIL`, and the Python sprint/backlog scan uses it for the bot-assigned candidate pass. The current dispatcher does not wire that deployment identity to the optional `email` input of `jira-sprint`, and secrets must not be copied into `instance.yaml`. Until that identity path is implemented and validated (Rehor #678 / REHOR-278 work is not merged upstream), the dispatcher profile scans unassigned candidates only and can miss bot-assigned sprint/backlog tickets. This is a pre-cutover blocker that needs an owner in Rehor core, not a value change in this repository.

## OpenCode guidance and native agents

- `instance/*/agent/AGENTS.md` are added as verbatim copies of the existing instance `CLAUDE.md`. OpenCode loads profile `AGENTS.md`; it does not apply Claude instruction assembly. The original `CLAUDE.md` files remain untouched for Claude rollback.
- Personas remain file-based: the `jira-sprint` workflow selects `personas/<name>/prompt.md` using the repo key and reads it in the session. Kessel has no native subagent delegation requirement, so this draft adds no `.opencode/agents` roles. Adding subagents would invent a delegation model and potentially change tool reach; this draft deliberately does not broaden access.
- Native workflow skills/commands (`wrap-up`, `triage`, `new-work`, `claim-ticket`, `slack-notify`, `gh-release-upload`, and shared push/PR skills) must be available from the promoted runner image and validated under OpenCode before cutover.

## Schedules

The legacy `ScaledObject` windows are mirrored literally, because the dispatcher implements KEDA cron-window semantics (`docs/scheduling.md`; `packages/agent-dispatcher/src/schedule.ts`):

| Profile | Timezone | Start | End | Legacy source |
| --- | --- | --- | --- | --- |
| Kessel | `UTC` | `30 3 * * *` | `0 0 * * *` | `deploy/template.yaml` KEDA (`30 3` → `0 0`, daily) |
| Gateway | `UTC` | `0 7 * * 1-5` | `0 0 * * 1-5` | `deploy/gateway-template.yaml` KEDA defaults (`07:00`–`00:00`, Mon–Fri) |

The dispatcher gates new sessions only; in-flight sessions drain after a window closes. Because the gateway end expression excludes Saturday and Sunday, the window opened on Friday remains active until Monday `00:00` UTC under KEDA semantics. The dispatcher evaluates the same literal expressions and reproduced that behavior in validation; the expressions were intentionally not "corrected", because changing them would diverge from the deployed scaler. No KEDA `ScaledObject` or warm-pool configuration is added for dispatcher-managed agents.

## dev-bot pin

`dev-bot` moves from `150a350` (upstream `main` at branch creation) to `a113ed3e55322c77b24fe4408c7a8fb499407074`.

- `a113ed3` contains top-level profile `env` support (#668), dispatcher profile discovery with required `image`, registered TypeScript preflights, and KEDA-style schedule evaluation (#674). It is the pin used by the Console Next and HCC migration drafts.
- Current upstream Rehor `master` is `b5d2e8d`; its only commits after `a113ed3` are dependency/CI updates, with no dispatcher or preflight behavior change.
- Moving the gitlink alone does not change the running image. The Kessel runner image must be rebuilt and published with this revision and the OpenCode V2 closure before dispatcher use.

## Validation performed

Actual dispatcher validation was run against this checkout with the upstream Rehor dispatcher built from `master` (`b5d2e8d`) in a separate validation worktree:

```text
node /tmp/rehor-192-validate.mjs /Users/martin/testing/kessel-ai-dev-hypershell-dispatch
discovered 2 profile(s)
...
validation: OK
```

The harness called the real `discoverAgentProfiles`, `validateConfig` against `createPopulatedRegistry`, and `isScheduleActive` implementations. Results:

- exactly two profiles were discovered: `gateway` and `kessel`, with the intended identities, labels, config paths, image, and schedules;
- image, `env`, `preflights`, and `schedule` fields parsed; profile discovery rejected neither;
- all declared preflights resolved with parameters: `gh-pr-status`, `gl-mr-status`, and `jira-sprint` (`label`, `include_backlog`, default `interrupted_retry_minutes`);
- Kessel schedule checks at `03:29`, `03:30`, `12:00`, `23:59`, and `00:00` UTC matched expectations;
- Gateway schedule checks at Monday `06:59`/`07:00`/`12:00`, Friday `23:59`, Saturday `00:00`, and Monday `00:00`/`07:00` UTC matched the KEDA literal-window behavior;
- repository checks: PyYAML parsed both profile files and `git diff --check` was clean.

Live Jira, GitHub, GitLab, memory-server, proxy, Gateway, OpenShell sandbox, image, and full cycle behavior are **not** validated by this preparation.

## Gates before cutover

1. **Dispatcher and image compatibility:** deploy a dispatcher image containing `a113ed3` (or current `master`) and build/publish a Kessel runner image from the same revision with the OpenCode V2 runtime closure. Confirm the configured `latest` tag resolves to that compatible image, then pin the exact digest/tag for the rollout. Do not infer compatibility from a floating tag.
2. **Bot-assigned candidate identity:** implement and validate deployment identity wiring for the `jira-sprint` bot-assigned pass, or obtain an explicit owner decision that unassigned-only intake is acceptable. The current draft cannot close this parity gap from the instance repository.
3. **Services gateway and credentials:** configure the dispatcher's authenticated services gateway for Jira, GitHub, and GitLab, and complete the applicable security review (including the REHOR-254 authenticated-gateway risk decision) before enabling live preflights. Verify the gateway routes, memory identity, Slack delivery, and Vertex allowlist with the promoted image.
4. **OpenCode assets:** verify native workflow skills/commands and MCP exposure in the sandbox. Confirm personas still load by file read and no L0/LSP assumption remains.
5. **Schedule and isolation:** verify both profiles report `off-schedule` outside their windows, dispatch inside them, and never share instance IDs, labels, boards, sessions, or checkouts. Ensure the legacy deployment is scaled to zero/stopped for an instance before the dispatcher handles it.
6. **Cycle checks:** run a no-work cycle for each profile (no sandbox starts) and one representative work cycle per instance in an approved test/canary context, including Jira lifecycle, repo cloning, PR/MR creation, feedback handling, and cleanup.
7. **Human review:** review the profile diff, identities, schedules, routing, image reference, and rollback readiness. GitOps/app-interface enrollment remains a separate change.

Do not enroll this repository in the dispatcher until the gates pass and an owner approves the cutover.

## Rollback

1. Remove this configuration repository from the dispatcher's configured repository list so neither profile is discovered. Verify discovery is gone.
2. Restore the previous dispatcher image/template revision if a rollout changed them.
3. If the legacy deployment was stopped during cutover, restore its prior app-interface/GitOps configuration and replicas, including the KEDA windows and image digest.
4. Restore the previous `dev-bot` gitlink and rebuild/redeploy the prior known-good Kessel runner image if the new image was promoted.
5. Confirm only one execution path is active for each instance before declaring rollback complete.

The legacy deploy templates are untouched, so no template rollback is required by this draft.

## Draft limitations

- No dispatcher enrollment, image publication/promotion, app-interface/GitOps change, deployment, Jira write, or live cycle.
- `jira-sprint` bot-assigned candidate parity is blocked by the missing deployment-identity-to-preflight wiring described above.
- Native OpenCode workflow skills/commands and sandbox MCP behavior are not proven here.
- The `latest` image tag is a placeholder gate, not a verified compatible artifact.
- Console Next PR #68 and HCC PR #121 are open drafts and are references for format/scope only; this draft does not depend on their merge.
