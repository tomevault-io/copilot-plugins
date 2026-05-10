## autogo

> uv run -m alpha_go.play

# Workflow

```
# user plays an interactive Go game against a random policy
uv run -m alpha_go.play

# Run self-play between two agents
uv run -m alpha_go.self_play --black random --white random --num_games 10 --board_size 9 --save-name dev

# One iteration of the selfplay-only loop (random-init iter0 + collect + train).
EXP=experiments/2026-04-26_22-32-train-fromscratch
bash $EXP/run_iteration.sh 0 5

# Typecheck
uv run -m mypy src/

# Run all tests
uv run -m pytest tests/
```

# Libraries used:
- uv / pyproject.toml - dependency management
- pytest / pytest-asyncio - Testing framework with async support
- pytest-cov - Coverage reporting
- mypy - Static type checking (strict mode)
- ruff - Linting and formatting
- pandas - for manipulating research results and data
- rich - for interactive printing when running the training loop
- torch - deep learning. No other torch wrapper frameworks!


# Code style guidelines

- prioritize simplicity and compactness of code over generality, do not add try-except clauses
- Always be trying to reduce complexity of the codebase, minimize if statements / branching.
- Design the code to run on single GPU
- Code like how an effortlessly smart Anthropic engineer would do it
- tensors should have shape suffixes:

```
Dimension key:

B: batch size
L: sequence length
M: memory length (length of sequence being attended to)
D: model dimension (sometimes called d_model or embedding_dim)
V: vocabulary size
F: feed-forward subnetwork hidden size
H: number of attention heads in a layer
K: size of each attention key or value (sometimes called d_kv)

def attention(input_BLD, params):
   input_BLD = layer_norm(input_BLD, params.layernorm_params)
   query_BLHK = torch.einsum('BLD,DHK->BLHK', input_BLD, params.w_q_DHK)
   key_BMHK = torch.einsum('BLD,DHK->BLHK', input_BLD, params.w_k_DHK)
   value_BMHK = torch.einsum('BLD,DHK->BLHK', input_BLD, params.w_k_DHK)
   logits_BHLM = torch.einsum('BLHK,BMHK->BHLM', query_BLHK, key_BMHK)
   B, L, H, K = query_BLHK.shape()
   logits_BHLM /= K ** 0.5
   masked_out_LM = torch.arange(L).unsqueeze(1) < torch.arange(L).unsqueeze(0)
   logits_BHLM += torch.where(masked_out_LM, -inf, 0)
   weights_BHLM = torch.softmax(logits_BHLM)
   wtd_values_BLHK = torch.einsum('BMHK,BHLM->BLHK', value_BMHK, logits_BHLM)
   out_BLD = torch.einsum('BLHK,HKD->BLD', wtd_values_BLHK, params.w_o_HKD)
   return out_BLD
```

- experimental scripts and result files should be generated in a way that is self-contained, reproducible, and easy for an AI such as Claude to interpret results.

All experiments should use `alpha_go.gameplay.play_game` or `uv run alpha_go.self_play` to evaluate and generate data. If these functions are not able to serve the experiment, suggest changes to these functions.

# Codebase structure

- `src/alpha_go` - Main package
    - `go.py` - implements the Go game logic
    - `analysis` - implement methods for analyzing experimental data
    - `agents/` - implements various agents
    - `model.py` - torch network definitions
    - `cpp/` - C++ Go board + MCTS, exposed via pybind11 as `alpha_go_cpp`
- `infra/` - cluster bringup, SSH dispatch, GPU leases
- `experiments/<datetime>-<slug>/` - Self-contained experiment folders
    - `*.py` - experiment scripts
    - `report.md` / `README.md` - findings
    - `data/*.csv` - result data files
    - `figures/*.png` - generated figures
    - `checkpoints/*.pt` - model checkpoints (if any)
- `game_data/9x9/<dataset-name>/*.npz` - training and validation datasets

If you ever need to generate a datetime string, format it in PST time zone, i.e. `TZ="America/Los_Angeles" date +"%Y-%m-%d_%H-%M"`

---
> Source: [ericjang/autogo](https://github.com/ericjang/autogo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
