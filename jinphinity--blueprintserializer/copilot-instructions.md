## blueprintserializer

> > **START HERE — before reading anything else, read the universal core context file:**

# BlueprintSerializer Agent Guide

> **START HERE — before reading anything else, read the universal core context file:**
>
> **`Plugins/BlueprintSerializer/blueprintserializer_cortext.md`**
>
> That file is the single source of truth for this plugin: build rules, regression
> workflow, architectural patterns, key source files, and baseline metrics.
> Everything below is a quick-reference supplement to it.

---

## Canonical References (read in this order)

1. **`blueprintserializer_cortext.md`** ← universal context, always first
2. **`gh issue list --repo Jinphinity/BlueprintSerializer --label implementation`** ← live work queue
3. **`git log --oneline -20`** ← recent changelog
4. `REGRESSION_BASELINE.json` — metric thresholds (machine-readable)
5. `ARCHIVE.md` — historical work item and run history (read-only reference)

---

## Work Discovery

- **Open issues with label `implementation`** = active code work items → close when code is done + regression passes
- **Open issues with label `validation`** = implemented, need verification against export batch → close with metric evidence
- **Closed issues with label `corpus-blocked`** = not reopenable without new corpus evidence
- **Closed issues with label `implementation` or `validation`** = done; see commit linked in close comment
- **NEVER reopen a `corpus-blocked` issue without new corpus evidence**

```bash
# Find your next task
gh issue list --repo Jinphinity/BlueprintSerializer --label implementation --state open

# Find items to validate and close
gh issue list --repo Jinphinity/BlueprintSerializer --label validation --state open

# Check overall progress
gh issue list --repo Jinphinity/BlueprintSerializer --state open
```

---

## Required Workflow for Any Non-Trivial Change

1. Read `blueprintserializer_cortext.md` (build rule, patterns, baseline)
2. Make code changes
3. Build — **host project editor target, not plugin name** (see cortext for command)
4. Run regression suite: `powershell -ExecutionPolicy Bypass -File Plugins/BlueprintSerializer/Scripts/Run-RegressionSuite.ps1`
5. Confirm `suitePass=True` in the JSON artifact — not from shell exit code
6. Update `REGRESSION_BASELINE.json` if new metrics were added
7. Close the GitHub Issue with metric evidence in the comment
8. Commit plugin + superproject with descriptive message referencing the issue number

---

## Anti-Regression Rules

- Do not claim success unless artifact files exist and contain expected metrics
- `suitePass=True` in `BP_SLZR_RegressionRun_<timestamp>.json` is the only valid success signal
- Treat Unreal command timeouts as inconclusive; confirm via log and report JSON files
- Never close a GitHub Issue without corpus evidence from a passing regression run

---

## Scope Priority

- **Primary:** C++ conversion fidelity — extraction completeness, node meta enrichment, dependency closure
- **Secondary:** Editor-heavy animation authoring — human-led, automation assists only

---

## Changelog

All work history is in `git log`. All historical work item tables and regression run logs are in `ARCHIVE.md`.

---
> Source: [Jinphinity/BlueprintSerializer](https://github.com/Jinphinity/BlueprintSerializer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
