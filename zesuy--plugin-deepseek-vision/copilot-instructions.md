## plugin-deepseek-vision

> Use GPT-5.6 Sol as the default primary agent. The primary agent owns the

# Sol-primary Engineering Rules

Use GPT-5.6 Sol as the default primary agent. The primary agent owns the
requirements, architecture, core implementation, integration, acceptance, and
final report. Delegation exists to isolate context-heavy, blocking, remote, or
environmentally noisy work; it is not an escalation ladder.

Do not use the role-specific `luna_worker` or `sol_advisor` agent types for this
project. Spawn generic subagents with an explicit model when model selection is
needed. Use GPT-5.6 Luna Max for ordinary delegated execution and choose a
reasoning effort appropriate to the packet. Use GPT-5.6 Sol for independent
review and choose a reasoning effort appropriate to the risk.

## Primary-agent ownership

The Sol primary agent is responsible for:

- interpreting requirements and maintaining the end-to-end plan;
- architecture, interface, compatibility, security, privacy, and data-integrity
  decisions;
- focused structural exploration and small, targeted source reads;
- core code changes and core refactors;
- integrating subagent evidence and resolving conflicting findings;
- inspecting the actual diff and deciding which validation is sufficient;
- handling review findings and delivering final acceptance.

Core implementation must remain with the primary agent. Delegate a code change
only when it is mechanical, independently scoped, and has disjoint file
ownership; do not delegate a change that determines architecture, public
behavior, error semantics, security posture, or compatibility.

## Delegation triggers

Start a subagent when a task is primarily useful as isolated evidence and any
of the following applies:

- it requires broad, read-only exploration across many files or directories and
  would add substantial raw material to the primary context;
- it runs tests, builds, packaging, deployment preflight, log monitoring, or
  another command that may block or take significant time;
- it requires network access, remote-state inspection, downloads, CI log
  collection, or external API investigation;
- it requires iterative work in a complex environment such as containers,
  toolchains, platform-specific runners, dependency resolution, or deployment
  infrastructure;
- it is an independent review of a completed change;
- it is mechanical work with explicit, disjoint file ownership and no unresolved
  design decision.

These are judgment-based triggers, not rigid file-count thresholds. Prefer a
subagent when the primary agent needs a concise conclusion and evidence rather
than the raw exploration or command transcript. Do not delegate a small,
targeted read or a compact CodeGraph query merely because it is exploratory.

Network or deployment delegation does not expand authorization. Read-only
research, preflight, monitoring, and post-deployment verification are suitable
for subagents. Publishing, pushing, traffic switching, migration, rollback, or
other external mutations require the same explicit scope and authority they
would require if the primary agent performed them. The primary agent must
retain control of high-impact or irreversible actions unless the user clearly
authorized an end-to-end delegated operation.

## Subagent packets

Every delegated packet must state:

- objective and relevant context;
- in-scope and out-of-scope files, systems, and actions;
- whether the packet is read-only or may write, with one owner per writable
  file;
- constraints and non-negotiable behavior;
- acceptance criteria and exact validation where applicable;
- the expected concise return: conclusion, evidence, changed files, validation,
  and unresolved risks;
- stop and escalation conditions.

Subagents must not dump large raw outputs into their return when a summary with
precise file locations, commands, error excerpts, or artifact paths is enough.
They must stop on material ambiguity, unexpected interface or dependency
changes, security or data-integrity impact, unavailable validation, meaningful
scope expansion, or two evidence-based failed attempts.

Subagents are not alone in the workspace. They must preserve unrelated and
concurrent edits, avoid reverting work they do not own, and adapt to changes
already present in shared files.

## Coordination and waiting

The primary agent tracks all subagent work and must not duplicate an operation
that a subagent is already performing. Continue only with core work that is
safe and independent of outstanding results.

When the next step depends on a subagent result and no independent primary work
remains, the primary agent must enter a wait state. After the result arrives,
inspect the evidence and any actual changes before continuing. Do not treat a
subagent summary as acceptance by itself.

## Testing, builds, deployment, and remote work

The primary agent defines the validation scope and acceptance criteria. A
subagent normally executes and waits for tests, builds, packaging, deployment
preflight, remote checks, and environment diagnostics, then returns a compact
result with the exact commands, status, key failures, and relevant artifact or
log locations.

The primary agent diagnoses the implications, makes core fixes, and decides
whether reruns or additional coverage are required. A failed delegated command
is evidence to evaluate, not an automatic reason to let the subagent redesign
or rewrite the core implementation.

## Independent review

Review completed implementation with a fresh generic subagent using GPT-5.6
Sol. Review is read-only unless the user explicitly asks otherwise. The review
packet must include the requirements, actual diff, relevant contracts, and
validation evidence.

The reviewer should focus on correctness, regressions, compatibility, security,
data integrity, error handling, race or resource risks, and missing tests. It
must report concrete actionable findings ordered by severity and identify the
affected files and locations. If it finds no issue, it should state which risk
areas it checked.

The primary agent evaluates every finding, performs any core fixes, reruns the
necessary validation through appropriate delegated packets, and retains final
acceptance responsibility.

## Acceptance

Before reporting completion, the primary agent must:

- inspect the actual diff and preserve unrelated user changes;
- confirm that delegated changes stayed within their assigned ownership;
- evaluate validation output and review findings rather than accepting summaries
  mechanically;
- resolve or clearly report remaining risks and unavailable validation;
- avoid claiming that a model, command, test, deployment, or review ran unless
  the agent activity or tool result proves it.

If a requested model is unavailable, report the limitation and use the best
available safe route without silently restoring the removed Luna-first or
advisor-escalation scheme.

---
> Source: [Zesuy/Plugin-Deepseek-Vision](https://github.com/Zesuy/Plugin-Deepseek-Vision) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
