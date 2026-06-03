## sunshine-scripts

> This repository contains Windows automation scripts for preparing and restoring a Sunshine/Moonlight streaming setup. The current implementation is PowerShell-based, with thin batch wrappers for Sunshine command integration.

# AGENTS

## Project Overview

This repository contains Windows automation scripts for preparing and restoring a Sunshine/Moonlight streaming setup. The current implementation is PowerShell-based, with thin batch wrappers for Sunshine command integration.

The runtime behavior is currently NVIDIA-focused and manages:

- virtual display enable/disable
- display resolution and refresh rate
- HDR state
- G-Sync state
- frame rate limiting
- optional RTSS integration

## Repo Layout

- `start_streaming.bat` and `stop_streaming.bat` are thin compatibility wrappers intended to launch the PowerShell entrypoints from Sunshine.
- `src/start_streaming.ps1` and `src/stop_streaming.ps1` contain the real implementation.
- `VirtualDisplayDriver/vdd_settings.xml` is the current Virtual Display Driver configuration asset used by this repo.
- `IddSampleDriver/option.txt` is kept only as legacy compatibility material for older VDD releases.
- `.local/` is ignored workspace state and must not be treated as durable project documentation.

## Implementation Rules

- Keep wrapper scripts minimal. Put real behavior in the PowerShell scripts unless there is a clear compatibility reason not to.
- Preserve the effective parameter surface across the PowerShell scripts and `README.md`.
- Keep the batch wrappers as pass-through launchers that forward arguments unchanged unless there is a specific compatibility reason to change them.
- Treat these as part of the current runtime contract unless intentionally redesigning them:
  - admin elevation behavior
  - PnP device lookup behavior
  - Sunshine-provided environment variables
- Avoid introducing new machine-specific assumptions unless they are truly required by the runtime. If a path or environment assumption is unavoidable, document it as current behavior rather than a universal rule for all machines.
- Keep docs aligned with code when changing:
  - script parameters
  - VDD lookup behavior
  - VDD config file expectations
  - tool integration behavior

## Formatting And File Conventions

- This repository uses CRLF text line endings per `.gitattributes`. Preserve CRLF when editing text files.
- Preserve existing file encodings when possible.
- `stop_streaming_task.xml` is a special case and should not be casually rewritten with generic text tooling, because its encoding matters to Windows Task Scheduler interoperability.
- Avoid unnecessary formatting-only churn in scripts and docs.

## Durable Documentation Guidance

- Distinguish between current runtime behavior and contributor guidance.
- It is acceptable to document that the current scripts expect Windows and currently reference fixed tool locations.
- It is not acceptable to generalize one contributor machine's shell setup, local authentication setup, or local executable discovery behavior into repo-wide instructions unless that behavior is intentionally required by the project.
- Any instruction that must survive clone or checkout belongs in tracked docs like `README.md` or this file, not under `.local/`.

## Validation Limits

- Agents working on this repository can usually guarantee:
  - documentation consistency
  - script syntax and basic static validity
  - consistency between wrapper scripts, PowerShell entrypoints, and tracked docs
- Agents usually cannot physically verify that a change still works end to end under Sunshine, Moonlight, Windows display settings, GPU driver behavior, or Virtual Display Driver behavior without access to a realistic test environment.
- Unless the agent has tooling that can fully reproduce the target setup, treat end-to-end runtime validation as a human verification step.
- When reporting results, be explicit about whether a change was:
  - statically reviewed only
  - validated with limited local checks
  - verified end to end on a representative runtime setup

## Docs Synchronization

- If script parameters or runtime behavior change, update `README.md` in the same change.
- If a change makes `AGENTS.md` inaccurate, incomplete, or misleading, update `AGENTS.md` in the same change with a focused edit.
- Keep `AGENTS.md` limited to durable repo guidance rather than temporary local or session-specific quirks.
- Keep current VDD guidance centered on `VirtualDisplayDriver/vdd_settings.xml`.
- Treat `IddSampleDriver/option.txt` as legacy-only documentation unless the project intentionally returns to the old VDD setup.

## Release Process

Use authenticated GitHub CLI or equivalent GitHub tooling for release publication.

Release naming pattern:

- release branch: `release/<version>`
- git tag: `<version>`
- GitHub release title: `v<version>`

Default release source:

- if the release request provides a specific release commit SHA, use that SHA
- otherwise, release from a freshly verified current tip of `origin/main`

Recommended preflight checks:

- verify the worktree is clean
- verify the intended release commit is explicitly identified
- if no release SHA was provided, fetch or otherwise verify that `origin/main` is current before using it as the default release source
- if no release SHA was provided, verify the local checkout is aligned with the verified tip of `origin/main` before creating the branch, tag, or release page
- verify the release branch does not already exist locally or remotely
- verify the release tag does not already exist
- verify authenticated GitHub access before creating the remote release

Release flow:

1. Create `release/<version>` from the intended release commit.
2. Create tag `<version>` at the same commit.
3. Push the release branch.
4. Push the tag.
5. Create the GitHub release using title `v<version>`.

Release notes style:

- use a short bullet list of included commits
- include the full commit SHA in each bullet
- include GitHub attribution where available
- end with a `Full Changelog` compare link

Example compare link shape:

```md
**Full Changelog**: https://github.com/<owner>/<repo>/compare/<previous-version>...<version>
```

Verification after release:

- verify the local branch and tag both resolve to the intended commit
- verify the remote branch and tag both resolve to the intended commit
- verify the published GitHub release metadata:
  - tag name
  - title
  - release body
  - release URL
- verify the release appears in the expected release ordering on GitHub

If a release is published with incorrect notes, correct the existing release in place rather than creating a replacement release.

---
> Source: [fehbari/sunshine-scripts](https://github.com/fehbari/sunshine-scripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
