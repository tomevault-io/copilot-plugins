## lucarne

> Provider-specific responsibility must stay inside provider-owned modules,

# Agent Instructions

## Provider Responsibility Boundary

Provider-specific responsibility must stay inside provider-owned modules,
descriptors, traits, or associated constants. Do not move provider ids, cursor
prefixes, transcript parsing rules, discovery details, file format quirks, or
resume semantics into public/common layers as ad hoc branching or literals.

Common layers may orchestrate through typed provider contracts only. If a change
needs provider-specific behavior, add it to the provider contract and implement
it at the provider boundary instead of teaching the common layer about that
provider.

## Development and Testing

During development and testing, use Rust nightly with the new build directory layout for Cargo commands.

Required pattern:

```bash
cargo +nightly <command> -Zbuild-dir-new-layout
```

Examples:

```bash
cargo +nightly check -Zbuild-dir-new-layout
cargo +nightly test -Zbuild-dir-new-layout
cargo +nightly clippy -Zbuild-dir-new-layout --all-targets --all-features
```

## Provider Boundaries

Provider responsibility must not be moved into public/common/core layers.

- Public/common/core layers may route opaque provider ids, but must not own provider id catalogs, provider-specific cursor formats, parsing rules, discovery rules, resume semantics, media visibility rules, or compatibility fallbacks.
- Provider-specific behavior belongs in the provider module, provider descriptor, or provider trait implementation.
- `agent-sessions` is the provider layer. It owns parse, discovery/find, watch, and provider descriptors. It must not own history orchestration, history cursors, history pagination, or history indexes.
- `lucarne::history` is the history application layer. It may consume `agent-sessions` provider descriptors and generic provider capabilities, but it must not mention or special-case concrete providers such as Codex, Claude, Gemini, Copilot, or Pi.
- Watch integration must be descriptor/trait driven. Do not add a public/common provider enum or a common watch match table when adding a provider; extend the provider descriptor/trait and implement the behavior inside that provider.
- Runtime/core provider ids are opaque provider contracts. Public/common layers must not keep a concrete provider-id catalog or deserialize by provider-name whitelist.
- Only provider identity is a static provider contract. Dynamic runtime metadata such as labels, display names, and binary paths must stay owned data (`String`/`SmolStr`), never `Box::leak`-promoted `&'static str`.
- History code should be split by responsibility (`provider`, `transcript`, indexing/orchestration) instead of accumulating unrelated concerns in one file.
- Hot paths must use bounded reads/windows. They must not scan or read whole session files to serve history, watch, metadata, transcript, or title requests.
- Starting history watch requires at least one core event subscriber. Do not parse provider history updates into a broadcast channel with zero receivers.
- This refactor does not require old compatibility. Prefer rejecting obsolete cursor/API formats or returning absence over introducing fallback work in shared layers.
- `agent-sessions` parsed provider structs should expose only data consumed by descriptor, parse, discovery, or watch contracts. Do not keep unused convenience getters, raw provider fields, or generic project aggregation APIs as speculative compatibility.
- Do not reintroduce a generic raw parser bridge such as `ProviderParser -> ProviderParsed<Body>`. Provider descriptors must enter through semantic byte parsing, discovery metadata parsing, or watch parsing; provider-private helpers may exist only as implementation details below that boundary.
- Provider and watch helpers must compile in `--no-default-features` single-provider matrices. Do not rely on `--all-features` to hide unused imports, dead helpers, or tests that only compile when unrelated providers are enabled.

---
> Source: [tuchg/Lucarne](https://github.com/tuchg/Lucarne) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
