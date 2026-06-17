## skeptic

> Use when running a question-first Veridical Data Science lifecycle: formulate the question, define protocol rules, clean under those rules, examine what the data can support, execute route-specific analysis under an explicit contract lock, evaluate claims, and communicate only claims that survived evaluation. Use when the user says "data analysis", "analyze this dataset", "analyze this CSV", "analyze this parquet", "data science", "Skeptic", or invokes any /skeptic subcommand.


# Skeptic - Veridical Data Science Lifecycle

Use a question-first lifecycle. Do not start from a favorite method. Do not assume train/validation/test splits. Do not assume predictive modeling is the default endpoint. `protocol` decides the project rules before `clean` starts; later stages must obey those rules. PCS applies throughout. `/skeptic:evaluate` is the formal route-appropriate PCS review before `/skeptic:communicate`.

## Mandatory Read: Core Principles

Before executing any Skeptic subcommand, read `references/core-principles.md`. That file defines the stage order, artifact model, stage purposes, and architecture contract. If any downstream stage file conflicts with it, `references/core-principles.md` wins.

## Subcommands

`/skeptic:formulate`, `/skeptic:protocol`, `/skeptic:clean`, `/skeptic:examine`, `/skeptic:analyze`, `/skeptic:evaluate`, `/skeptic:communicate`, `/skeptic:auto`.

## How Each Stage Loads Its Context

- stage entry: `references/{stage}/{stage}.md` -- read once at stage start.
- per-cycle specs: `references/{stage}/cycles/{cycle}.yaml` -- read one at a time as each cycle runs.

Each stage writes a canonical YAML (`{NN}_{stage}.yaml`), runs a project-side Python script (`{NN}_{stage}.py`, one function per cycle, invoked `--cycle X`), and renders the final markdown (`{NN}_{stage}.md`) only at stage close.

---
> Source: [Filivignaga/skeptic](https://github.com/Filivignaga/skeptic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
