## sm12x-llm-bench

> Public repo: `SM12X-SOCOM/SM12X-LLM-BENCH`.

# AGENTS.md — SM12X-LLM-BENCH

Public repo: `SM12X-SOCOM/SM12X-LLM-BENCH`.

## What this is

A standalone click-run benchmark client for SM12X Blackwell (GeForce RTX 50,
RTX PRO 6000, DGX Spark). **Not** a `hard-subset` patch to `r0b0tlab/r0b0bench`.
Do not modify r0b0bench unless the user names that repo.

## Prior art (read-only)

- `r0b0bench` — read as prior art for OpenAI-compatible lane/report shape;
  re-fetch before citing a SHA.
- `local-inference-lab/llm-inference-bench` — pin recorded in `docs/PLAN.md`
  and credited in `third_party/NOTICE.md`.

Do not vendor `llm_decode_bench.py` (16k-line monolith). Copy only small
cited functions (stride select, Wilson, exact McNemar, MC letter extract)
and credit them in `third_party/NOTICE.md`.

## Hard rules

1. No secrets, tokens, or `/home/<user>/...` host paths in commits.
2. GPQA Diamond plaintext is cache-only; never commit it.
3. Official BFCL scorers stay unforked if/when added.
4. GSM8K: last-bolded number, never first-number.
5. Subset metrics always report `@n` and Wilson 95% CI.
6. Distinct item contracts: do not mix r0b0bench core-subset GSM8K@200
   (head-n) with even-spread GSM8K samples.
7. tau2 is out.
8. Output / traces stay outside the git tree (or gitignored `out/`).
9. Ship on `main` when the user asks to publish commits; no extra
   productization (Pages, GHCR) unless asked.

## CLI

`sm12x-bench` with subcommands `profiles / doctor / deps / run / compare`.
The public flag surface uses the `SM12X_*` env namespace; document new
flags in the README before adding them.

---
> Source: [SM12X-SOCOM/SM12X-LLM-BENCH](https://github.com/SM12X-SOCOM/SM12X-LLM-BENCH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
