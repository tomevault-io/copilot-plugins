## agent-experiments

> - **Environment name**: `finance-agent`

# Finance Agent Project

## Environment Setup

### Conda Environment
- **Environment name**: `finance-agent`
- **Activation command**: `source /Users/lakhan/miniconda3/etc/profile.d/conda.sh && conda activate finance-agent`

Use the `finance-agent` conda environment for all Python-related tasks (running scripts, tests, imports verification, etc.).

## Running the Application

### Celery Worker
```bash
./start_celery.sh
```

### FastAPI Server
```bash
./start_api.sh
```

## Coding Style

### Python Function Calls
Always use keyword arguments when calling functions (except for single obvious arguments):

```python
# Bad
process_data("input.csv", "output/", True, "xlsx")

# Good
process_data(
    input_path="input.csv",
    output_dir="output/",
    verbose=True,
    format="xlsx",
)
```

Single argument calls can use positional:
```python
# OK
len(items)
os.path.exists(path)
```

---
> Source: [lakhand7/agent-experiments](https://github.com/lakhand7/agent-experiments) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
