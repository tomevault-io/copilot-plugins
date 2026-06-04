## remat

> If the regular local Python or `uv run` path cannot run tests because dependencies such as `pytest`, `expecttest`, or the required PyTorch build are missing, use the llama4x GB unified conda environment from the MSL root:

# remat Agent Notes

## Python Test Environment

If the regular local Python or `uv run` path cannot run tests because dependencies such as `pytest`, `expecttest`, or the required PyTorch build are missing, use the llama4x GB unified conda environment from the MSL root:

```bash
source dev/xl_conda.sh activate xlformers_gbunified_conda:conveyor-stable
PYTHONPATH=remat python -m pytest remat/tests/api_test.py -q
```

Run the activation command from `genai/msl/`, or adjust the `dev/xl_conda.sh` path accordingly.

---
> Source: [meta-pytorch/remat](https://github.com/meta-pytorch/remat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
