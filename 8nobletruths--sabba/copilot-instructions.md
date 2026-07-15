## sabba

> Sabba finds security bugs and proves each one by triggering it. A model proposes

# Sabba Agent

Sabba finds security bugs and proves each one by triggering it. A model proposes
candidates, and an execution oracle runs an exploit in a sandbox and decides whether a
security property broke. Nothing is reported unless the exploit reproduces. The pipeline
that exists today targets C and C++ memory safety. The design extends the same discipline
to any language and to smart contracts, Solidity first.

## Where to start reading

- `docs/SABBA_AGENT_DESIGN.md`: how the current C and C++ bug-finder fits together. The
  oracle is the anchor; every other part proposes candidates.
- `docs/PROVERS_MULTI_DOMAIN_DESIGN.md`: how the oracle generalizes from one sanitizer
  pipeline into a registry of provers, one per vulnerability class and runtime. Covers
  managed languages, web applications, and Web3 and Solidity, where a finding is proven on
  an EVM fork by a fund-drain or solvency invariant. Read this before adding a prover for a
  new language or chain.
- `docs/WATER_LAYER_DESIGN.md`: the design for the next layer, an agent that learns skills,
  keeps them as runnable code, runs without a frontier model, and can be rebuilt from a
  signed seed after deletion. Read this before working on anything under a future `water/`
  package. Provers are skills the genome accumulates.

## Layout

- `sabba/harness/oracle.py`: the execution oracle, the shared verifier. Never fork it. It
  generalizes into the prover registry.
- `sabba/harness/agent.py`, `sabba/harness/orchestrator.py`: the reasoning agent and driver.
- `sabba/harness/fuzz.py`: the model writes a harness, the oracle proves the crash. This is
  the seed of the Water Layer's skill compiler.
- `sabba/harness/symbolic/synth.py`: the Z3 synthesizer.
- `sabba/llm/`: the model backend switch (OpenRouter, Anthropic, GLM). The Water Layer's
  Teacher and Resident both sit behind `llm/base.py`.
- `sabba/sandbox/`: docker and local execution isolation. It grows environment builders per
  domain: a compile step, a fuzz harness, a seeded web container, an EVM fork.
- `sabba/memory.py`, `sabba/history.py`: the seed of the Water Layer genome.
- `sabba/cli.py`, `sabba/repl.py`, `sabba/tui.py`: the command line, the REPL, the TUI.

## Working here

- Writing voice for all code, docs, and commits: human-authored, plain, no em-dashes, no
  AI-generated look. Match the tone of the existing docs.
- Keep secrets out of the repo. A pre-commit hook blocks anything that looks like a
  credential. See `CONTRIBUTING.md`.
- The oracle is shared by the agent, by any future training reward, and by serving. What
  counts as a real result is decided in one place, and provers are added, never forked.
- Exploit PoCs are dual-use. Prove on a fork or a local target, never against systems you
  are not authorized to touch, and disclose with the re-runnable bundle.
- Do not commit or push without being asked.

---
> Source: [8NobleTruths/sabba](https://github.com/8NobleTruths/sabba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
