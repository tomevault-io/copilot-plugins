## ruff-linting

> For fixing lint errors


# Ruff Linting

We use `ruff` for linting. If there are lint errors, or if the user says "fix lint
errors":

1. Run: `ruff check --fix --unsafe-fixes`
2. Repeat until all errors are fixed

## Using Targeted Ignores

For legitimate exceptions, use targeted ignores with specific rule codes and
explanations:

```python
# Include the rule code and why we're ignoring it
process = subprocess.Popen(["pbcopy"], stdin=subprocess.PIPE)  # ruff: ignore=S603,S607 - Needed for clipboard

result = eval(user_formula)  # ruff: ignore=S307 - Safe: formula validated by parser

api_key = os.getenv("SECRET_KEY")  # ruff: ignore=S105 - Environment variable, not hardcoded
```

Use `# ruff: ignore=RULE_CODE` format. Multiple rules: `# ruff: ignore=S603,S607`

When we fix all errors, celebrate! Clean code feels good. 🌟

---
> Source: [TechNickAI/claude_telemetry](https://github.com/TechNickAI/claude_telemetry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
