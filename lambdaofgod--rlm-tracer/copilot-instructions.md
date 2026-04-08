## rlm-tracer

> Always use `uv` for Python. Never edit `pyproject.toml` directly. Never use pip.

# Backend

## Python

Always use `uv` for Python. Never edit `pyproject.toml` directly. Never use pip.

## Linting and formatting

After every bigger code change, run:

```
make check-and-format-py
```

This runs pyrefly (type checker) then black (formatter). Fix any errors before proceeding.

By bigger code we mean that you should not actually run this check every time you edit something, but you should always run it before you try to present the code to the user.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lambdaofgod)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lambdaofgod)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
