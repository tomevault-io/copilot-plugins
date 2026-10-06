## kstack

> Guidance for coding agents working in this repo lives in

# AGENTS.md

Guidance for coding agents working in this repo lives in
**[`CLAUDE.md`](./CLAUDE.md)** — what kstack is, the `SKILL.md` rendering
pipeline, the three install modes, the response-envelope contract, how to run
each test tier, and the contribution workflow. Read that file.

This pointer exists because kstack renders its skills into nine agent CLIs
(`claude`, `codex`, `opencode`, `cursor`, `factory`, `slate`, `kiro`, `hermes`,
`pi`), and not all of them look for a file named `CLAUDE.md`.

One rule is repeated here rather than only linked, because it is project policy
rather than convention and a partial read shouldn't miss it:
**AI assistance must be disclosed.** [`CONTRIBUTING.md`](./CONTRIBUTING.md) does
not accept contributions authored entirely by an LLM, and asks for a trailer on
every commit an AI tool helped with:

    git commit -s --trailer "Assisted-by: AGENT_NAME:MODEL_VERSION" -m "<message>"

---
> Source: [kubetail-org/kstack](https://github.com/kubetail-org/kstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
