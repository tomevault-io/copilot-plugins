## planning

> If you're an AI session (Claude Code, another agent, future-me) picking up this project, read this first. It's the fast path to understanding where we are without reading the entire conversation history that produced the current state.

# CLAUDE.md — repo-root handoff for Claude sessions

If you're an AI session (Claude Code, another agent, future-me) picking up this project, read this first. It's the fast path to understanding where we are without reading the entire conversation history that produced the current state.

## What this project is (one paragraph)

**Farmer** is a .NET 9 control plane that orchestrates Claude CLI workers on Hyper-V Ubuntu VMs, with a Microsoft Agent Framework (MAF) retrospective agent on the host that reviews every run. NATS JetStream + ObjectStore for coordination, Jaeger for traces, HTTP `/trigger` for ingress, OTel-instrumented throughout. Target audience: Azure/.NET developers learning agent orchestration. The competitive differentiator is using MAF "as much as possible" while keeping workers autonomous on VMs. See [README.md](./README.md) for the full pitch and architecture diagram.

## Current phase state (as of 2026-04-15 session)

Everything below is merged to `main`.

- **Phase 5** shipped: externalized runtime, OTel, real SSH end-to-end verified.
- **Phase 6** shipped: real `worker.sh` + Claude CLI on VM, `RetrospectiveStage` + MAF OpenAI `gpt-4o-mini`.
- **NATS cutover (PR #5)**: file-based `InboxWatcher` retired. Every stage transition publishes a `RunEvent` to the `FARMER_RUNS` JetStream stream; run artifacts upload to the `farmer-runs-out` ObjectStore bucket. See [ADR-010](./docs/adr/adr-010-nats-messaging-cutover.md).
- **Phase 7 retry driver (PR #8)**: opt-in retry via `RetryPolicy` on the `/trigger` body. Driver loops up to `max_attempts` on configured verdicts; each retry gets a synthetic `0-feedback.md` prompt with the prior attempt's `ReviewVerdict.Findings` + `Suggestions`. Chain linked via `parent_run_id`. See [ADR-011](./docs/adr/adr-011-retry-driver.md).
- **VM release fix (PR #10)**: `RunWorkflow.ExecuteAsync` now releases the reserved VM in a `finally` block. Before this, `IVmManager.ReleaseAsync` was never called by anything; in-process retry chains failed at attempt 2's ReserveVm. See [docs/session-retro-2026-04-15.md](./docs/session-retro-2026-04-15.md).
- **IWorkflowRunner seam + real-Retry demo (PR #12)**: `RetryDriver` now depends on an `IWorkflowRunner` interface so its loop is testable (2 integration tests with a `FakeWorkflowRunner`). Also restored cost-report persistence that PR #8 accidentally dropped. New `WORKER_MODE=fake-bad` produces adversarial canned output on the first attempt and clean output on the retry -- the loop fires on real `Retry`/`Reject` verdicts instead of a contrived `retry_on_verdicts: ["Accept"]`. See [docs/retry-demo-2026-04-16.md](./docs/retry-demo-2026-04-16.md).
- **Tests**: 133 green (128 unit + 5 integration with NatsServerFixture).

## The plan file for the active session

Plans live at `C:\Users\Derek\.claude\plans\robust-forging-iverson.md` (outside the repo -- session artifact, not tracked in git). The commit log is authoritative when the plan file disagrees.

## Runtime directory (NOT in git)

Runtime state lives at `C:\work\iso\planning-runtime\`, deliberately outside the repo. Path is configurable via `Farmer:Paths` in `appsettings.Development.json`. Structure:

```
C:\work\iso\planning-runtime\
├── data\sample-plans\     ← worker inputs (copied from repo data/sample-plans/)
├── runs\{run_id}\         ← immutable after completion, one dir per run
├── nats\                  ← JetStream store_dir (FARMER_RUNS stream + farmer-runs-out bucket)
├── outbox\
├── qa\
```

## Registered ports

| Service | Port | Status |
|---|---|---|
| `farmer/api` | 5100 | dotnet, HTTP (`scripts/dev-run.ps1`) |
| `nats-server` | 4222 | NATS core (`infra/start-nats.ps1`) |
| `nats-monitoring` | 8222 | NATS HTTP monitoring |
| `jaeger/otlp-grpc` | 4317 | Jaeger OTLP ingest (`infra/start-jaeger.ps1`) |
| `jaeger/ui` | 16686 | Jaeger UI (http://localhost:16686) |

## Common gotchas

- **SSH key path.** `FarmerSettings.SshKeyPath` defaults to `id_ed25519`. The legacy `id_rsa` is encrypted with a passphrase on this machine, and `Renci.SshNet.PrivateKeyFile` can't talk to `ssh-agent`. Don't "fix" this by changing to `id_rsa` — it will fail at `Deliver` stage with `SshPassPhraseNullOrEmptyException`.
- **SSH uses absolute paths for SCP destinations.** `Renci.SshNet`'s `ScpClient` does NOT expand `~`. The VM config uses `/home/claude/projects` as `RemoteProjectPath`, not `~/projects`. See [commit `a3c3c5b`](../git) on the verification branch for history.
- **Mapped drive is read-only from Windows.** Writes to VM always go through SSH/SCP. Reads come through the mapped drive (`O:\projects\` for `claudefarm2`). There's a ~500ms SSHFS cache lag that `MappedDriveReader` handles with a retry loop.
- **`CollectStage` rejects empty `files_changed`**. Phase 5 fake workers used sentinel strings (`FAKE_WORKER_NO_REAL_CHANGES`) to satisfy this. Phase 6 workers will populate `Manifest.Outputs[]` AND leave `files_changed` non-empty for back-compat. See [ADR-003](./docs/adr/adr-003-anti-drift-contract.md) and `CollectStage.cs`.
- **Three-file anti-drift invariant.** `events.jsonl`, `state.json`, and `result.json` must agree on the final phase and stages_completed list. Two regression tests (`BugRegression_*`) pin this. See [ADR-003](./docs/adr/adr-003-anti-drift-contract.md) for the full history of why.
- **Middleware ordering matters.** `WorkflowPipelineFactory` builds the middleware list in this exact order: `Telemetry → Logging → Eventing → [fresh CostTracking] → Heartbeat`. Don't reorder. `CostTrackingMiddleware` is `new`'d inline per-run (see [ADR-004](./docs/adr/adr-004-workflow-pipeline-factory.md)) and is NOT in DI.
- **MAF + OpenAI, not MAF + Anthropic.** The host retrospective agent uses `Microsoft.Agents.AI.OpenAI 1.1.0` (stable), not the preview Anthropic package. The VM worker still uses Claude CLI. This is a deliberate hybrid — see [ADR-006](./docs/adr/adr-006-openai-over-anthropic-maf.md) and [ADR-009](./docs/adr/adr-009-hybrid-maf-host-cli-vm.md).

## Do-not-touch list

- **`.claude/worktrees/charming-moore/`** — another agent's idle worktree. User explicitly said "leave it alone, I'll look at it later". Don't nuke it, don't commit into it, don't switch into it.
- **`origin/claude/phase5-otel-api`** — a parallel Phase 5 implementation from a different agent. We cherry-picked `WorkflowPipelineFactory` from it (see [ADR-004](./docs/adr/adr-004-workflow-pipeline-factory.md)). Don't merge the rest of that branch without a plan — it's a different architecture than ours.
- **Any pushed feature branches** — do not rebase, do not force-push. If you need to change history on a pushed branch, stop and ask the user.
- **`main` branch's README and license** — if you update the README on main, do it via a proper commit flow, not a direct push.

## Build + test + run

```powershell
cd C:\work\iso\planning

# Build + test
dotnet build src\Farmer.sln                # expect clean, 0 warnings
dotnet test src\Farmer.sln                 # expect 133 green (128 unit + 5 integration)

# Start infra (idempotent; no-ops if already listening)
.\infra\start-nats.ps1                     # nats-server on :4222/:8222
.\infra\start-jaeger.ps1                   # jaeger on :4317/:16686 (downloads on first run)

# Optional: verify worker.sh parity between repo and vm-golden
.\infra\check-worker-parity.ps1            # OK / DRIFT / FAIL; -Deploy auto-fixes drift

# Start Farmer.Host (uses appsettings.Development.json: local paths + NATS + Jaeger)
# dev-run.ps1 runs the parity check pre-flight; use -SkipWorkerCheck to bypass.
.\scripts\dev-run.ps1                      # binds http://localhost:5100

# In a second window -- trigger a run + see where to look:
.\infra\Farmer.SmokeTrace.ps1              # fake-mode 7/7 green, prints Jaeger URL + NATS stats
.\infra\Farmer.SmokeTrace.ps1 -WorkerMode real   # real Claude CLI (~5-20 min)

# Or manual curl:
curl.exe -X POST -H "Content-Type: application/json" `
  --data-binary "@scripts\demo\sample-request.json" `
  http://localhost:5100/trigger

# With retry:
curl.exe -X POST -H "Content-Type: application/json" -d '{
  "work_request_name": "react-grid-component",
  "worker_mode": "fake",
  "retry_policy": { "enabled": true, "max_attempts": 2, "retry_on_verdicts": ["Retry"] }
}' http://localhost:5100/trigger
```

## Key read order if you're picking this up cold

1. [README.md](./README.md) -- overall architecture and status
2. This file -- session-level gotchas
3. [docs/adr/README.md](./docs/adr/README.md) -- load-bearing design decisions (ADR-010 NATS cutover, ADR-011 retry driver are the newest)
4. [docs/phase5-pattern.md](./docs/phase5-pattern.md) -- Phase 5 invariants (filesystem as truth, anti-drift)
5. `git log --oneline -20` on `main` -- source of truth for what's committed
6. The plan file at `C:\Users\Derek\.claude\plans\robust-forging-iverson.md` -- current working plan if it exists and isn't stale

## User collaboration notes

- The user wants hard, specific critique with receipts. Lead with issues, not wins. Roasts must include fixes. (See `C:\Users\derek\.claude\projects\D--work-planning\memory\feedback_critique_style.md` if available.)
- The user prefers atomic commits that each build + test green in isolation. Use `git stash --keep-index --include-untracked` to validate each commit's staged state before committing.
- The user pushed back on tool-allowlist conservatism for the VM worker — full dangerous mode is the design, not a compromise. See [ADR-008](./docs/adr/adr-008-workers-full-dangerous-mode.md).
- The user's target audience is Azure/.NET devs learning MAF. Code comments should lean toward tutorial-style when the code is demonstrating a new MAF pattern.
- "Data is the product." Failures are captured as data, not treated as terminal errors. See [ADR-007](./docs/adr/adr-007-qa-as-postmortem.md).

---
> Source: [djcdevelopment/planning](https://github.com/djcdevelopment/planning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
