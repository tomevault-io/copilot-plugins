## kernel-agent-ascend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Mission of this repo (as of 2026-06-04): verify the authenticity of the Ascend kernel-agent corpus and toolchain.**
> 语料/工具使用真实性审计。This charter governs the whole `Kernel-Agent-Ascend/` repo — it auto-loads under
> `KernelWiki/`, `msprof-report-skill/`, and `docs/`. It sits *between* two existing CLAUDE.md files, which it does
> not replace:
> - **Schema / controlled vocabulary / page types** → `KernelWiki/CLAUDE.md` (the three-layer architecture, tags/aliases/schemas, confidence + reproducibility ladders).
> - **Two-stack project layout / NV→Ascend mapping / ralph flow** → top-level `/models/wangakang/kernel_agent/CLAUDE.md`.
> This file adds the layer they lack: **what we are verifying, how, and what a future agent is authorized to do about what it finds.**

## Why this mission exists

`validate.py` going green means frontmatter matches the schema and tags are in the controlled vocabulary. It does **NOT**
mean the corpus is true. The repo has already burned itself on this: 7 fabricated blogs (unreachable `hiascend.com/forum/thread-*`
URLs) were deleted on 2026-06-03, and the 2026-06-04 baseline audit found **17 of 19 `verified` pages resting on hollow
evidence** that `validate.py` passed without complaint. The gap between "validates" and "is true" is this mission's entire reason
to exist.

The core insight, stated once: **"test passes" ≠ "claim verified", and "validates" ≠ "real".** Every rule below is a way of
closing that gap.

## Posture: audit → gate (two phases, one set of checks)

1. **Phase 1 — audit the existing corpus.** Run the four axes below over everything already committed, produce an
   `data/authenticity-audit.md` verdict, and remediate per the policy. (Baseline pass done 2026-06-04 — see that file.)
2. **Phase 2 — freeze the same checks as a standing gate.** Anything *new* (page, bundle, PR, doc, blog) must pass the same
   axis checks before it is considered real. The gate is not a separate findings ledger — **the gate IS the verifier scripts
   exiting clean** (`validate.py`, `verify_core_prs.py --strict`, `verify_verbatim.py --strict`, `audit_authenticity.py`,
   `verify_bundle_numerics.py` on-NPU). The `.md` record proves they ran.

## The four authenticity axes

The mission names two halves — **语料 (corpus)** and **工具使用 (tool-usage)**. Corpus splits into three axes; tool-usage is the fourth.

### Axis 1 — PR source authenticity (`sources/prs/**`)

**Tiered, forge-routed, checks merged-state — not just existence.**

- **Offline gate (runs anywhere):** flag seed-pattern pages (`PR-000X` with non-kernel `changed_paths` like `azureml/`),
  any `merge_sha: unknown`, and any `status: merged` not yet substantiated against upstream.
- **Online audit + new-PR gate:** resolve every PR against **its own forge** — `gitee.com` → Gitee v5 REST
  (`scripts/forge_resolve.py`), `github.com` → `gh`. Confirm the PR exists, title/author match the page, and **the recorded
  `status` matches upstream** (`merged_at` / `state`). Backfill the real anchor SHA.
- **Forge SHA semantics differ — this matters:** GitHub exposes `merge_commit_sha`; **Gitee never populates it** (null even
  for merged PRs), so the only anchor is `head.sha`. A page reading `merge_sha: unknown` is not "missing data" — for a Gitee PR
  the right field is `head.sha`, recorded with `merge_sha_kind: head.sha`.
- **The Ascend corpus deliberately includes closed-not-merged PRs** as stock-operator analogues. That is legitimate — so the
  check is **"does the page tell the truth about upstream state"**, NOT "must be merged". The defect is a page claiming `merged`
  when upstream closed it, which the old GitHub-only `--strict` could never catch (it 404'd on every Gitee PR).

### Axis 2 — on-hardware bundle authenticity (`artifacts/kernels/<slug>/variants/`)

**Tiered: static forensics always; on-hardware re-run for audit + required for new bundles.** Bundles are the only evidence that
lifts a page to `verified`, so they are the highest-stakes artifacts.

- **Static forensics (always-on gate):** every `files[*].sha256` in `PROVENANCE.yaml` matches the file on disk; `bench.md`
  headline numbers trace to the `op_summary*.csv` cells; column names are real CANN-8.3 msprof columns; no physically-impossible
  values. Static forensics catches tampering and internal inconsistency but **cannot** catch a self-consistent hand-authored fake.
- **On-hardware spot re-run (audit + new bundles):** only re-running on real 910C silicon distinguishes a genuine capture from a
  self-consistent fabrication. Run `scripts/verify_bundle_numerics.py` on the NPU (it asserts each op computes the right answer
  against an independent reference). For a *new* bundle the gate **requires** this at creation time.
- **All 13 current bundles are `asset_mode: derived`** (a harness we wrote + a real capture), not `verbatim`-upstream like the NV
  stack. That is acceptable — see the synthesis stance below — provided the capture is real and disclosed.

### Axis 3 — doc / blog authenticity (`sources/docs/**`, `sources/blogs/**`)

**Both checks, severity-split.** Two genuinely different failure modes:

- **HIGH — placeholder cited as evidence (offline gate).** `grep` every source for `PLACEHOLDER` / "when filled in" / TODO,
  then cross-reference: is that source cited in a `verified` page's `evidence_basis`, a bundle `PROVENANCE` `derived_from`, or a
  `version-claims` `evidence_source_id`? A placeholder stub standing in as a verified page's official-doc leg **fakes the
  confidence gradient** — this is the worst failure mode and the most common one found in the baseline.
- **MEDIUM — unreachable / forum-style URL (online audit).** The deleted-fake-blog smell. Check `url:` resolves (HTTP 200).
- **LOW — reachable but stub, not cited as evidence.** Honest scaffolding; leave it, note it.

### Axis 4 — tool-usage authenticity (the verifier scripts themselves)

**A verifier that passes without actually checking anything is its own fabrication.** For each verification script ask:

- **Can it pass while skipping its check (inert green)?** The canonical example: `verify_core_prs.py --strict` called
  `gh`/GitHub for Gitee repos → 404 on all 8 → green-but-blind. It never anchored a single SHA, yet docs cited "9 PRs 字节一致"
  as upstream-verified. (Fixed 2026-06-04 to be forge-aware.)
- **Does it test logic or real evidence?** The 47 + 94 pytest are real and pass — but they exercise **parser logic on synthetic
  in-memory trees**, they do NOT prove any `.prof` capture is real. "94 passed" is a true statement about the parser, not about
  the corpus. Keep that distinction explicit in any claim.
- **Is the documented invocation even correct?** HANDOFF prescribes `vllm_ds` for capture (doesn't exist) and warns off
  `llm_test` (which actually imports `torch_npu 2.7.1` fine). Verify env claims by running them, don't trust the prose.

## Verdict taxonomy (use these words in findings)

- **real** — exists, reachable, and the page's claims about it match upstream / measurement.
- **mislabeled** — the artifact is real but the page states something false about it (e.g. `status: merged` on a closed PR; a
  `verified` page whose cited leg is hollow). Fix the label, never delete the artifact.
- **placeholder-as-evidence** — a disclosed stub doing duty as a verified page's evidence leg. HIGH severity.
- **fabricated** — does not exist upstream / unreachable / invented. The only verdict that warrants deletion (with human sign-off).
- **inert-verifier** — a check that passes without verifying. Tool-axis equivalent of fabrication.

## Remediation policy — what an agent is authorized to do about a finding

**Tiered by reversibility, executed through the NV mechanism. Mirror the NVIDIA stack
(`/models/wangakang/kernel_agent/Kernel_Agent_NVIDIA/`) — it is the worked example of provable PR authenticity
(verbatim `key-files` byte-matched by `verify_verbatim.py` + resolved SHAs).**

**Auto-fix (mechanical, evidence-backed, and ALWAYS logged):**
- Backfill a real anchor SHA resolved from the correct forge API (`scripts/backfill_pr_status.py`).
- Flip a provably-false `status:` to match upstream (`merged`→`closed`/`open`, or `open`→`merged` when upstream merged it).
- Downgrade a `verified` page to `source-reported` when **no** surviving evidence leg holds (no bundle + placeholder doc +
  closed PR). Annotate the page body with a dated audit note.
- On a `verified` page that retains a real on-hardware bundle, keep `verified` but **annotate** the weaker cited legs for
  transparency rather than hiding them.

**NEVER auto-do (report-only + ask a human first):**
- **Delete any corpus.** A real-but-mislabeled artifact gets its label fixed, never deleted. (PR-455 is a real Gitee PR
  mislabeled `merged` — the fix is to correct the status, not to remove the page.) Deletion repeats the repo's own self-harm.
- **Fabricate replacement evidence.** Do not invent a capture, a SHA, a PR, or a doc to "close" a gap. A missing bundle stays
  missing and is logged as a gap. This is the repo's oldest red line: 不可凭空造 PR/blog/URL.

## Synthesis stance — when "honest synthesis with disclosure" passes

**Tiered by what the asset backs. Disclosure is necessary everywhere; it is sufficient only where no truth-claim rides on it.**

- **Test fixtures (exercise parser logic, not a truth claim): synthesis + disclosure → PASS.** The `prof_cann8` fixture
  re-expresses real captured values under canonical column names and adds plausible cache-hit columns — legitimate, because the
  test checks that the parser resolves column *names*, not the numbers' provenance. It is disclosed in its README.
- **A `verified` page's evidence leg: must be directly-measured (a real `op_summary`) or verbatim-upstream.** Synthesis here
  fakes the confidence gradient and is FLAGGED — same principle as the placeholder-doc case (fine as scaffolding, not as evidence).
- **Undisclosed synthesis anywhere → HIGH severity**, regardless of what it backs.

## Output convention — the audit record

Mirror the NV precedent `Kernel_Agent_NVIDIA/KernelWiki/data/phase3-verify-verbatim-audit.md`. Each audit run appends/emits a
markdown record at **`KernelWiki/data/authenticity-audit.md`** with: **Captured** (date — `Date.now` is unavailable in this env,
stamp from the run), **Command(s)**, **Exit**, **Environment** (gh version + account, CANN, torch_npu env, NPU), **Scope**
(counts per axis), **stdout**, **Findings** (verdict taxonomy + severity + target + action_taken), **Reproducibility** (exact
steps). The record is human-readable and re-runnable; it is not the gate — the gate is the verifiers exiting clean.

## The gate — run these; they must come back clean for new work

```bash
cd /models/wangakang/kernel_agent/Kernel-Agent-Ascend/KernelWiki
python3 scripts/validate.py                       # schema + tags + AC-1..AC-11
python3 scripts/generate-indices.py               # after any frontmatter change
python3 scripts/audit_authenticity.py --closed <closed-pr-ids>   # exit 1 if any CRITICAL verified page
python3 scripts/verify_core_prs.py --strict       # forge-aware: status+anchor vs upstream (exit 2 = env/rate-limit, inconclusive)
# on a 910C box, for bundle truth (not just schema):
source /usr/local/Ascend/ascend-toolkit/set_env.sh
/root/miniconda3/envs/llm_test/bin/python scripts/verify_bundle_numerics.py   # 9/9 PASS on real silicon
```

**Env reality (verified 2026-06-04, overrides stale HANDOFF prose):** capture/numerics env is **`llm_test`**
(`torch_npu 2.7.1.post2`, imports fine) — NOT `vllm_ds` (does not exist). Gitee anonymous API is 60 req/h; a sweep exhausts it
and returns HTTP 403 → treated as ENV/exit-2 (inconclusive), never a content failure. Set `GITEE_TOKEN` to lift to 5000/h.

## Do-not list (authenticity-specific; general repo rules live in the other two CLAUDE.md files)

- Do not treat `validate.py` green as "authentic" — it checks schema shape, never URL reachability, never whether a doc is a
  placeholder, never whether a PR actually merged, never whether a bundle is a real capture.
- Do not call `gh` for a Gitee repo — `gh` only speaks GitHub. Route by the `url:` host (`scripts/forge_resolve.py`).
- Do not record `merge_sha: unknown` for a Gitee PR — the anchor is `head.sha` (Gitee has no `merge_commit_sha`).
- Do not promote a page to `verified` on a placeholder doc + a closed PR. `verified` needs a real official-doc leg AND
  (a merged upstream PR OR a real on-hardware bundle).
- Do not delete a mislabeled-but-real artifact, and do not invent evidence to close a gap. Log the gap.

---
> Source: [Ryannnice/Kernel-Agent-Ascend](https://github.com/Ryannnice/Kernel-Agent-Ascend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
