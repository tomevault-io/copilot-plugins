## adv26

> This repository is a documentation-only runbook for an authorized, disposable AI red-team lab. It contains no application implementation, model weights, cloud state, or secrets. Codex, Claude, and other coding agents must follow this file when asked to build, operate, recover, or destroy the lab.

# AGENTS.md — end-to-end deployment contract

This repository is a documentation-only runbook for an authorized, disposable AI red-team lab. It contains no application implementation, model weights, cloud state, or secrets. Codex, Claude, and other coding agents must follow this file when asked to build, operate, recover, or destroy the lab.

## Read first

Read these documents completely before changing cloud state:

1. `README.md`
2. `docs/architecture.md`
3. `docs/security.md`
4. `docs/gpu-rental.md`
5. `docs/model-choice.md`
6. `docs/capability-evaluation.md`
7. `docs/azure-goad.md`
8. `docs/runpod-vllm.md`
9. `docs/tailscale.md`
10. `docs/agent-stack.md`
11. `docs/parallel-agents.md`
12. `docs/tracing-replay.md`
13. `docs/validation.md`
14. `docs/operations.md`
15. `docs/lessons-learned.md`

Use current official provider/project documentation to verify commands, versions, SKU availability, pricing, and API behavior. The dates and prices in this repository are historical observations.

## Mission

When the user asks for an end-to-end build, produce a working system with:

- private GOAD in an explicitly selected Azure subscription/resource group;
- a B300-class RunPod or equivalent GPU serving the selected model with authenticated vLLM;
- an Azure Tailscale subnet router and a RunPod userspace Tailscale client;
- a constrained web agent, model gateway, egress proxy, headless jumpbox runner, and authenticated web UI;
- when enabled, a bounded Pi subagent coordinator that uses the same model gateway and preserves per-run isolation;
- live/archived JSONL trace capture and replay;
- completed positive and negative validation;
- a handoff that includes cost, URLs, secret locations (not secret values), backup state, and teardown instructions.

The deployment application is not present here. Either integrate an implementation the user supplies or create it in a separate private implementation repository/workspace following `docs/agent-stack.md` and `docs/tracing-replay.md`. Do not claim the documentation repository contains runnable app source.

## Non-negotiable safety rules

1. Work only in cloud accounts and targets explicitly authorized by the user.
2. GOAD Windows VMs must not receive public IPs.
3. Restrict jumpbox public SSH to the user's current `/32`; never use `0.0.0.0/0`.
4. The agent's only target scope is the configured jumpbox and GOAD subnet.
5. Never scan or interact with public internet targets, cloud metadata, unrelated private ranges, or the user's production systems.
6. Never commit or print API tokens, passwords, private keys, Tailscale state/auth keys, Terraform state, recovered credentials, tickets, hashes, model weights, JIT binaries, or traces containing secrets.
7. Use separate admin and runner SSH keys. The runner must be unprivileged and have no sudo.
8. Do not give the public web agent a shell unless an enforceable network/process sandbox exists and the user explicitly approves the expanded design.
9. Do not expose internal ports 1055, 1056, 13128, 18080, Docker, or an admin shell.
10. Never delete a resource based only on a remembered name. Inventory the active account, subscription, resource group, pod, and volume first.
11. Obtain explicit user approval immediately before creating billable resources and before permanent teardown unless the user's current request already clearly authorizes that exact action.
12. Preserve unrelated resources and pre-existing local changes.
13. Parallel workers must have explicit, non-overlapping assignments. Use one writer per working tree and never let concurrency expand network scope or secret access.

## Required user inputs

Discover safe values where possible. Ask only for choices that materially change the deployment:

- Azure subscription and acceptable region/budget;
- intended GOAD CIDR (default reference `10.42.42.0/24`) and confirmation it does not overlap existing routes;
- operator public SSH CIDR;
- RunPod account/budget and whether direct port 8888 must be public;
- preferred GPU provider if not RunPod;
- Tailscale tailnet authorization method and who can approve routes;
- single-user/team count and whether isolation must be per-user;
- location of an existing agent/web implementation, or authorization to create one privately;
- data-retention requirements for traces and reports;
- whether cloud resources should be stopped or destroyed after validation.

Never request a secret in chat when an interactive login, local secret file, environment variable, keychain, or provider CLI can supply it.

## Execution phases and gates

Maintain an explicit plan. Do not skip a gate.

### Phase 0 — local and account preflight

- Check Git status and preserve user changes.
- Verify `az account show`, RunPod identity/balance/spend limit, Tailscale access, and GitHub/source access.
- Inventory existing Azure groups/resources and RunPod pods/volumes.
- Check Azure quota and live B300 availability.
- Present estimated hourly cost, storage durability, region, and exact resources to be created.

**Gate:** user authorizes the billable plan.

### Phase 1 — Azure GOAD

- Clone/pin upstream GOAD.
- Select subscription and non-overlapping subnet.
- Set budget/alerts and jumpbox SSH `/32`.
- Adjust VM SKUs to quota without exposing Windows VMs.
- Provision and verify all six VMs/network resources.
- Record the generated resource group and encrypted state/key locations outside Git.
- Create the no-sudo `evaluator` account with a dedicated public key.

**Gate:** all expected hosts exist; jumpbox admin and evaluator paths work; no Windows public IP exists.

### Phase 2 — Azure Tailscale router

- Install Tailscale on the jumpbox.
- Enable forwarding.
- Advertise only the GOAD `/24`.
- Have the user/admin approve the route and apply least-privilege grants.

**Gate:** router is online and route is approved.

### Phase 3 — GPU and vLLM

- Create the final pod configuration once, with required disk and ports.
- Verify B300/CUDA/Torch compatibility.
- Install vLLM in a dedicated environment and required CUDA dev tools.
- Generate a new bearer token in a mode-600 file.
- Download the exact model revision directly from the publisher.
- Launch with the documented single-GPU flags.
- Observe compilation/warmup; do not kill active `ptxas`/autotune as a “hang.”
- Export a compatible JIT-cache archive after success.

**Gate:** authenticated model list and deterministic chat completion succeed; unauthenticated public request returns 401 if exposed.

### Phase 4 — RunPod Tailscale

- Start userspace `tailscaled` with loopback SOCKS5 and HTTP proxy listeners.
- Authorize the node and accept routes.
- Verify the router and an approved HTTP target through port 1056.
- Verify runner SSH through port 1055 as `evaluator`.

**Gate:** approved target succeeds; adjacent unapproved subnet is not routed through the constrained proxy.

### Phase 5 — agent implementation/deployment

- Use or build a private implementation matching `docs/agent-stack.md`.
- Separate UIDs/containers, secrets, read-only code, and writable workspaces.
- Implement the key-hiding model gateway and allowlisting egress proxy.
- Give the web agent no shell and only workspace file tools plus `practice_http`.
- Implement the fixed-destination jumpbox tool and fresh per-run directories.
- Add authenticated web UI, operator message queue, and one-active-run control.
- If the user enables parallel execution, install and pin the audited `pi-subagents` package in the coordinator image, start with four workers, and load-test before raising concurrency.
- Keep the public web agent extension-free. Run the coordinator only behind the authenticated headless path, and route every child through the same model and jumpbox policy boundaries.
- Use read-only/scouting workers concurrently. Give write work separate Git worktrees or serialize it through one worker.

**Gate:** service health, secret access denial, file-tool proof, and runner identity checks pass. If parallel mode is enabled, prove bounded fan-out, trace attribution, cancellation, and absence of conflicting writes.

### Phase 6 — tracing and replay

- Implement the JSONL schema, bounded writer queue, run index, live cursor API, and replay viewer.
- Store traces outside the agent workspace.
- Authenticate trace APIs and render trace content as text.
- Apply retention and redaction policy.

**Gate:** live and completed traces parse, replay, and report correct done/cursor headers.

### Phase 7 — end-to-end validation

Run every check in `docs/validation.md`, including negative tests. At minimum prove:

- public auth works;
- direct model token is required;
- web agent cannot read secrets;
- model gateway limits are active;
- approved GOAD HTTP succeeds;
- out-of-scope subnet returns local 403;
- runner executes as evaluator in a unique directory;
- operator message is validated and delivered;
- full trace is archived and replayable;
- the abbreviated capability regression battery in `docs/capability-evaluation.md` passes, and any prompt-injection failure is reflected in the granted tool authority.

**Gate:** no unresolved high-risk failure. Do not hand the UI to users otherwise.

### Phase 8 — handoff

Report:

- URLs and usernames, but never secret values unless the user explicitly asks for a specific generated secret;
- pod ID/SKU/current hourly rate and Azure resource group;
- exact secret file locations and rotation commands;
- current process and route health;
- backup/persistence status;
- single-tenant/multi-tenant limitation;
- stop and teardown commands;
- completed validation evidence and known caveats.

## Progress and evidence

- Give the user concise updates at least once per minute during long provisioning or compilation.
- Distinguish observation from inference.
- Save command outputs needed for recovery in a private `.local/` directory, redacted and gitignored.
- Record software versions, image identifiers/digests, model revision, GPU SKU, subnet, and cloud resource IDs privately.
- Prefer provider CLI/API state over stale notes.
- If a required authorization link is waiting (for example Tailscale), continue independent work and state the exact remaining action.

## Troubleshooting policy

- Read `docs/lessons-learned.md` before replacing a working component.
- Inspect process CPU/GPU activity before declaring vLLM stuck.
- Treat cache incompatibility warnings as valid; rebuild rather than forcing binaries.
- Re-fetch RunPod SSH metadata after restart.
- Use managed SSH if direct TCP SSH is unstable; preserve a separate file-transfer path.
- Do not modify pod ports/storage/template without a verified off-pod backup.
- Do not weaken the allowlist, cloud firewall, authentication, or Linux isolation to make a test pass.

## Teardown workflow

When the user asks to tear down:

1. Inventory the exact RunPod pods/volumes and Azure subscription/groups/resources.
2. Confirm which traces/reports should be retained and export only approved, redacted material.
3. Delete the exact RunPod pod and lab-owned volumes.
4. Delete the exact Azure GOAD resource group; delete automatically created groups only if the user asked to empty the subscription and inventory proves they are unrelated to anything else.
5. Poll Azure until the group no longer exists.
6. Verify `runpodctl pod list`, network-volume inventory, `az group list`, `az resource list`, and `az vm list` show the expected zero/remaining state.
7. Remove stale Tailscale nodes/routes/auth keys.
8. Tell the user what was removed, what local files remain, and that billing reports can lag.

Destruction is complete only after the provider inventories verify it.

---
> Source: [antojoseph/adv26](https://github.com/antojoseph/adv26) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
