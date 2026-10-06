## qwen3-tts

> Read `ENGINEERING.md` before changing the repository. It is the single normative source

# Agents — repository entrypoint

Read `ENGINEERING.md` before changing the repository. It is the single normative source
for planning, evidence, backend claims, benchmark lifecycle, privacy, staging and
commits; do not duplicate or override those rules here.

`PLAN.md` is the concise current task list. Load only the referenced reviewed
`.work/*.md` addendum when a task needs detail. Raw benchmark/profiler material belongs
in the private evidence areas described by `ENGINEERING.md`.

Build and gates: `make blas`; `./qwen_tts --caps`, `--self-test`, `--dispatch-map`, and
`make test-golden` as applicable to the change.

---
> Source: [gabriele-mastrapasqua/qwen3-tts](https://github.com/gabriele-mastrapasqua/qwen3-tts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
