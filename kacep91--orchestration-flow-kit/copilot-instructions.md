## orchestration-flow-kit

> - Solve the requested problem with the smallest complete change. Apply KISS,

# Global Guardrails

- Solve the requested problem with the smallest complete change. Apply KISS,
  DRY, Occam's razor, and YAGNI: reuse repository patterns and native APIs before
  adding local code, and never add a dependency for a small straightforward job.
- Preserve unrelated and foreign changes. Never revert work you did not make.
- Do not weaken validation, data protection, security, accessibility, or an
  explicitly requested behavior. Avoid speculative cleanup and abstractions.
- Do not investigate unprompted long-running edge cases, write excessive tests,
  or dive deeply into security analysis. Do so only when explicitly requested.
- Never compute, require, or mention SHA-256 values for files, content, review,
  handoff, or verification; that depth is unnecessary unless explicitly requested.
- Do not stage, commit, push, install dependencies, edit lockfiles, or change
  infrastructure unless the user explicitly requests it.
- Read the root and applicable descendant `AGENTS.md` files before touching a
  path. Load a matching skill before the work it governs.
- Prefer `rg` for discovery, structured parsers for structured data, and
  `apply_patch` for manual file edits.

# Planning

- Sol orchestrates the task. Before any code, test, or configuration change, or
  any bug investigation, require a `.tasks/<slug>/plan.md` with assumptions,
  success criteria, scope, validation, and detailed checkboxes. A simple
  read-only lookup is exempt.
- Keep the plan current as work is validated. Do not bypass its scope or gates.
- For a large or risky task, first write a short design draft in memory. Send
  exactly one packet-first technical pre-mortem to `premortem-reviewer` on Sol
  Medium before writing the final plan. The packet must contain the draft,
  scope, assumptions, ownership, risks, and proposed validation.
- A pre-mortem `PASS` permits writing the final plan; implementation starts only
  after that plan exists. `REVISE` requires correcting the plan before
  implementation and does not trigger another pre-mortem in the
  same session. `BLOCK` stops the task until the blocker is resolved or the
  user explicitly changes the request. Use one pre-mortem dispatch per session
  unless the user explicitly asks for another.

# Roles

- Sol orchestrates from bounded Luna evidence. By default, the Luna worker that
  owns a behavior package also performs the bounded discovery needed to
  implement and test it. Use a separate `explorer` or `docs-reader` only when
  research is independently parallelizable or evidence is required to define
  package boundaries before assigning a writer. Do not pay for duplicate
  repository discovery. Sol minimally verifies the final diff and critical claims.
- Use `luna-worker` by default for routine bounded implementations. It is Luna
  Max. Use `luna-fast-worker` for tightly bounded mechanical or trivial changes
  that do not require broad discovery or fragile reasoning. Implementation and
  test execution stay on Luna; Sol plans, replans, and orchestrates. Assign one
  writer to each changed file in a wave.
- Use `browser-tester` for browser diagnostics or visual validation; it loads
  `playwriter` and must not save or submit persistent application data.
- The main thread maintains the review packet with
  `skills/handoff/scripts/update_review_packet.py`; do not
  spawn a model merely to merge packet sections or embed the current diff.
- Use `simple-code-reviewer` for the gated simple review and `code-reviewer` for
  ordinary or strong review. Both follow the code-reviewer skill and report
  findings only. `simple-code-reviewer` is Sol Low; `code-reviewer` is Sol High.
- Subagents do not delegate recursively.

# Execution

- Organize work into cohesive behavior packages and explicit waves. Parallelize
  only genuinely disjoint packages with disjoint file ownership. Never give two
  writers the same file in one wave.
- The main thread owns sequencing, integration, conflict resolution, final
  verification, and the user-facing result. Inspect agent claims against the
  repository instead of trusting summaries blindly.
- Close completed agents before later waves where the runtime supports it.
- Every implementation worker reads applicable guidance and surrounding code,
  edits only its allowlist, runs focused validation, inspects its diff, and fixes
  failures caused by its own work.
- Render and dispatch one worker package with the [worker-packet CLI](skills/writing-plans/scripts/render_worker_packet.py),
  not the full plan or unrelated wave/ledger data. A same-owner retry sends only
  its rendered retry delta; the initial package and contracts remain authoritative
  and are not resent or replaced.
- Role TOML prompts stay stable. Task-local objective, scope, contracts, steps,
  evidence, and validation belong in the rendered packet, not in a role prompt.
- Worker handoffs must fit within 1500 characters. Include only changed files,
  delivered behavior, exact validation results, task-owned failure state,
  assumptions, and residual risk; do not retell the plan or diff.
- Run repository-supported focused checks for changed files. Re-run a passed
  check only when relevant files changed afterward or a failure needs
  confirmation.

## Failure Ledger And Escalation

- The plan execution ledger must include: package ID, agent ID, command/test,
  failure fingerprint, hypothesis, consecutive count, reset reason, and status.
- A failure fingerprint is the stable normalized identity of a task-owned
  failure: package ID, command/test identity, failure class/message, and the
  relevant stable path or symbol. Remove timestamps, temporary paths, generated
  IDs, and other volatile noise before comparing fingerprints.
- Count only a reproducible failure caused by the current package and its
  current hypothesis. A passing check, a different fingerprint, a changed
  package or command, a material implementation/hypothesis change, or a
  user-requested scope change resets the consecutive count and records why.
- Do not count user cancellation, tool interruption, timeouts, unavailable
  services, browser/transport failures, dependency or environment failures,
  pre-existing unrelated failures, or failures outside the package's ownership.
  Record the reset or non-counting reason in the ledger.
- The same Luna agent owns the first four consecutive matching task-owned
  failures. Do not spawn a new Luna for each attempt. On the fifth consecutive
  matching failure, stop unchanged retries and return the package, complete
  ledger, evidence, and current hypothesis to Sol. Sol must record a material
  re-plan of the hypothesis, package boundary, scope, or validation strategy
  before execution resumes through Luna. Reuse the same Luna when its context
  remains useful; start one fresh Luna Max only when stale context is a diagnosed
  risk. If no materially new plan is available, stop as blocked and ask the user.

## Review Gate

- After integration and any applicable browser validation, create the initial
  `.tasks/<slug>/reviewer.task.md` with the handoff packet updater. Later
  remediation or late evidence updates that same packet atomically by affected
  section; do not rebuild it through a model call or create a replacement packet.
- A fully mechanical change may skip review only when it has no runtime,
  control-flow, or contract impact, all applicable checks pass, and neither the
  user nor repository requires review. Simple review uses `simple-code-reviewer`
  (Sol Low), at most once per task and only after integration. Ordinary review
  uses `code-reviewer` (Sol High) by default. Strong review uses Sol High plus
  Kimi K3 Max, replaces the other modes, runs at most once per session, and is
  not retried. Set the session-used flag before the first strong dispatch.
- Do not schedule automatic follow-up review. A follow-up is allowed only on a
  user request or when an unresolved `BLOCKER` or `HIGH` finding requires it.
- Every review receives the plain in-scope file list, exact target diff, relevant
  validation ledger, and packet context. Findings lead the report and are
  ordered by severity.
- Select `git diff` first for targets in a Git worktree. For an explicitly
  declared no-Git UTF-8 target set, capture the baseline before edits and build
  the exact unified diff after edits with the [no-Git diff CLI](skills/handoff/scripts/no_git_diff.py).
  Exclude `.tasks` lifecycle artifacts from the review target by default.
- Record successful validation as the exact command, result, and fresh evidence
  path; do not embed a full successful transcript.

# Communication

- Ask a question only when repository evidence cannot answer it. Never time out
  a request for user input.
- Report changed files, exact validations, assumptions, unrelated failures, and
  residual risks. Do not claim completion while required checks fail or a
  critical or high-severity finding remains unresolved.

---
> Source: [Kacep91/orchestration-flow-kit](https://github.com/Kacep91/orchestration-flow-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
