## council-ex

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

CouncilEx — a library (not an app) for multi-model LLM council workflows: run LLMs in parallel rounds, aggregate/judge outputs, stream tokens, call tools, observe via telemetry + PubSub.

## Hard rules

### NEVER publish to Hex without explicit instruction

Running `mix hex.publish` (or any command that actually pushes the package to hex.pm) on this repo requires the user's explicit, in-the-moment authorization for that specific action.

- `mix hex.publish --dry-run` is allowed without asking.
- All prep work (ex_doc setup, mix.exs metadata polish, README updates, license file, package description tuning) is allowed without asking.
- Hex publishes are irreversible — once a version lands on hex.pm it can only be retired, not removed.
- A user saying "prep for Hex" does NOT grant publish authority.
- A user saying "publish 0.X.Y now" or equivalent specific instruction does.
- When in doubt, run `--dry-run` and ask.

## Commands

```bash
mix deps.get                       # fetch deps
mix compile --warnings-as-errors   # CI compiles this way — warnings fail the build
mix format --check-formatted       # CI gate
mix credo --strict                 # CI gate
mix dialyzer                       # CI gate; PLT cached in priv/plts (mix.exs pins the path)
mix test                           # run suite
mix test test/council_ex/foo_test.exs:42   # single test by file:line
mix docs                           # build ex_doc; must build with zero warnings
mix council.diagram MyCouncil      # print Mermaid/ASCII diagram of a council module
mix run examples/<name>.exs        # run an example (needs a real provider API key — see below)
```

Match all five CI gates (compile-as-errors, format, credo --strict, dialyzer, test) before claiming work is done — that is exactly what `.github/workflows/ci.yml` runs.

## Architecture

**Library, not a server.** `CouncilEx.Application` starts a minimal tree: one `Registry` (`CouncilEx.Runner.Registry`) for run lookup. Council runs are **caller-owned, unsupervised pids by default** (GenServer.start semantics) — the library does not bundle a supervisor for them. `CouncilEx.Supervisor` is an opt-in `DynamicSupervisor` wrapper for callers who want tenant isolation / bulk-terminate.

**One execution path, two council forms.** `CouncilEx.run/3` (sync), `start/3` / `start_link/3` (async, return `{:ok, pid}`) all dispatch polymorphically over either a **module-form council** (`use CouncilEx` DSL — `member`/`round`/`chair` macros, compiles to `Mod.__council__/0` → `%Spec{}`) or a **`%DynamicCouncil{}` data form** (pipeable builder, JSON ser/de, registry-by-name). Identical semantics. `CouncilEx.validate/1` gates config before any process spawns.

**Run lifecycle.** `CouncilEx.RunServer` is the GenServer driving a single run start→finish. Round execution + state live in `lib/council_ex/runner/` (`round_exec.ex`, `run_state.ex`, `recorder_process.ex`).

**Rounds & topologies.** A round implements the `CouncilEx.Round` behaviour. Built-in round atoms (`:independent_analysis`, `:peer_review`, `:vote`, `:pairwise_elimination`, …) map to modules via `@builtin_rounds` in `lib/council_ex.ex`. Prebuilt multi-round topologies live in `lib/council_ex/councils/` (`Specialist`, `Tournament`, `WeightedConsensus`, `JuryWithRetry`, …).

**Providers.** Pluggable `CouncilEx.Provider.Adapter` behaviour; stock adapters in `lib/council_ex/provider/adapters/` (OpenAI/Anthropic/Gemini/Ollama/OpenRouter). `CouncilEx.Providers.Instructor` is the engine for structured output (Ecto schema cast + tool-shaped fallback), the bounded tool-call loop (parallel exec), and streaming — `stream/3` drives the same loop across SSE round-trips. `CouncilEx.Providers.Mock` is tests/fixtures only, **never production**.

**Capability layers** (each independent per member): structured output, streaming, tools, per-member `:confidence`. Plus cross-cutting subsystems — aggregators (`lib/council_ex/aggregators/`), `Reliability` store, `BiasDetector` (diagnostic), `Recorder`, `Registry`, and the `AutoCouncil` routing layer (a council that picks itself via pluggable strategies).

**Observability.** 10 frozen PubSub events on topic `"council_ex:run:#{run_id}"`, documented in `CouncilEx.Events` — treat this surface as a contract. Telemetry `[:council_ex, :run|:round|:member|:tool, :*]`, a verbose stdout tracer (`verbose: true | :debug`), and diagram tooling (`CouncilEx.Diagram.{to_ir,topology,sequence}`).

## Conventions & gotchas

- **Optional deps are genuinely optional.** `ecto_sql`/`postgrex`, `oban`, `redix`, `phoenix_pubsub` are `optional: true`; the library compiles fine without them and the backend modules are simply omitted. Don't write core code that assumes any of them is loaded.
- **Deployment shape via `:mode` config.** `CouncilEx.Config` `:mode` (`:single_node` default / `:multi_node`) flips Registry / Reliability / Recorder backends in one place; per-key overrides win over the mode default. See `docs/PERSISTENCE.md`.
- **`test/support/` is compiled in `:test` env only** (`elixirc_paths`). Test helpers: `import CouncilEx.Test` → `script_council/2`, `capture_events/2`, `assert_round_completed/3`. Mimic + Bypass back the test stack.
- **Examples use real provider keys only** — no Mock fallback. They need `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` / `GEMINI_API_KEY` / `OPENROUTER_API_KEY` set.
- **Docs must build clean.** `mix docs` should emit zero warnings; `mix.exs` `docs/0` carries `skip_code_autolink_to` for intentionally-hidden refs, and callback references in `@doc`/extras use the `c:Mod.fun/arity` form.
- Extensive per-topic docs live in `docs/` (COUNCILS, COMPOSITION, PERSISTENCE, AUTO_COUNCILS, PER_MEMBER_CAPABILITIES, OBSERVABILITY, …) and are wired into ex_doc `extras`.

---
> Source: [brewingelixir/council_ex](https://github.com/brewingelixir/council_ex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
