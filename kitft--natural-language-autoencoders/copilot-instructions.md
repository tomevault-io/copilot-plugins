## natural-language-autoencoders

> - **This is an open-source repo.** Only standard libs: `pathlib.Path`,

# NLA — instructions for Claude / AI assistants

## Constraints

- **This is an open-source repo.** Only standard libs: `pathlib.Path`,
  `pyarrow`, `transformers`, `datasets`, `httpx`, `pyyaml`, `numpy`, `orjson`,
  `safetensors`, the public `anthropic` SDK, and whatever Miles/SGLang pull in.
  No private/internal dependencies.
- **Miles is upstream, not ours.** Don't edit files under `miles/` — extend via
  subclassing (`NLAFSDPActor`) and the `--*-path` function-pointer args. The
  two upstream patches we depend on (`--custom-actor-cls-path`,
  `--force-use-critic`) are documented in `docs/design.md` §2.
- Miles uses argparse; match that for CLIs in `nla/`.
- Storage and completion-provider backends are pluggable via import-path
  strings (`--storage-cls`, `--provider-cls`). The shipped implementations are
  `LocalStorage` and `AnthropicProvider`. Cloud storage / other LLM APIs are
  bring-your-own — don't hardcode bucket paths or vendor SDKs into `nla/`.

## Key invariants (do not break these)

- **Data-gen NEVER normalizes** — all parquets store raw vectors
  (`norm="none"`). `stage3_build` asserts input `norm == "none"`. Normalization
  happens at injection time (`injection_scale`) and at loss time (`mse_scale`),
  both read from the sidecar.
- **Stage-1 split is DOCUMENT-level** — partition by unique `doc_id`, all rows
  from the same doc go to the same bucket. Never split positions from one doc
  across `av_sft` / `ar_sft` / `rl`.
- **Stage-0 `_MIN_POSITION = 50`** — need enough left-context for the
  activation to be meaningful. Earlier positions decode to noise.
- **Critic extraction is suffix-anchored** — no scan, no marker token. The
  critic prompt template ends with `... <summary>`; training extracts at
  `tokens[-1]`. `critic_suffix_ids` in the sidecar is for sanity-checking only.
- **Per-doc keyed RNG** — same `(seed, doc_id)` → same sampled positions
  regardless of chunk boundaries, slice ordering, or process count. This is
  what makes multi-GPU stage-0 sharding bit-reproducible.
- **Injection hook scans for the token ID inside the hook** (`inputs[0]`), not
  from precomputed positions. Miles reorders samples twice before the forward
  pass; any precomputed index is wrong by construction.
- **`cp_size == 1` only.** Context-parallel splits each sample across ranks
  and breaks the neighbor check. NLA sequences are short; CP buys nothing.
- **Sidecar is the contract.** Token IDs, prompt templates, `injection_scale`,
  `mse_scale`, `d_model` — all loaded from `nla_meta.yaml` and asserted
  against the live tokenizer at startup. Never hardcode them.

## Debugging

If injection silently fails the actor sees the literal CJK marker char and
free-associates Chinese. Grep generated text for CJK — that's the loudest
smoke test for the entire injection path. See `docs/inference.md`
§ "Debugging: injection-failure smell" for the cause checklist.

---
> Source: [kitft/natural_language_autoencoders](https://github.com/kitft/natural_language_autoencoders) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
