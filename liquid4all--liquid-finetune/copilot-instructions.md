## liquid-finetune

> - Do not use `/tmp` for large materializations such as model downloads, HF caches, tokenized datasets, checkpoint exports, vLLM assets, or package caches.

# Repo Notes

## Storage

- Do not use `/tmp` for large materializations such as model downloads, HF caches, tokenized datasets, checkpoint exports, vLLM assets, or package caches.
- Prefer repo-local caches for small/reusable artifacts and persistent shared storage such as `/lambdafs` for large model/checkpoint artifacts.
- `/tmp` is acceptable only when the tool requires node-local scratch space or short socket paths, such as Ray runtime/session directories.

---
> Source: [Liquid4All/liquid-finetune](https://github.com/Liquid4All/liquid-finetune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
