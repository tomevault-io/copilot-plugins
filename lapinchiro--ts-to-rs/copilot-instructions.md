## ts-to-rs

> TypeScript → Rust conversion codemod CLI tool.

# CLAUDE.md

TypeScript → Rust conversion codemod CLI tool.

## Response Language

Always respond to the user in **Japanese**. Commit messages must also be in **Japanese**. Code, comments, and documentation may be in English, but conversational responses and commit messages must be in Japanese.

## Tech Stack

- **Language**: Rust
- **TS parsing**: swc_ecma_parser + swc_ecma_ast
- **CLI**: clap
- **Testing**: cargo test + insta (snapshots)
- **Lint**: clippy
- **Formatting**: rustfmt

## Key Commands

```bash
cargo build                # debug build
cargo build --release      # release build
cargo check                # fast type check
cargo test                 # run all tests
cargo fix --allow-dirty --allow-staged  # auto-fix unused imports etc.
cargo clippy --all-targets --all-features -- -D warnings  # lint
cargo fmt --all --check    # format check
cargo llvm-cov --ignore-filename-regex 'main\.rs' --fail-under-lines 89  # coverage (threshold 89%, excluding main.rs)
cargo llvm-cov --html                  # generate HTML report (target/llvm-cov/html/)
./scripts/check-file-lines.sh        # .rs file line count check (threshold: 1000 lines)
./scripts/hono-bench.sh              # Hono conversion rate benchmark (directory mode)
./scripts/hono-bench.sh --both       # both directory + single-file modes
```

### Hono Benchmark

Measures Hono framework conversion success rate. Run after conversion feature changes to quantify impact.

- **Run**: `./scripts/hono-bench.sh` (verifies `cargo build --release` and auto-clones Hono repo)
- **Analysis**: `scripts/analyze-bench.py` auto-invoked, aggregating error JSON by category
- **History**: `bench-history.jsonl` (JSONL, one line per run). View with: `python3 -c "import sys,json; [print(f\"{json.loads(l)['timestamp'][:10]} clean={json.loads(l)['clean_pct']}% errors={json.loads(l)['error_instances']}\") for l in open('bench-history.jsonl')]"`
- **Error JSON**: `/tmp/hono-bench-errors.json`
- **Error inspection**: `scripts/inspect-errors.py` — エラーの詳細分析ツール（下記参照）

**Note**: "Clean" = zero conversion errors (`--report-unsupported` with 0 errors), separate from whether generated Rust compiles.

### Error Inspection

`/tmp/hono-bench-errors.json` の詳細分析には `scripts/inspect-errors.py` を使用する。アドホックな Python ワンライナーでの解析は禁止。

```bash
python3 scripts/inspect-errors.py                        # カテゴリ別集計
python3 scripts/inspect-errors.py --kind TYPEOF          # kind で部分一致フィルタ
python3 scripts/inspect-errors.py --category TYPEOF_TYPE # カテゴリで完全一致フィルタ
python3 scripts/inspect-errors.py --file client          # ファイル名で部分一致フィルタ
python3 scripts/inspect-errors.py --source               # エラー箇所の TS ソースを表示
python3 scripts/inspect-errors.py --discriminant --source # Discriminant エラーの AST ノード種を推定
python3 scripts/inspect-errors.py --raw                  # フィルタ後の JSON 出力
```

フィルタは組み合わせ可能（例: `--category INDEXED_ACCESS --source`）。不足機能があれば同スクリプトに追加する。

## Architecture

See [README.md](README.md#ディレクトリ構成) for directory structure.

```
TS source → Parser (SWC AST)
  → ModuleGraph (import/export analysis)
  → TypeCollector + TypeConverter (build TypeRegistry)
  → TypeResolver (pre-compute expression types, expected types, narrowing)
  → Transformer (AST + type info → IR)
  → Generator (IR → Rust source code)
  → OutputWriter (file output, mod.rs generation)
```

## Shared Agent Docs

The Claude and Codex environments are intended to coexist.

- Shared guidance for both agents lives under `doc/agent/`
- Codex entrypoint is `AGENTS.md`
- Claude-specific rules remain under `.claude/`
- Codex-specific settings live under `.codex/` and `.agents/skills/`

## Core Principles

- **Ideal implementation**: Pursue the logically most ideal implementation regardless of cost. No compromises, no ad-hoc solutions. "Too much effort" and "good enough for now" are not valid justifications
- **KISS**: Minimal complexity for current requirements. When conflicting with "ideal implementation", prioritize the ideal
- **YAGNI**: Implement only what is needed now. No unrequested features or extensions
- **DRY + Orthogonality**: DRY eliminates duplication of *knowledge*, not *code appearance*. Keep duplication if sharing increases coupling

## Code Conventions

- `unwrap()` / `expect()` only in test code — see `.claude/rules/testing.md`
- `unsafe` prohibited (requires documented reason + user approval)
- `clone()` acceptable initially; leave TODO comment for unnecessary clones
- Public types/functions must have doc comments (`///`)

## Quality Standards

Maintain **0 errors, 0 warnings** for all changes. Run /quality-check upon work completion.

Coverage threshold ratchet: when measured coverage exceeds threshold by 2+ points, raise threshold by 1 point.

## Code of Conduct

- **Ideal implementation primacy** — 本プロジェクトの最上位目標は「理想的な TS→Rust トランスパイラの獲得」。ベンチ数値は defect 発見のシグナルであり最適化ターゲットではない。Structural fix > interim patch。詳細は `.claude/rules/ideal-implementation-primacy.md`
- **Uncertainty-driven investigation** — 不確定要素は一級市民として TODO に `[INV-N]` 形式で記録し、影響範囲が絞れるまで調査を尽くしてから実装に進む。`todo-prioritization.md` Step 0 参照
- **No unilateral conversion feasibility judgments** — "difficult in Rust" is never a valid reason to defer or deprioritize. Applies across all phases: TODO, plan.md, PRD. See `.claude/rules/conversion-feasibility.md`
- **Strict PRD completion criteria** — see `.claude/rules/prd-completion.md`
- **PRD design review**: PRD の設計セクション作成後、凝集度・責務分離・DRY の 3 観点で第三者目線のレビューを行う — see `.claude/rules/prd-design-review.md`
- **Incremental commits**: Commit at each phase completion for multi-phase work — see `.claude/rules/incremental-commit.md`
- **Pre-commit doc sync**: Update tasks.md / plan.md before commit messages — see `.claude/rules/pre-commit-doc-sync.md`
- **Bulk edit safety**: Script bulk replacements follow dry run → review → execute — see `.claude/rules/bulk-edit-safety.md`
- **Git operation restrictions**: Only the user performs `git commit` / `push` / `merge`. Claude only proposes commit messages
- **Questions with decision criteria**: Present options, pros/cons, and recommendations. No vague "Is this OK?" questions. Decide yourself when possible
- **Verification principle**: Define verification items and expected results before execution. No post-hoc judgments
- **Debugging**: Hypothesize root cause before next fix attempt. Never repeat the same fix twice
- **Deferred recording**: Record out-of-scope issues in `TODO` — see `.claude/rules/todo-entry-standards.md`
- **Document sync**: When changing code, update plan.md, README.md, CLAUDE.md, doc comments if they become inaccurate
- **Handoff documentation**: Document *why* decisions were made, not just what was decided
- **rust-analyzer**: Run `rust_analyzer_set_workspace` at work start. Reload after config changes. Do not ignore diagnostics errors

## Workflow

| Trigger | Skill |
|---------|-------|
| New feature or bug fix | /tdd |
| Work completion (before commit) | /quality-check |
| After feature addition | /refactoring-check |
| **PRD (backlog/ task) completion** | /backlog-management (TODO update → backlog deletion → plan.md cleanup → next PRD) |
| End of development session | /todo-audit |
| backlog/ operations | /backlog-management |
| Work request with empty backlog/ | /backlog-replenishment |
| PRD creation | /prd-template |
| Work request with empty TODO | /todo-replenishment |
| Investigation tasks | /investigation |
| TODO review (periodic / after major additions) | /todo-grooming |
| Conversion correctness audit | /correctness-audit |
| Hono conversion loop | /hono-cycle (single) or `/loop 0 /hono-cycle` (continuous) |
| Rule creation or modification | /rule-writing, /rule-maintenance |
| Large-scale refactoring (10+ sigs, 5+ files) | /large-scale-refactor |

## Proactive Improvement Principle

When discovering problems or inconsistencies, proactively investigate and fix before the user points them out:

- Do not dismiss warnings, errors, or inconsistencies as "temporary issues"
- Identify root causes before addressing problems
- Judge by "is it in the correct state?" not "is it working?"

## Skill Self-Improvement

Skills evolve with the environment. They are not static prompts.

### Observe

Record in `TODO` with `[skill-feedback:<skill-name>]` tag when:

- Skill instructions were ambiguous, causing hesitation
- Skill steps no longer match the codebase/environment
- User requested a direction change mid-skill (= insufficient instructions)
- You supplemented judgments not written in the skill

Include: what happened, why it's a problem, improvement proposal.

### Amend

When noticing improvement points during skill execution:

1. Explain the issue and its impact
2. Present a proposed diff
3. If approved, apply via `/rule-writing` + `/rule-maintenance`

### Passive Learning

When receiving behavioral correction from the user:

1. Generalize the instruction (pattern, not specific case)
2. Determine storage: project rules → `.claude/rules/`, project preferences → this `CLAUDE.md`
3. Present content and location for confirmation before writing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lapinChiro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
