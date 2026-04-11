## fuffa

> This file is referenced from [AGENT.md](AGENT.md). Use it when working on **Fuffa** with Gemini.

# Gemini – Fuffa project instructions

This file is referenced from [AGENT.md](AGENT.md). Use it when working on **Fuffa** with Gemini.

## Repo summary

- **Fuffa**: Go + eBPF/XDP experiment.
- **Key paths**: `main.go`, `pkg/xdp-ebpf/`, `kernel_ebpf/`.
- **Commands**: `make`, `make test`, `make test-in-docker`.

## Conventions for Gemini

- Prefer standard Go style and the existing patterns in `pkg/xdp-ebpf/`.
- When changing eBPF or XDP behavior, keep C in `kernel_ebpf/` and Go in `pkg/xdp-ebpf/` and `main.go`.
- Run tests with `make test` (or `make test-in-docker`) after code changes.
- For release/notes workflow, follow the scripts under [.github/](.github/).

For full project context and links to other agent docs, see [AGENT.md](AGENT.md).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gsantomaggio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Gsantomaggio)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
