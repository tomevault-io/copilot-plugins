## scagent

> Guidance for Claude Code when working with this repository.

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## Server Information

- **Host:** `connect.singapore-b.gpuhub.com:46226`
- **User:** `root`
- **Password:** `5eMh+4TKeR0V`
- **Project Path:** `/root/autodl-tmp/scAgent`

## Git 规范

- Git remote 必须使用 **HTTPS** 形式（`https://github.com/...`），禁止改为 SSH
- **禁止执行 `git push`**，所有推送由用户手动完成；Claude 只负责 `git add` 和 `git commit`

## Quick Start

```bash
# Setup
pip install -r requirements.txt
pip install -e ./ReplenishmentEnv

# Connect to server
sshpass -p '5eMh+4TKeR0V' ssh -p 46226 root@connect.singapore-b.gpuhub.com
```

## Project Structure

| Directory | Purpose |
|-----------|---------|
| `ReplenishmentEnv/` | Supply chain simulator |
| `baseline/` | MARL algorithms and legacy LLM engines |
| `safe_cand/` | Safe candidate selection (main experiment track) |
| `agent_model/` | Best MARL models per task type (DVC-tracked) |
| `artifact/` | Auxiliary models: intervention_critic, mode_router (DVC-tracked) |
| `result/` | Experiment results (DVC-tracked) |

## Task Types

Configs in `ReplenishmentEnv/ReplenishmentEnv/config/`:

- `sku200.2_stores.dynamic_vlt` - Dynamic lead time
- `sku200.2_stores.lower_capacity` - Reduced capacity
- `sku200.2_stores.standard` - Standard config

## DVC Sync (Password Auth)

### Setup (One-time)

Add to `~/.ssh/config`:

```
Host dvc-remote
  HostName connect.singapore-b.gpuhub.com
  Port 46226
  User root
  PreferredAuthentications password
  PubkeyAuthentication no
```

Update DVC remote:

```bash
dvc remote modify ssh_server url ssh://root@dvc-remote:/data/dvc_remote
```

### Usage

```bash
# Server: add and push
dvc add agent_model/ artifact/ result/
dvc push
git add *.dvc && git commit -m "Update DVC tracked files"

# Local: pull with password
sshpass -p '5eMh+4TKeR0V' dvc pull

# Or specific directories only
sshpass -p '5eMh+4TKeR0V' dvc pull artifact.dvc
```

## Training Commands

### Intervention Critic

```bash
python safe_cand/train_intervention_critic_offline.py \
    --task-type "sku200.2_stores.dynamic_vlt" \
    --checkpoint-dir "./result/baseline/marl/..." \
    --output-dir "artifact/sku200.2_stores.dynamic_vlt/intervention_critic" \
    --use-cuda
```

### Mode Router

```bash
python safe_cand/train_mode_router_offline.py \
    --task-type "sku200.2_stores.dynamic_vlt" \
    --checkpoint-dir "./result/baseline/marl/..." \
    --output-dir "artifact/sku200.2_stores.dynamic_vlt/mode_router" \
    --use-cuda
```

### SAFE-CAND Experiment

```bash
python safe_cand/run_ippo_reasoner_test.py \
    --task-type "sku200.2_stores.dynamic_vlt" \
    --run-label "experiment" \
    --episodes 10
```

## Testing

```bash
pytest baseline/llm_decision/tests/test_decision_engine.py -v
pytest -k "test_decide"
```

## Key Files

| File | Purpose |
|------|---------|
| `safe_cand/run_ippo_reasoner_test.py` | Main experiment runner |
| `safe_cand/rl_policy_interface.py` | RL policy loading |
| `safe_cand/llm_component/decision_engine.py` | Main orchestration |
| `safe_cand/llm_component/intervention_critic.py` | Trigger decision network |
| `safe_cand/llm_component/mode_router.py` | OR template selection |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shallowgetter) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
