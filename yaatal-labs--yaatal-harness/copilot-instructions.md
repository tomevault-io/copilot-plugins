## yaatal-harness

> Yaatal-Harness is the **AI control plane** of Yaatal — runtime custody, audit, behavioral policy,

# CLAUDE.md

Yaatal-Harness is the **AI control plane** of Yaatal — runtime custody, audit, behavioral policy,
and the self-improvement loop. Read `README.md` and `ARCHITECTURE.md` first; design docs live in
`docs/` (`CONTROL-LOOP.md`, `CLI-FIRST-TOOLS.md`, `POLICY-DISTRIBUTION.md`).

Gate before any PR: `cargo check --workspace --all-targets` (the workspace must keep compiling;
`crates/yaatal-api` is a stub excluded from the workspace — leave it out).

## Development policy — Ponytail (lazy senior dev mode)

Agent-written code in this repo follows the [Ponytail](https://github.com/DietrichGebert/ponytail)
(MIT) efficiency ladder. Before writing code, stop at the first rung that holds:

1. Does this need to be built at all? (YAGNI) → skip it
2. Does it already exist in this codebase? → reuse the helper/util/pattern
3. Does the standard library do it? → use it
4. Does a native platform feature cover it? → use it
5. Does an already-installed dependency solve it? → use it
6. Can it be one line? → make it one line
7. Only then: write the minimum working code

The ladder runs **after** you understand the problem, not instead of it — read fully, trace the
real flow end-to-end, then climb. Fix root causes once, not symptoms per caller. No abstractions
that weren't requested; no new dependency if avoidable; deletion over addition; boring over
clever. Mark intentional simplifications with `ponytail:` comments naming the known ceiling and
the upgrade path.

Never lazy about: understanding the problem, input validation at trust boundaries, error handling
that prevents data loss, security, explicit requirements — and non-trivial logic leaves one
runnable check behind.

Full skills (`/ponytail-review`, `/ponytail-audit`, `/ponytail-debt`):
`/plugin marketplace add DietrichGebert/ponytail` → `/plugin install ponytail@ponytail`.

---
> Source: [Yaatal-labs/Yaatal-Harness](https://github.com/Yaatal-labs/Yaatal-Harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
