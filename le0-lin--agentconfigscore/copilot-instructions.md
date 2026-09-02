## agentconfigscore

> Keep the runtime dependency-free on Python 3.10+.

# Agent instructions — AgentConfigScore

Keep the runtime dependency-free on Python 3.10+.

## Verification

- Run `PYTHONPATH=src python -m unittest discover -s tests -v` after scanner or CLI changes.
- Run `PYTHONPATH=src python -m agent_config_score.cli examples/demo` when changing scoring or output formatting.

## Scope

- Put scanning and scoring logic in `src/agent_config_score/scanner.py`.
- Keep CLI argument parsing and terminal output in `src/agent_config_score/cli.py`.
- Add a regression test for every false positive or false negative that is fixed.

## Safety

- Never print the full value of a detected credential.
- Keep contradiction detection conservative; do not claim semantic conflicts without strong evidence.

---
> Source: [LE0-Lin/AgentConfigScore](https://github.com/LE0-Lin/AgentConfigScore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
