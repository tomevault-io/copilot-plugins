## driftless

> Driftless is public OSS for applying shared Claude + Codex agent workflows. Keep

# Driftless Agent Guidance

Driftless is public OSS for applying shared Claude + Codex agent workflows. Keep
it public-safe, useful to non-developer students and maintainers, and focused on
reducing setup, git/GitHub, security, validation, and raw-script burden.

## Product Identity
- User goal: apply practical agent workflows without becoming a toolchain
  operator.
- Audience: non-developer students and maintainers who need clear setup,
  safety, evidence, and recovery paths.
- Core promise: one public shared tier, two profiles. Shared improvements live
  in `profiles/shared/` once and are consumed by both Claude and Codex profiles.
- Public docs must explain user value. Do not include private campaign notes,
  promotion plans, or internal positioning goals.

## Cross-Runtime Learning
- When maintainers prove an improvement or lesson in real use, evaluate it for
  Driftless before calling the work done.
- System/skill/hook/script/prompt changes follow
  `profiles/shared/contract/SHARED_DESIGN_CONTRACT.md` §8: root cause first,
  principle-based guidance over case rules, shared tier before tool-specific
  splits, and no spec/case overfitting or one-off special-casing unless evidence
  shows the exception lowers user effort, maintainer effort, time, tokens, cost,
  recurrence risk, or maintenance cost.
- Modular parallel development is the default for broad work: name the owned
  module surface, write set, read-only surfaces, public contract, validation
  command, rollback path, and `parallel_safe` state before implementation or
  worker split. Use the Ponytail ladder as a compact heuristic, but never remove
  validation, security, privacy, data-loss handling, accessibility,
  maintainer-visible state, or evidence gates to make a patch smaller.
- When any project/session exposes a systemic recurrence risk, apply the
  smallest safe repo-local prevention in that same session when scope is clear:
  record it through `learning-loop`, update the relevant skill/hook/script/test,
  and validate it. Stop at a proposal for host-global, credential, billing,
  destructive, private, or unclear changes.
- Public-safe, tool-agnostic improvements go to `profiles/shared/`.
- Claude-specific improvements go to `profiles/claude/`; Codex-specific
  improvements go to `profiles/codex/`.
- Private, unsafe, account-specific, promotion, or internal strategy material
  must not be copied into the public repo. Record a sanitized follow-up or
  explicit skip reason instead.

## Evidence
- Do not claim an improvement is reflected without command evidence such as
  `rg`, a relevant gate, or a real install/use check.
- Static doc changes do not prove behavior. Behavioral claims need real use or a
  bounded end-to-end test.
- Local workflow CI/CD is the default authority for routine agent work: use
  repo-local gates, scripts, harnesses, and local merge-ref validation as
  completion proof. Do not create or wait on GitHub Actions/workflow CI/CD,
  hosted checks, or request `workflow` scope unless a current
  maintainer-approved exceptional issue names it.

## Clean Primary Checkout
- Do not start non-trivial work in the primary/root checkout. Create an issue
  worktree first:
  `pwsh.exe -NoProfile -ExecutionPolicy Bypass -File .\scripts\New-IssueWorktree.ps1 -Issue <number> -Slug <short-name>`.
- If the primary checkout is already dirty, stop and report it. Do not stash,
  reset, clean, or keep editing the dirty primary checkout without manager
  approval.

## PowerShell Shell Contract

Default repo tasks:
`pwsh.exe -NoProfile -ExecutionPolicy Bypass -File .\scripts\task.ps1 <task>`.

Normal agent and manager work uses PowerShell 7 through `pwsh.exe` only.
Legacy compatibility probes stay isolated under `scripts/winps51/` and are not
the default task route. No Bash heredoc in PowerShell; use a PowerShell
here-string or a checked-in script file. Details: `docs/powershell-shell-contract.md`.

## Session Claims and Conflict Arbitration
- `New-IssueWorktree.ps1` acquires a session claim automatically. For any other
  non-trivial start, claim first:
  `pwsh -NoProfile -ExecutionPolicy Bypass -File .\scripts\New-SessionClaim.ps1 -Mode Acquire -Issue <n> -Branch <branch>`;
  Release it when the work merges or stops.
- On a claim conflict or duplicate-work finding, follow
  `docs/cross-agent-work-arbitration.md`: deterministic rules R1-R5 first, then
  one short manager question. Keep one asset name on one surface
  (`Test-WorkSurfaceDuplication.ps1` enforces this in local gates).

---
> Source: [mizan0515/driftless](https://github.com/mizan0515/driftless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
