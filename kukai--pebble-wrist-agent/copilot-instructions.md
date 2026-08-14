## pebble-wrist-agent

> This file routes tasks to the right process level. Details live in

# CLAUDE.md — Workflow Router

This file routes tasks to the right process level. Details live in
`.claude/rules/` and `.aidlc-rule-details/` (loaded only when triggered).
Do NOT add project knowledge here; put it in the section at the bottom.

## 1. Task Routing (evaluate top-down, first match wins)

**Tier 2 — AI-DLC** (declare by starting work with `Using AI-DLC`):
Applies if ANY of the following is true:
- Requirements are not yet written down in `aidlc-docs/` and cannot be
  stated in ≤3 sentences without open questions
- The change spans 2 or more top-level components (distinct top-level
  directories under the source root, or separate services/packages)
- A new project, feature, or module is created from scratch
→ Follow `.aidlc-rule-details/`. Record all artifacts under `aidlc-docs/`.
→ If the user did not declare `Using AI-DLC` but criteria match: STOP and
  propose it. Do not silently start heavyweight process.

**Tier 1 — Plan mode:**
Applies if ANY of the following is true (and Tier 2 does not apply):
- 3 or more files will be edited, or any new file is created
- A public interface, DB schema, config format, or dependency changes
- The task involves deleting or rewriting an existing feature
→ Enter plan mode, present the plan, wait for approval, then execute.
→ Inside an active AI-DLC construction phase, individual implementation
  tasks also use this tier (Plan mode nests inside AI-DLC).

**Tier 0 — Direct execution:**
Everything else: read-only questions, edits to 1–2 existing files with a
clear spec, doc fixes, running tests.
→ Execute immediately. No plan, no ceremony.

**Escalation rule:** If unsure between two tiers, start with the lighter
one. The moment scope exceeds its criteria mid-task, STOP, say so, and
upgrade. Never downgrade silently.

## 2. Persistent Memory (survives sessions — use it)

- Decisions live in `aidlc-docs/decisions/` by default. If `Decisions
  location` is set in Project-Specific below, that path is the single
  source of truth instead — never maintain both.
- BEFORE designing or planning anything: check the decisions location
  for a file matching the feature. If it exists, read it first and treat
  its decisions as fixed unless the user reopens them.
- AFTER any design decision is approved: append it to
  `aidlc-docs/decisions/<feature>.md` (one file per feature, append-only,
  each entry dated).
- AFTER completing a Tier 1/2 task: append a 3–5 line entry to
  `aidlc-docs/log/YYYY-MM.md`: what changed, why, gotchas found.
- Session context is disposable; `aidlc-docs/` is the source of truth.

## 3. Self-Improvement (proposal only — never self-apply)

After completing a Tier 1/2 task, IF rework occurred (a plan was revised
after approval, or code was rewritten after review), you MUST output a
short retrospective:
1. Where the rework happened and its root cause
2. A concrete diff proposal for CLAUDE.md or `.claude/rules/*`
Then STOP. Never edit CLAUDE.md or `.claude/rules/` yourself. When the
user approves a proposal, apply it and append one line to
`aidlc-docs/changelog.md`: date, change, reason.
If no rework occurred, output nothing. Do not pad.

## 4. Hard Rules

- Never commit or push without explicit user instruction.
- Never write secrets, tokens, or credentials into any file in this repo.
- Cloud/web sessions: this repo is your entire context. If something
  seems missing (env vars, credentials), say so; do not improvise.
- Before implementing a behavior change requested via an ambiguous,
  emotionally-loaded verb ("reset", "clear", "fix"), restate the
  expected end state as concrete before/after examples (what the
  screen shows, what state persists) and get confirmation — before
  writing code. Do not iterate by shipping one interpretation at a
  time and waiting for correction.
- When a working tree straddles multiple long-lived branches (e.g.
  a feature branch plus a disposable `staging` branch), run `git
  branch --show-current` immediately before every commit. Do not
  assume the checkout left by a prior step.

---

## Project-Specific (edit per project; keep under 30 lines)

- Decisions location (leave blank to use aidlc-docs/decisions/): `docs/adr/ADR.md`（追記専用・既存番号維持）
- Language / runtime: C (Pebble SDK 3, watch) / ES5 JavaScript (PebbleKit JS, phone)
- Build: `pebble build` / `pebble install --emulator emery` / `pebble logs`
- Test: なし（エミュレータ・実機での手動確認）
- Lint: なし
- Conventions that differ from language defaults:
  - pkjs は ES5 必須（アロー関数・`const`/`let`・`findIndex` 等禁止）
  - C 内の日本語リテラルは UTF-8 エスケープ `\xNN` で記述
- 現行仕様は `SPECS.md`（上書き更新のスナップショット）、AppMessage キーは `appinfo.json` を参照
- 設定 Web UI: `config/index.html`（GitHub Pages でホスト）
- GitHub 運用: PR タイトルは英語・本文は日本語。main へのマージ前に必ず
  `staging` ブランチ経由で CloudPebble ビルドを確認（skill: `pre-merge-staging`）
- 詳細な過去の経緯・制約は `CLAUDE.legacy.md` と `docs/adr/ADR.md` を参照

---
> Source: [kukai/pebble-wrist-agent](https://github.com/kukai/pebble-wrist-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
