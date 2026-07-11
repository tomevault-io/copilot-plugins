## agent-orchestration-harness

> - `Remove-Item -Recurse`

# Agent Orchestration Harness Instructions

## Hard Safety Rules

禁止批量删除文件或目录。

不要使用：

- `del /s`
- `rd /s`
- `rmdir /s`
- `Remove-Item -Recurse`
- `rm -rf`

需要删除文件时，只能一次删除一个明确路径的文件。

正确示例：

```powershell
Remove-Item "C:\path\to\file.txt"
```

如果需要批量删除文件，应停止操作，并询问用户，让用户手动删除。

## Project Gates

- Do not begin `P0-01` until `P0-00` has been rerun and accepted by a human.
- Do not begin `M0` until all P0 runtime-control tasks pass.
- Do not implement M1 runtime code during P0 or M0.
- Do not modify `src/**` during P0-00 remediation.
- Do not install Codex, Claude SDKs, or project dependencies unless a task contract explicitly allows it.

## Sandbox / Permission-Profile Rules

- Every write-root listed in any `.codex/config.toml` permission profile (e.g. `docs/tasks`, `evidence`, `.evidence-local`, `schemas`, `scripts`, `tests`) MUST, before Codex is launched or restarted, (a) exist as a directory, AND (b) be owned by the current user (e.g. `Holly`) — NOT by `BUILTIN\Administrators`.
- Reason: on startup the Windows sandbox setup helper grants the sandbox user a write ACE on each write-root via `SetNamedSecurityInfoW`, which requires `WRITE_DAC`. The helper runs non-elevated (UAC-filtered token), so it has `WRITE_DAC` only on user-owned (or sandbox-owned) dirs. If a write-root is missing, OR is owned by `BUILTIN\Administrators` (which happens when it was created from an ELEVATED terminal), the grant fails with `SetNamedSecurityInfoW failed: 5` and the **entire** setup refresh aborts with `helper_unknown_error: setup refresh had errors` — after which no command, file read, or test can run.
- The leaf-only profile makes the repository root read-only, so a running agent CANNOT `mkdir` a missing top-level write-root from inside the sandbox. Create it externally first — from a **non-elevated** (non-Administrator) PowerShell, so it is owned by the user:

```powershell
New-Item -ItemType Directory -Force G:\agent-orchestration-harness\<new-write-root>
(Get-Acl G:\agent-orchestration-harness\<new-write-root>).Owner   # must be MS-...\Holly, not BUILTIN\Administrators
```

- Do NOT create write-roots from an elevated/Administrator terminal — that yields `BUILTIN\Administrators` ownership the non-elevated helper cannot re-ACL. If a write-root is already Administrators-owned, delete and recreate it non-elevated (it must be empty first).
- Diagnostic: if you hit `helper_unknown_error: setup refresh had errors`, grep `.codex/.sandbox/sandbox.<date>.log` for `SetNamedSecurityInfoW failed: 5` — the named path is the offending write-root. Fix its existence/ownership, delete `.codex/.sandbox/setup_error.json`, and restart Codex.

## Evidence Rules

- Do not persist raw command output as evidence unless a redaction gateway is in place.
- Prefer sanitized summaries, hashes, and manifests.
- Keep raw local logs under `.evidence-local/`; do not commit them.
- Do not read or copy `.env`, credentials, tokens, SSH keys, or other secret-bearing files.

## Workflow Rules

- Each P0 subtask needs a task contract, assignment, acceptance criteria, evidence requirements, and stop conditions.
- Planner outputs must pass schema validation and deterministic lint before execution once those tools exist.
- Worker output must be reviewed by an independent reviewer before downstream use.
- Coordinator state transitions must be explicit and append-only once the state machine exists.

---
> Source: [submerge12/agent-orchestration-harness](https://github.com/submerge12/agent-orchestration-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
