## stt-mcp

> Before installing dependencies, changing CUDA/model configuration, registering an MCP client, or

# Agent Instructions

## Mandatory setup and acceptance guide

Before installing dependencies, changing CUDA/model configuration, registering an MCP client, or
claiming this project works on a machine, read and follow [`docs/SETUP.md`](docs/SETUP.md) from top
to bottom.

That runbook is the authoritative agent procedure for:

- hardware inspection, AI recommendation, user confirmation, and persisted backend selection;
- Parakeet CPU/Metal and Granite CUDA setup on each documented platform;
- NVIDIA driver, CUDA 12.8, VRAM, FFmpeg, Python 3.13, and `uv` checks when applicable;
- downloading exact pinned Parakeet or Granite assets, verifying checksums, and reviewing trust and
  license boundaries;
- running static gates, CLI transcription, and the opt-in real FastMCP acceptance test;
- registering, unregistering, and cleaning up MCP clients safely.

The setup agent must show the hardware recommendation, ask the user to confirm or choose a
supported alternative, and persist the concrete choice. Do not silently choose a backend. Do not
auto-download Parakeet assets at runtime, relax exact revisions or checksums, or claim an untested
platform works.

Granite must remain fail-closed. Do not remap `CUDA_VISIBLE_DEVICES`; exactly one NVIDIA GPU must be
visible and the conservative 8 GiB globally free startup floor must pass. This floor is not a claim
that inference intrinsically consumes 8 GiB. Parakeet is an additional backend, not a relaxation of
Granite's CUDA contract.

Use `uv` exclusively for Python environments, dependencies, commands, and builds. Keep stdout
clean when running the stdio server because stdout is the MCP protocol wire.

## Completion criteria

An environment/backend pair is not verified until all applicable commands in `docs/SETUP.md` pass,
including:

1. Hardware inspection, user-confirmed selection, and FFmpeg probes.
2. Exact selected runtime/model revision and checksum verification.
3. Ruff, BasedPyright, `ty`, Mypy, pytest, and package build.
4. A real local speech file transcribed through
   `tests/test_mcp_acceptance.py` with both SRT and VTT artifacts published.
5. Zero surviving `parakeet-cli` or `stt_mcp.worker` processes after acceptance exits.

---
> Source: [sm18lr88/STT-MCP](https://github.com/sm18lr88/STT-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
