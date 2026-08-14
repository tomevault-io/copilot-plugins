## ja-en-direct-s2st-benchmark

> - Implement the experiment pipeline in the order documented in `DESIGN.md`.

# Repository instructions

- Implement the experiment pipeline in the order documented in `DESIGN.md`.
- Never modify the external corpus repository or write derived artifacts below `CORPUS_ROOT`.
- Keep model IDs and immutable revisions together. Reject configurations that omit a revision.
- Default to the `smoke` profile. Never start `full` training implicitly.
- Keep long-running operations resumable and use atomic writes for manifests and state.
- Do not overwrite an existing artifact unless the caller explicitly passes `--overwrite`.
- Use only the train split for training and vocoder fitting.
- Keep GPU tests behind the `gpu` pytest marker.
- Do not commit audio, features, checkpoints, model caches, predictions, or large evaluation output.

---
> Source: [Yaaamashiro/ja-en-direct-s2st-benchmark](https://github.com/Yaaamashiro/ja-en-direct-s2st-benchmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
