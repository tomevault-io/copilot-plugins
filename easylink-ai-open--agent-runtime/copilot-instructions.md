## agent-runtime

> This package is the standalone agent runtime core. Treat independence from the

# agent-runtime — working notes

This package is the standalone agent runtime core. Treat independence from the
product application as an invariant, not a preference.

## The one hard rule: no product dependencies

The runtime must not import application packages from this repository. It may
own provider-neutral LLM types, provider HTTP clients, standard tool/prompt/
cache primitives, and other reusable runtime capabilities.

**Never** add an import of:
- any database / ORM (`sqlalchemy`, ...)
- any web framework (`fastapi`, `uvicorn`, ...)
- any product or application package
- any product-specific tool, product sandbox backend, store, or channel

Prefer standard-library implementations for built-in provider clients unless a
third-party package clearly earns its weight. If the loop needs application
state or product behavior, add a protocol and inject it.

## Mechanism, not policy

The kernel provides frameworks; the product fills in content.

- `collaboration.py` defines the `CollaborationMode` *structure* and the
  permission/injection *mechanism*. It defines no concrete modes, hard-codes no
  tool names, and carries no prompt copy. Blocked tool names and developer
  instructions are supplied by the product when it constructs a mode.
- `SystemPromptProvider` is a protocol; the real prompt (memory, skills,
  environment) is built in the product layer.
- `ToolDispatcher` exposes tools; the kernel owns none.

When tempted to "just hard-code the plan-mode tools" or "just build the system
prompt here," stop — that belongs in the product.

## No persistence coupling

`TurnResult` carries `messages` + metadata only. Do not add a `Session` field or
any storage concept. Persisting a turn is the product's responsibility.

## Module map

- `protocols.py` — the injection interfaces + no-op defaults. The kernel's
  boundary. `ModelClient` speaks neutral `LLMRequest`/`LLMResponse`.
- `tools.py` — standard `ToolRegistry` and `RegistryToolDispatcher`.
- `prompting.py` — standard `PromptParts` and `PromptProvider`.
- `cache.py` — standard prompt cache strategy.
- `messages.py` — neutral conversation model: `Message` + `TextPart` /
  `ImagePart` / `ToolCallPart` / `ToolResultPart`. The loop operates only on
  these. `ImagePart` (URL or base64) is mapped to provider image formats in the
  wire converters.
- `llm/` — LLM configuration, neutral types, wire converters, and built-in
  provider HTTP clients:
  - `llm/config.py` — `LLMConfig` (`api`, `model`, `base_url`, `api_key`, etc.).
  - `llm/types.py` — `LLMRequest` / `LLMResponse` / `LLMStreamEvent` /
    `StopReason`. `system` is top-level; tool args are dicts; usage normalized.
  - `llm/openai.py`, `llm/anthropic.py` — pure neutral↔provider dict converters.
    Converters depend on `llm/types.py`, never the reverse.
  - `llm/clients.py` — internal HTTP provider clients selected from `LLMConfig`.
- `config.py` — `AgentConfig` (model, max_steps, temperature, stream, retries,
  tool-error policy).
- `errors.py` — `AgentLoopInterrupted`, `WaitingForUserInput`, `ModelCallError`.
- `loop.py` — `AgentLoop`: pure orchestration over neutral types. Streaming,
  model retries, tool-error containment, HITL pause, graceful interruption.
- `core.py` — `Agent`: assembles the loop; public entry. `messages` is
  `list[Message]`.
- `factory.py` — `create_agent(...)`.
- `types.py` — `ToolSpec`, `TokenUsage`, `ToolCallback`.
- `events.py` — `TurnResult`, `CacheUsageInfo`, stream event-type constants.
- `budget.py` — `IterationBudget`.
- `hooks.py` — `AgentHooks` protocol + Noop/Composite.
- `collaboration.py` — collaboration-mode mechanism.
- `context/estimation.py` — pure token estimation (dict + neutral Message).
- `context/compaction.py` — `Compactor` protocol + `NoopCompactor` (default) and
  `SummarizingCompactor`. The compactor selects head/tail and summarizes the
  middle via the injected `ModelClient` (neutral types, any provider). Pure
  selection/tool-pair-safety logic; the summary text comes from the model.

## Provider neutrality is an invariant

The loop must never reference OpenAI/Anthropic dict keys (`tool_calls`,
`tool_call_id`, `function`, `content` blocks, `input_schema`, etc.). All such
shapes live ONLY in `llm/openai.py` / `llm/anthropic.py`. To support a new
provider, add a `llm/<name>.py` converter — do not special-case it in the loop.

## Testing

Tests must not hit the network, a DB, or product packages. Provider client tests should
monkeypatch the runtime HTTP boundary. The existing loop tests use
`FakeModelClient` / `FakeToolDispatcher`.

```bash
.venv/bin/python -m pytest tests/ -q
```

## Versioning discipline

Downstream products (samantha-agent, agent-cloud) pin this package with exact
versions (`agent-runtime==X.Y.Z`). That only works if every released state has
a version. The rules:

- Any change to the public surface — `Message` and content parts, `TurnResult`,
  the protocols in `protocols.py`, stream event constants, `AgentHooks`,
  `CollaborationMode`, `ToolEffect` — requires a version bump in
  `pyproject.toml` **in the same commit**, plus a matching `vX.Y.Z` git tag.
- Pre-1.0 semantics: bump **minor** for any breaking or additive API change,
  **patch** for bug fixes with no surface change.
- Never let downstream import code that isn't in a tagged version. If a product
  needs an in-flight runtime change, land and tag the runtime change first,
  then bump the product's pin.
- Push tags together with the commit: `git push origin main vX.Y.Z`.

## Relationship to products

The runtime is standalone. Product applications should construct `Agent` with
`LLMConfig` plus runtime primitives configured by product policy. Memory and
session behavior remain product concerns. Do not reach back into a product
package from this package to "save time" — that breaks the invariant.

---
> Source: [easylink-ai-open/agent-runtime](https://github.com/easylink-ai-open/agent-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
