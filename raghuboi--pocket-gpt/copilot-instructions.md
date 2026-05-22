## pocket-gpt

> From-scratch reproductions of foundational ML papers. Pure PyTorch, no frameworks.

# pocket-gpt

From-scratch reproductions of foundational ML papers. Pure PyTorch, no frameworks.

## Mission

Build minimal, readable reproductions that a senior engineer can `cat model.py` and understand every line. Each paper gets its own model file. Training loops stay under 400 lines.

## Structure Rules

**Flat root, vertical slices.** No `src/` directory. No more than 2 levels of nesting.

```
model.py              # Current paper's model
train.py              # Training loop
sample.py             # Inference
configurator.py       # Config override system
config/               # Python config files
data/                 # Dataset prep scripts
tests/                # Unit tests
requirements.txt      # Python dependencies
```

**Adding a new paper reproduction:**
1. Archive current model: `mv model.py model_attention.py` (if keeping multiple)
2. Write new `model.py` from scratch (~300-500 lines)
3. Update `train.py` if training loop differs significantly
4. Add config files to `config/`
5. Add data prep to `data/`
6. Add tests to `tests/`
7. Update README.md with new paper entry

## Code Standards

- **Pure PyTorch only.** No HuggingFace, no fairseq, no PyTorch Lightning.
- **One concept per class.** Each class maps to a paper section.
- **Docstrings with paper references.** Every class cites the paper section it implements.
- **Type hints on all public signatures.**
- **No nested functions in model code.** Extract to top-level or class methods.

## Config Pattern

Use `configurator.py` (Karpathy pattern). Configs are Python files with plain variable assignments. CLI args override config values.

```bash
python train.py config/baseline.py --batch_size=32 --dropout=0.2
```

## Testing

- Test each architectural component independently
- Verify shapes, gradients, and paper formulas
- Run: `pytest tests/ -v`

## Dependencies

Keep `requirements.txt` minimal. Only direct dependencies. No dev tools in production deps.

## Git

- Commit often with descriptive messages: `feat: implement scaled dot-product attention`
- One logical change per commit
- Keep `.gitignore` comprehensive (see below)

## .gitignore

Must include:
- Python: `__pycache__/`, `*.pyc`, `*.egg-info/`, `.pytest_cache/`
- Virtual envs: `.venv/`, `venv/`
- Training outputs: `out-*/`, `checkpoints/`
- Data: `data/*/raw/`, `data/*/*.bin`, `data/*/*.zip`, `data/*/*.tmp`
- Experiments: `wandb/`, `mlruns/`
- IDE: `.vscode/`, `.idea/`
- OS: `.DS_Store`, `Thumbs.db`
- Logs: `*.log`

---
> Source: [Raghuboi/pocket-gpt](https://github.com/Raghuboi/pocket-gpt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
