## pg-brakes

> Concise operating rules for everyone (humans and agents) building this repo.

# CLAUDE.md — engineering principles for pg_brakes

Concise operating rules for everyone (humans and agents) building this repo.
Current product claims are governed by [`capabilities.json`](capabilities.json),
[`ADR-0001`](docs/adr/0001-v0-1-safety-postures.md), and
[`CONTEXT.md`](CONTEXT.md). The files under [`docs/spec/`](docs/spec/) are frozen
historical design records; they are useful background, but they do not describe
the current supported envelope where the current sources supersede them. The
**process** spec lives in GitHub issue #1 and this file.

---

## 1. Red/green TDD — the failing test comes first, always

- Write the **failing test first**, watch it fail (RED), then make it pass (GREEN).
  Capture both states in the PR (paste the RED output and the GREEN output).
- Every crate and every behavior change ships with at least one real test.
- No production code without a test that motivated it.

## 2. Engineering posture: deterministic floor + fail-closed

- The **deterministic floor** is the safety guarantee: native-role WALL, the
  proxy's Recognized Read Subset and budgets, timeouts, and posture-specific
  write refusal or certification. No model is in that path.
- **Fail closed:** absence of required signal means least privilege; on doubt,
  deny or abort. Production Read disables write tools. Experimental Write is
  non-production and accepts only the tiny Certified Write Envelope in ADR-0001:
  one schema-qualified ordinary table, `UPDATE` or `DELETE`, exact equality on
  its catalog-discovered sole `int4` primary key or a non-empty literal key list,
  literal assignments, no primary-key update, no triggers or other excluded
  relation semantics, and full-row pre-images. Rehearsal records the exact key
  set on a checked-distinct operator-provisioned target; signed, expiring,
  single-use approval binds it; apply and revert revalidate catalog identity,
  exact cardinality, and after-image state. Unsupported or uncertain shapes are
  refused before candidate SQL executes.
- The LLM risk-gate is **tighten-only** — it can block/hold/escalate but can **never
  loosen** below the floor. In the MVP the `RiskEngine` is a stub returning `Allow`
  and intent tiers T0–T2 are captured/logged only (SPEC §15.1).

## 3. Roles

- **Manager** (engineering manager): coordinates only — never writes code, tests, or
  reviews personally. Creates issues, dispatches agents, drives the PR lifecycle.
- **Implementer:** picks up one issue in its **own git worktree**, builds it red/green,
  verifies all CI commands locally, opens a **draft PR**, posts evidence. **No self-merge.**
- **Reviewer:** a **different** agent (never the author) reviews every PR by running the
  **real samorev agents** (§4, step 2) — not a generic reviewer merely *told* to "apply samorev" —
  and posts a verdict. Reviews are mandatory.

## 4. PR lifecycle — enforce in order, loop until satisfied

1. **CI green** — all jobs pass on the PR (paste the run link).
2. **samorev review** — run the **actual samorev review agents**
   ([samorev](https://github.com/Tanya301/samorev), checked out locally at `~/github/samorev`,
   Apache-2.0), **not** a generic reviewer merely told to "apply samorev." samorev has **two
   surfaces**: (a) the **Bun CLI `samorev review --fetch`** is a *deterministic* gate that
   checks **CI status + draft state only** and runs **no** AI agents (its
   Security/Bugs/Tests/Guidelines/Docs rows are always `0`); (b) the **`/review-mr` Claude Code
   slash command** runs the **5–6 parallel LLM review agents**. `/review-mr` targets both GitLab
   MRs and GitHub PRs (its posting/report formatting is GitLab-leaning), and the agents
   themselves are provider-agnostic — pg_brakes is on **GitHub**, so the reviewer runs **the
   agents directly** against the GitHub PR diff, each loading its real definition from
   `~/github/samorev/agents/*.md`:
   **security-reviewer** (Opus, **blocking**) · **bug-hunter** (Opus, **blocking**) ·
   **test-analyzer** (Sonnet, non-blocking/configurable) · **guidelines-checker** (Sonnet,
   non-blocking) · **docs-reviewer** (Sonnet, non-blocking). (samorev's optional
   **sqitch-migration-checker** is **N/A** — pg_brakes is Rust with no Sqitch migrations.)
   The **guidelines-checker** loads the project's repo-specific rules — for pg_brakes that is
   **this CLAUDE.md** (there is no separate rules dir to point at). Score each finding with
   samorev's **0–10 confidence** and its three tiers (blocking / non-blocking / potential); a
   finding **blocks merge** when its severity is **CRITICAL/HIGH/MEDIUM** and **confidence ≥ 8**
   (samorev's agent-agnostic `classify_finding` keys on severity + confidence only, never on
   which agent raised it). In practice that means **security-reviewer or bug-hunter**, since the
   other in-scope agents emit mostly lower-severity findings (the optional sqitch checker is
   N/A here).
   - **Omit all SOC2 items — they are NOT relevant to this project** (a single self-hostable
     OSS control plane with no SOC2 scope; samorev's `/review-mr` SOC2 check is optional and we
     leave it **off**). No agent may raise, score, or block on a SOC2 finding.
   - samorev's agents are **LLM-driven and non-deterministic**: take the **union of findings
     across runs** — never let a lucky clean re-run hide a real finding a prior run surfaced.
3. **Real testing with evidence on the PR** — command outputs, integration runs,
   numbers. Docker/integration tests are env-gated but **actually run** with evidence.
4. **Verdict by a non-author reviewer** — a formal GitHub **APPROVE**, or (if the bot
   identity authored the PR and APPROVE is blocked) a COMMENT review starting
   **"REVIEWER VERDICT: APPROVE"** / **"REQUEST CHANGES"**.
5. On approve + green + evidence → **squash-merge + delete the branch**.
   Otherwise → author fixes → **LOOP**.

> **No self-merge. Never claim green if it isn't. Never fabricate evidence.**
> If a step is skipped, say so.

## 5. License hygiene — Apache-2.0 only, GPL/AGPL banned

- The project ships under **Apache-2.0** (`LICENSE`). Dependencies must be
  **Apache-2.0 / MIT / BSD / ISC** only (SPEC §4).
- Rust: **`cargo deny check`** gates licenses, bans, advisories (RUSTSEC), and
  sources. `deny.toml` is the **AGPL guard** — GPL/AGPL/LGPL deps make the check FAIL.
  This now covers the MCP server too: it is the native Rust `pgb-mcp` (`crates/mcp`),
  a workspace member, so its deps are license-checked by the same `cargo deny` gate
  (the original non-Rust MCP server + its separate license-check script are gone —
  EPIC #83; `cargo deny` is now the single license gate for the Rust-only workspace).

## 6. pgDog clean-room rule (AGPL — inspiration only)

[pgDog](https://github.com/pgdogdev/pgdog) is **AGPL**. You may **study its approach
for inspiration**, but you must **NEVER copy its code** — not a line, not a snippet.
Everything here is a **clean-room** implementation. When in doubt, don't look;
design from the current ADR, capability registry, and upstream Postgres
documentation.

## 7. Local CI commands — run ALL of these green before pushing

```sh
# Rust workspace (single-language; the MCP server `pgb-mcp` is a workspace member
# in crates/mcp, so these commands build, test, and license-check it too).
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings
cargo build --workspace --locked
cargo test  --workspace --locked
cargo deny  check                     # licenses + bans + advisories + sources
```

CI mirrors these in `.github/workflows/ci.yml` (triggers: push + pull_request).
Toolchain is pinned by `rust-toolchain.toml` (Rust **1.90.0**, edition 2024).

## 8. Where things live

- **Current product claims:** `capabilities.json`, ADR-0001, and `CONTEXT.md`.
- **Frozen historical design:** `docs/spec/`; do not silently present it as
  current implementation status or edit it in feature PRs.
- **Current decisions / rationale:** `docs/adr/`; record new product-envelope
  changes there and update the capability registry in the same change.
- **Crates:** `crates/{proxy,warden,core,policy,clone-orchestrator,pgwire,audit,cli,mcp,applyd}`.
- **MCP server (Rust):** `crates/mcp`, binary **`pgb-mcp`** (the one and only deployable
  MCP server — EPIC #83). **Deploy/dev-stack:** `deploy/`. **Protocols:** `proto/`.

---
> Source: [NikolayS/pg_brakes](https://github.com/NikolayS/pg_brakes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
