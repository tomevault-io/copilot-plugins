## reconops

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

This repo currently contains **design documents only** — no Go code, no `go.mod`, no Makefile yet. The implementation has not started. All build/test/lint commands described in `DOCS/Prompts/PROJECT.md` (§10–§11) are forward-looking targets, not commands you can run today.

When asked to implement, scaffold against the layout in `PROJECT.md` §11 (`cmd/hub`, `cmd/agent`, `internal/proto`, `internal/hub/{api,web,store,runner,investigator,llm,auth}`, `internal/agent/{conn,collect,collectors,exec}`).

## Source of truth

Two long-form Russian design docs drive everything:

- `DOCS/Prompts/PROJECT.md` — system design (architecture, proto sketch, SQLite schema, collector contract, security model, MVP roadmap).
- `DOCS/Prompts/BASE_TASKS.md` — prompt engineering for the LLM investigator (system prompt, tool schemas, API params, edge cases).

Read both before proposing changes to architecture, the agent↔hub protocol, the data model, or the investigator loop. They are the spec.

## Architecture invariants — do not violate

These are not stylistic preferences; they are load-bearing properties of the system.

1. **Read-only by protocol.** The gRPC `Hub` service has only `Enroll` and `Connect`; `HubMsg` carries only `CollectRequest` / `CancelRequest` / `ConfigUpdate`. Never add a verb that mutates target-host state. Read-only is enforced in five layers (PROJECT.md §3.4): protocol, compiled-in collector catalog, exec gateway whitelist, OS-level sudoers, CI lint banning write syscalls in `collectors/`. Any new collector must respect all five.

2. **One tool_use per turn.** The investigator's contract with Claude requires exactly one `tool_use` block per assistant turn (`tool_choice: {"type":"any"}`, see BASE_TASKS.md §2–§3). Hub additionally enforces this by discarding extra blocks. Do not relax this — the step-by-step UX depends on it.

3. **Evidence-first findings.** `add_finding` schema requires `evidence_refs` with `minItems: 1`. Don't make it optional.

4. **Operator directives are MUST, not hints.** `OPERATOR HYPOTHESIS [priority: HIGH]` discards Claude's pending proposal and forces re-plan. `IGNORED` finding permanently closes a branch. These are encoded in the system prompt at MUST level.

5. **Three-tier context.** Claude sees compact summaries in messages (~500–2000 tokens per result); full structured data only via `get_full_result(task_id)`; raw artifacts only via `search_artifact`. Never load full artifacts into the LLM context.

6. **Compiled-in collectors.** No dynamic plugin loading. New collector = new agent release.

## LLM defaults

The LLM client (`internal/hub/llm/`) speaks **OpenAI-compatible** chat/completions with function-calling tools. Default backend is **OpenRouter**; any compatible endpoint works (vLLM, LiteLLM, raw OpenAI). All settings external — never hardcode.

- `model`: from `RECON_LLM_MODEL` env or `hub.yaml.llm.model`. Compile default: `anthropic/claude-sonnet-4.5`.
- `base_url`: `RECON_LLM_BASE_URL` / `llm.base_url`. Default `https://openrouter.ai/api/v1`.
- `api_key`: `RECON_LLM_API_KEY` (envname overridable via `llm.api_key_env`). Required at runtime; if missing, hub starts but investigator endpoints return 503.
- `max_tokens: 4096`, `temperature: 0`.
- `tool_choice: "required"` (OpenAI semantics — closest equivalent of Anthropic's `{"type":"any"}`).
- **Extended thinking is NOT used** — not portable across backends. Documented deviation from BASE_TASKS.md §2 (which targets native Anthropic Messages API).
- Compaction triggers near 150K tokens (PROJECT.md §7.4) — Week 4.

## Stack constraints

- Go 1.22+, single static binary per side (hub, agent).
- SQLite via `modernc.org/sqlite` (pure Go, **no CGO**).
- Web UI: stdlib `net/http` + `html/template` + HTMX + Alpine.js + Tailwind from CDN. Avoid build pipelines for the frontend in MVP.
- gRPC with `protoc-gen-go`, mTLS mandatory on agent↔hub.
- Structured logs via `slog`.

## Working language

Design docs and operator-facing UI strings are in Russian. Code, identifiers, comments, and commit messages are in English (per global CLAUDE.md). When quoting from PROJECT.md / BASE_TASKS.md in discussion, keep the Russian original — translation drift loses nuance.

---
> Source: [vasyakrg/reconops](https://github.com/vasyakrg/reconops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
