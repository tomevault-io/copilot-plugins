## lumyn

> Status: Normative v3.1 planning contract; product runtime not implemented

# AGENTS.md — Lumyn Repository Contract

Version: 3.1
Status: Normative v3.1 planning contract; product runtime not implemented
Scope: This repository only.

## 1. Scope And Intent

- Build Lumyn as the provider-to-consumer application layer for consequential
  API and SDK changes: provider-originated intent to consumer-controlled,
  tested draft PR and consented rollout evidence.
- Treat `docs/product/prd.md` as the product source of truth.
- Treat `docs/product/plan.md` as the human-readable active plan.
- Treat the v3.1 operating documents and the compiled control set under
  `.factory/artifacts/prd-to-plan/lumyn-migration-mvp/` as one planning
  authority.
- Do not describe bounded-agent migration execution, patch delivery, branch
  delivery, PR-bundle delivery, or draft-PR delivery as implemented.
- Regenerate the complete active v3.1 control set whenever its PRD, plan,
  acceptance, task, validation, or closure semantics change. Do not hand-edit
  one compiled artifact to bypass another.
- Keep factoryd dispatch paused. The external Factory
  `profiles/lumyn.yaml` profile and the factoryd bundle/runtime have not been
  requalified against this v3.1 generation.
- Treat the M1 public/synthetic corpus and developer-harness implementation
  packet as immutable historical task evidence. M1's benchmark and lifecycle
  scope is closed with recorded process debt, but none of its 15 linked
  product acceptance items is terminal. This rebaseline authorizes no new
  Lumyn product runtime implementation or live product action.
- Keep `.factory/artifacts/prd-to-plan/lumyn-mvp/` and its task, pilot, and
  lifecycle artifacts immutable as historical evidence.
- Keep Factory run evidence under `.factory/artifacts/`, scratch under
  `.factory/tmp/`, and daemon state under `.factoryd/`.
- Keep consumer-private runtime artifacts in an explicitly configured,
  non-committable root outside the consumer checkout and every public source
  repository.
- Keep independent review, holdout, trace-grade, attestation, shipping, and PR
  lifecycle evidence lifecycle-owned. `task-executor` may not write
  `.factory/artifacts/lifecycle-evidence/` or
  `.factory/artifacts/pr-lifecycle/`.

## 2. North Star

Every product change should improve one or more of:

- provider-paid completion of a consequential API or SDK sunset campaign;
- provider-originated, reusable, confirmed change intent;
- revocable consumer installation and event-specific authorization;
- consumer-controlled repository access and execution;
- bounded deterministic or agent-assisted patch candidates;
- deterministic, baseline-aware repository and workflow verification;
- tested draft-PR delivery with legible patch, local-branch, and PR-bundle
  fallback;
- event-bound, consumer-consented provider rollout status;
- exact Agent Runner and model egress, credential, network, disclosure, cost,
  and provenance controls;
- proof-honest residual-risk reporting;
- consumer review and human merge authority;
- fail-closed handling of unsupported or ambiguous integrations.

## 3. Product Authorities: Two Principals, Two Authorities

Keep the two principals separate:

- The API Provider owns API/SDK intent, the sunset objective, compatibility
  window, supported semantics, and campaign sponsorship.
- The API Consumer Organization owns repository access, commands, model egress,
  credentials, execution, disclosure, branch policy, review, and merge.

Provider payment or campaign sponsorship never grants consumer repository
authority. Consumer participation never lets Lumyn invent or rewrite provider
intent.

Use explicit terms:

- `api_provider` or `change_authority` for the API seller;
- `agent_runner_vendor` for the company supplying the selected coding-agent
  harness;
- `model_provider` for the endpoint used by bounded-agent execution;
- `api_consumer_organization` for the repository-owning organization;
- `consumer_maintainer` for the human with approval and merge authority;
- `lumyn_operator` for the service operator coordinating the campaign.

Do not use bare `provider` where the meaning could be ambiguous.

## 4. Non-Negotiable Product Constraints

- Analyze only explicitly authorized repositories.
- Never claim coverage of all downstream integrations.
- A Provider Change Contract is authoritative when accountably confirmed. Its
  provider event and contract remain non-executable data, and v3.1 does not
  require an elaborate PKI, universal event network, or receipt protocol.
- Provider material is data, never executable authority. Do not execute
  provider-supplied scripts or let repository/provider content widen tools,
  permissions, network access, or writable paths.
- Record stable change identity, audience, source, target, semantic intent,
  unresolved questions, provenance, confirmation, and
  supersession/withdrawal state used by every migration plan.
- The first provider channel is a signed versioned manifest at a pinned
  provider-controlled HTTPS URL. It embeds the Provider Change Contract or
  pins its exact provider-controlled URL. Verify origin, enrolled key,
  sequence, freshness, retrieved-byte contract digest, audience, and lifecycle
  state; attended import is recovery and cannot authorize
  installed-preauthorization writes.
- Derive every update-run authorization from a revocable Consumer Installation
  binding provider/channel, repository/package root, selectors, actions, exact
  Agent Runner adapter and version policy, execution-funding mode,
  credential/usage-billing ownership, model-data, GitHub, retention, expiry,
  and disclosure. Provider input may narrow but never widen that authority.
- Treat task- and campaign-level product-authority arrays as capability
  universes, never runtime grants. Before every product action, select a named
  route and freeze its exact required plus conditionally selected capability
  union. A composed campaign must reuse the validated action routes and must
  not grant their aggregate union to every installation.
- Impact analysis is read-only.
- Planning is read-only and must precede every write. A Consumer Maintainer
  either approves the exact event plan or has explicitly selected bounded
  `installed_preauthorization`; any out-of-policy plan pauses before mutation.
- Treat installation action modes as ceilings. Store no GitHub token in an
  installation; issue a short-lived token through the approved local or CI
  credential broker only for an in-policy delivery step.
- Run patching only in an isolated consumer-local workspace within approved
  paths and file, line, and diff budgets.
- Prefer a deterministic transform when the approved intent maps exactly to a
  supported recipe. Use a bounded agent only for approved plan items that need
  repository-specific reasoning.
- `agent_execution_policy` defaults to `disabled`; notify-only, scan-only, and
  deterministic-only work requires no Agent Runner credential. An
  `agent_assisted` item pauses until the Consumer Maintainer explicitly
  configures and authorizes a qualifying route.
- A bounded agent must have the consumer-selected exact Agent Runner adapter
  and version, executable source/digest, conformance digest, authorized auth
  mode and entitlement class, model and endpoint, Agent Runner and model
  credential environments, request disclosure, network allowlists, tools,
  prompt/tool versions, writable paths, turn, token, time, cost, file, and diff
  budgets. Launch support is Codex and Claude Code after each adapter passes
  the common conformance suite and an approved live canary. Cursor remains
  deferred until it passes the same gate.
- Start a clean, ephemeral Agent Runner session for every attempt from a
  neutral home/config root and an explicitly resolved executable. Never resume
  a personal or unrelated agent conversation or allow repository-local PATH
  shadowing. Static native user/project rules are ignored unless the consumer
  explicitly selects them; when selected, their digest is recorded and their
  content remains untrusted data that cannot widen Lumyn authority. Executable
  plugins, MCP servers, and hooks are prohibited for the MVP.
- Run the Agent Runner with explicit mounts, no host home or OS credential
  store, no ambient service sockets or unrelated inherited descriptors,
  inherited child-process restrictions, host-enforced egress, a pinned
  backend/version/configuration and qualification digest plus host platform,
  hard CPU/memory/PID/process-tree/disk/open-file quotas, and cleanup evidence.
  An unenforceable boundary blocks launch.
- Every agent action selects one exact authorization topology: local runtime,
  runner-mediated, direct-model, or hybrid. Remote topologies cannot launch
  without their required runner/model network, credential, and disclosure
  minimums; registry access remains independently selected.
- Never silently switch Agent Runner adapter, version, Model Provider, model,
  endpoint, credential owner, or usage-billing owner. An unavailable or
  unqualified selected route blocks agent execution.
- Model output is an untrusted patch candidate. It cannot approve its own plan,
  widen its scope, run undeclared tools, access ambient credentials, or grade
  its own result.
- Do not claim byte-identical patch determinism for agent-assisted output.
  Record model, endpoint, version, parameters, prompt/tool digests, attempt
  identity, token/cost use, and resulting patch digest.
- Verification is deterministic with respect to its pinned repository head,
  commands, fixtures, toolchain, environment, and evidence policy. Generation
  provenance and verification strength remain separate.
- Missing business values, ambiguous provider intent, prompt injection,
  unsupported code, or exhausted budgets fail closed as `needs_input`,
  `unsupported`, `uncertain`, or `blocked`.
- Repository commands run without network by default and through a supported
  fail-closed isolation backend.
- Dependency lifecycle scripts require separate consumer approval.
- Production credentials and production mutations are prohibited in the MVP.
- Redact secrets before persistence, model egress, or sharing.
- Raw consumer code, diffs, logs, traces, prompts, responses, agent sessions,
  and credentials are never visible to the API Provider. Only enumerated,
  consented campaign status or aggregate fields may cross that boundary.
- External model disclosure is separate from provider disclosure. It must name
  exact request classes, Agent Runner Vendor and downstream Model Provider
  processing, logging/training/retention terms, and deletion posture.
- For configured execution, the default `consumer_managed` mode uses a
  qualifying consumer-owned agent account, enterprise subscription, API
  credential, or local runtime; the consumer owns third-party usage billing
  and Lumyn receives no reusable credential. The route must expose the actual
  model identity and permit non-interactive automation. An optional
  `provider_sponsored_lumyn_managed` mode may use Lumyn-paid, task-scoped
  credentials inside the same consumer-authorized local or CI boundary. The
  broker binds issuer, installation/event/plan/attempt and runner/model
  audience and maximum one-hour TTL. One-time redemption creates one
  attempt-scoped session; multiple calls are allowed only within hard
  token/cost quotas, with no refresh, post-attempt replay, or cross-attempt
  reuse. Revocation and reconciliation through a vendor-native bounded
  credential or approved budget-enforcing proxy are mandatory. The API
  Provider pays the campaign but never receives code, context, agent-session
  access, or credentials in either mode.
- Preserve these delivery states separately: patch artifact, optional local
  branch, PR bundle, remote branch, draft PR, review, and merge.
- Local patch and PR-bundle fallback require no GitHub credential. Remote
  branch and draft-PR delivery require separate short-lived authorization;
  manual-only delivery cannot close the product proof.
- Never write to the default branch or auto-merge.
- Use only the canonical successful-verification labels `static_verified`,
  `repo_verified`, `workflow_contract_replay_passed`,
  `workflow_verified_replay`, `workflow_verified_mock`, and
  `workflow_verified_sandbox`.
- A `workflow_verified_*` label requires an approved entrypoint executed from
  the exact patched repository head plus observed interaction and outcome
  evidence in that environment. Independent contract replay cannot exceed
  `repo_verified`.
- Unimplemented commands must return a typed nonzero result.
- The v2 `provider-signed acknowledgement` and receipt-backed billing protocol
  are deferred compatibility concepts, not active v3 prerequisites.

## 5. Initial MVP Boundary

- Provider-paid API or SDK update channels launched through services-assisted
  sunset campaigns.
- Consumer-local or consumer-controlled CI execution.
- Consumer-selected Codex or Claude Code Agent Runner after common conformance;
  Cursor is not launch scope.
- GitHub-hosted TypeScript/Node repositories.
- One explicitly selected package root and one official npm SDK per run.
- Direct imports and statically resolvable wrappers within the approved scope.
- Deterministic transformations where exact mappings are available.
- Bounded-agent patch generation for approved repository-specific changes.
- Deterministic repository and workflow verification for every candidate patch.
- Patch artifact, local branch, and PR bundle as fallback handoff.
- At least one same-run first-campaign proof from authenticated provider event
  and installed preauthorization through an organically agent-assisted item on
  the consumer-selected qualified runner, independent exact-head verification,
  a tested Lumyn-opened draft PR under separate short-lived grants, and a
  consented provider-received status projection; separate runs do not qualify.
- Human review and merge.
- Authentication, production-data migrations, cross-language campaigns,
  generated-client regeneration, default-branch writes, and automatic merge
  remain out of scope unless a later approved contract says otherwise.

## 6. Required Boundaries

- `docs/product/`: product requirements and human plan.
- `docs/dev/`: repo-local engineering and validation guidance.
- `docs/architecture/`: architecture, trust boundaries, ADRs, and findings.
- `.factory/artifacts/prd-to-plan/lumyn-migration-mvp/`: active compiled v3
  planning, task, validation, acceptance, and closure control set.
- `.factory/artifacts/prd-to-plan/lumyn-mvp/`: immutable historical plan.
- `.factory/artifacts/task-runs/`: task-owned validation and work proof.
- `.factory/artifacts/lifecycle-evidence/`: independent lifecycle evidence.
- `.factory/artifacts/pr-lifecycle/`: PR lifecycle evidence.
- `schemas/`: versioned executable artifact contracts.
- `cmd/lumyn/`: CLI entrypoint and process result.
- `internal/source/`: source parsing only.
- Future `internal/change/`, `internal/installation/`,
  `internal/authorization/`, `internal/impact/`, `internal/status/`,
  `internal/migrationplan/`, `internal/agent/`, `internal/workspace/`,
  `internal/patch/`, `internal/verify/`, and `internal/github/`: distinct
  product boundaries.
- Consumer-private instances of plans, prompts, responses, patches, evidence,
  and PR bundles remain outside the checkout.

## 7. Trust And Capability Gates

Public-fixture planning and deterministic validation default to:

- no ambient secrets;
- no live network;
- no customer repositories;
- no model endpoint;
- no provider sandbox;
- no GitHub writes.

Live product work uses private, schema-backed, task-scoped grants:

- `customer_repo_read`: repository, readable paths, exclusions, expiry,
  retention, deletion, and evidence owner;
- `customer_repo_write`: approved plan digest, writable paths, isolated
  workspace, file/line/diff budgets, expiry, and rollback;
- `command_execution`: exact commands, working directory, mounts, tool roots,
  timeout/output budgets, environment classes, network posture, lifecycle
  scripts, socket/descriptor policy, and host-isolation backend;
- `model_request_disclosure`: exact source/context classes permitted to leave
  the consumer plane, prohibited classes, redaction, logging, training,
  retention, and deletion terms;
- `agent_runner_network`: exact Agent Runner Vendor control-plane endpoint and
  operation allowlist, separate from any downstream model endpoint;
- `agent_runner_credential`: consumer-owned or task-scoped brokered runner
  credential environment, auth mode, entitlement class, scopes, isolated
  injection, expiry, revocation, prohibited persistence, and evidence;
- `model_network`: exact model-provider endpoint and operation allowlist,
  request/response, token, time, retry, and cost budgets;
- `model_credential`: credential environment, scopes, isolated injection,
  expiry, revocation, and evidence;
- `package_registry_read`: exact registry or immutable snapshot, package,
  integrity, toolchain, lifecycle-script, expiry, and read-only budget;
- `sandbox_request_disclosure`, `sandbox_network`, and `sandbox_credential`:
  independent non-production workflow-verification grants;
- `github_branch_write`: repository, non-default branch namespace, base commit,
  token expiry, and rollback;
- `github_pr_write`: repository, authorized remote branch, base branch,
  draft-only posture, token expiry, idempotency key, and approved plan/evidence
  refs;
- `provider_reporting`: exact event-bound fields the consumer permits Lumyn to
  share with the API Provider; campaign proof requires a consented status
  projection, never raw evidence;
- `artifact_retention` and `artifact_deletion`: exact artifact classes, storage
  boundary, TTL, expiry/revocation triggers, receipt owner, retry, and orphan
  route.

`customer_repo_read`, `customer_repo_write`, `command_execution`,
`model_request_disclosure`, `agent_runner_network`, `agent_runner_credential`,
`model_network`, `model_credential`, `github_branch_write`, and
`github_pr_write` are independent. A plan approval is not a write grant. An
Agent Runner account is not necessarily a direct Model Provider account. A
local branch is not a remote branch. A PR bundle is not a GitHub write. A
remote-branch grant is not a PR grant.

Wildcard or ambient grants are prohibited. Model, registry, sandbox, and GitHub
network allowlists use exact endpoints. Factory's closed worker capabilities
remain `approval`, `credentials`, and `network`; they govern the implementation
worker and never substitute for Lumyn product authority.

## 8. Evidence And Proof Rules

- Keep intent, impact, generation mode, patch, verification, delivery,
  permission, cost, and residual-risk axes separate.
- Bind evidence to provider-confirmed intent, repository base/head, plan digest,
  generation mode, patch digest, explicit `agent_execution_policy`,
  verification commands, environment, and artifact hashes. When agent
  execution is configured, additionally bind selected Agent Runner
  adapter/version, executable source/digest and conformance digest, auth mode
  and entitlement class, execution-funding mode, credential and usage-billing
  ownership, and actual model/prompt/tool provenance.
- Invalidate dependent evidence when any bound input changes.
- Capture pre-existing repository failures before patching.
- Treat deterministic, agent-assisted, and manual patch provenance separately.
- Treat model completion as generation evidence only.
- Verification commands and scoring must be independently reproducible from the
  exact candidate head.
- Independent verification runs in a fresh process and view with frozen
  command/config digests, no Agent Runner/model credentials, and no
  generation-owned evidence write handle.
- Production evidence is outside the MVP.
- Cleanup failure, boundary violation, redaction uncertainty, stale evidence,
  or missing causal execution blocks stronger verification labels.
- Keep consumer-private prompts, responses, code, diffs, logs, and traces
  outside public source and provider-visible evidence.
- Only non-resolving opaque holdout commitments may be committed. Held-out
  inputs and answer material remain evaluator-controlled and unavailable to
  `task-executor`.
- Independent lifecycle artifacts bind the exact task, lifecycle run,
  validation run, candidate digest, and work-proof marker. The implementation
  worker cannot self-grade or self-attest.
- Historical evidence proves only its recorded semantics.

## 9. Required Validation

For normal changes, run:

- `make lint-fast`
- `make test-fast`
- `make test-coverage`
- `make test-contracts`

Before PR or merge, run:

- `make prepush-full`
- `make lifecycle-evidence`

`make prepush-full` validates source, planning, historical candidate controls,
tests, and build without requiring its own not-yet-written work-proof marker.
The immutable M1 markers, task reports, review, holdout, retained original-head
bundle, landed-content proof, exact-main checks, and non-reusable process
exception are checked by `make lifecycle-evidence`. The command validates their
historical bindings and mutation resistance; it does not re-evaluate the
frozen M1 candidate against later repository changes or grant reuse. CI runs
both phases.

If any command is skipped, record the exact reason in validation evidence.

GitHub Actions `validate` runs the same full gate. CodeQL Go analysis is the
security scanner risk lane. Coverage misses require an approved scoped
exception.

Passive Codex review settle is required before merge. Green CI alone is not
merge-ready when Codex review is enabled. Do not merge manually through
`gh pr merge`, the GitHub UI, or a connector until the latest PR head has the
configured terminal review evidence.

GitHub `main` remains protected by pull-request review, strict `validate` and
`CodeQL analyze` checks, admin enforcement, conversation resolution, and
force-push/deletion protection, including the
`protect-main-from-direct-push` ruleset. Audit it with
`make audit-remote-protection`.

The PR lifecycle report path remains:

```text
.factory/artifacts/pr-lifecycle/<work_item_id>/pr-lifecycle-report.json
```

## 10. Runtime And Distribution Pins

- Language: Go.
- Go version: `1.26.5`.
- Module: `github.com/Clyra-AI/lumyn`.
- Product status: v3.1 planning and compiled controls only; bounded-agent
  execution is not implemented.
- Initial distribution: explicitly licensed, integrity-signed design-partner
  local runner or consumer-controlled CI package.
- Consumer execution: local or consumer-controlled CI.
- Target ecosystem: TypeScript/Node and one official npm SDK per run.
- Generation: deterministic transform or bounded agent, selected per approved
  plan item.
- Verification: deterministic, pinned, baseline-aware, and independent of
  generation mode.
- Model route: exact model provider, model/version, endpoint, credential,
  disclosure, retention, token, time, retry, and cost policy before use.
- Factory artifact namespace: `.factory/artifacts/`.
- Public OSS/self-serve and Homebrew require a separate approved license,
  security, contribution, support, vulnerability-response, and release
  integrity gate.

Changing runtime, execution plane, target language, authority, model egress,
credential/network posture, distribution, or active plan path requires an ADR
or explicit decision update before implementation.

## 11. Factory And factoryd Operation

Factory owns the planning, task-packet, validation, review, shipping, and
evidence contracts. The repo-local v3.1 control set is planning authority, not
factoryd execution proof.

factoryd dispatch remains paused until a separate, reviewed change:

1. rebaselines the external Factory `profiles/lumyn.yaml` profile;
2. proves the factoryd bundle/runtime can validate and execute the exact active
   mission without stale or shallow narrative-derived semantics;
3. reconciles the checked-in paused configs with that qualified runtime; and
4. explicitly authorizes the selected task and positive runtime budgets.

The separately packeted attended M1 public/synthetic corpus and
developer-harness implementation is closed and historical. The current
rebaseline authorizes no task implementation, Lumyn product runtime, live
Agent Runner/model, consumer repository access, product command execution,
GitHub write, or merge. M2.5 requires a separate external-evidence preflight
and explicit task authorization.

Runner-ready packets include exact acceptance item IDs, dependencies, paths,
commands, risk, lifecycle gates, evidence, proof level, runtime pins,
capabilities, budgets, stop conditions, changelog/versioning intent, and
semantic invariants.

Conditional Factory capabilities are not ambient grants. Activation must bind
one frozen task/action mode, the exact selected capability set, evidence ref,
scope digest, and expiry; every selected capability must be granted as one
complete set.

Product workers may write task-scoped evidence but must not mutate active
planning truth, lifecycle evidence, or PR-lifecycle evidence.

The canonical implementation-to-merge chain is:

1. `task-executor`
2. `validation-gate`
3. `code-review` when risk requires it
4. `holdout-evaluator` when selected by policy
5. `trace-grader` when selected by policy
6. `evidence-attestor` when selected by policy
7. `commit-push`
8. `post-merge-monitor`
9. `repair-feedback` when a gate fails

Independent workers must produce task-bound, current-work-proof, passing
artifacts before `commit-push`. Do not use deprecated lifecycle aliases.
For historical M1, `code-review` preceded `holdout-evaluator`; the review does
not claim access to the later holdout result. The strict gate binds both
producers against the validation report's task-executor identity and rejects
self-review, self-provisioning, cross-candidate replay, reversed chronology,
later-workspace reinterpretation, or reuse of M1's consumed authorization and
task-specific process exception.

## 12. Stop Conditions

Stop and request a human decision when:

- provider, Lumyn operator, model provider, and consumer authority are unclear;
- provider intent is unconfirmed or would execute supplied code;
- repository access lacks an exact active grant;
- a read-only phase would mutate state;
- the approved plan no longer matches patch inputs;
- a model route lacks exact disclosure, endpoint, credential, tool, network,
  token, time, retry, or cost policy;
- repository or provider content attempts to widen authority or tools;
- agent output would be treated as verification or self-approved;
- a path, diff, command, network, credential, branch, or PR boundary is
  ambiguous;
- a repository command lacks enforceable host isolation;
- required business input is missing;
- production access would be required;
- repository tests require unapproved network or lifecycle scripts;
- redaction or evidence freshness is uncertain;
- held-out inputs or answer material would be visible to an implementation
  worker;
- an implementation worker could write independent lifecycle evidence;
- provider-visible data exceeds consumer consent;
- a remote branch or PR write is inferred from a patch, local branch, or PR
  bundle;
- the default branch or auto-merge is requested;
- a task depends on product-signal evidence that does not exist;
- a task treats this planning rebaseline as product implementation authority;
- factoryd dispatch is attempted while the mission is paused, the external
  Factory profile or runtime is unqualified, or the active v3 control surfaces
  disagree;
- required CI, coverage, CodeQL, review, `commit-push`, post-merge, or
  item-level closure evidence is missing without an approved exception.

---
> Source: [Clyra-AI/lumyn](https://github.com/Clyra-AI/lumyn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
