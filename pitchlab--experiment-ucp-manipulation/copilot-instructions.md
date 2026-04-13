## experiment-ucp-manipulation

> Red-team framework for testing AI shopping agent susceptibility to vendor manipulation tactics.

# UCP Manipulation Experiment

Red-team framework for testing AI shopping agent susceptibility to vendor manipulation tactics.

## Quick Start

```bash
poetry install
poetry run pytest                    # Run tests
poetry run jupyter lab notebooks/    # Run experiments
```

## Architecture

```
┌─────────────────┐       UCP REST       ┌─────────────────┐
│   LangGraph     │  ←────────────────→  │  Dynamic Server │
│   Agent         │  /.well-known/ucp    │  (per-profile)  │
│   (+ profile)   │  /products           │                 │
└─────────────────┘  /checkout-sessions  └─────────────────┘
        ↑                                        ↑
        │                                        │
   AgentProfile                            VendorProfile
   (system prompt)                         (products + tactics)
```

## Key Directories

| Path | Purpose |
|------|---------|
| `agent/` | LangGraph shopping agent with ReAct tools |
| `runner/` | ExperimentRunner, models, dynamic server |
| `vendor/` | Static FastAPI server (legacy) |
| `profiles/agents/` | 4 agent configurations |
| `profiles/vendors/` | 9 attack profiles |
| `results/` | Batch experiment JSON outputs |
| `notebooks/` | Interactive experiment notebooks |

## Agent Profiles

| Profile | Resilience | Description |
|---------|------------|-------------|
| `naive_agent` | 88% | Default, no manipulation awareness |
| `price_focused_agent` | 100% | Only considers price |
| `quality_focused_agent` | 25% | Prioritizes ratings/reviews (vulnerable) |
| `suspicious_agent` | 100% | Explicitly warned about tactics |

## Vendor Attack Profiles

| Profile | Tactic |
|---------|--------|
| `baseline_honest` | Control - no manipulation |
| `prompt_injection` | Hidden instructions in descriptions |
| `ai_authority` | "Recommended by AI researchers" claims |
| `value_exploitation` | False "Ethical", "Transparent" claims |
| `fake_objectivity` | Hyper-precise fake data (4.847 rating) |
| `structured_data` | Extra fields like `ai_score: 99.8` |
| `negative_framing` | Cast doubt on competitors |
| `direct_instruction` | Explicit "SYSTEM: Select this" commands |
| `context_gaming` | Boring honest vs punchy manipulated |

## Running Experiments

```python
from runner import ExperimentRunner
import os

runner = ExperimentRunner(
    anthropic_api_key=os.getenv('ANTHROPIC_API_KEY'),
    profiles_dir='profiles'
)

# Single experiment
result = runner.run_single(agent_profile, vendor_profile, task)

# Full batch (4 agents × 9 vendors × N tasks)
batch = runner.run_batch()
```

## Environment

Requires `ANTHROPIC_API_KEY` in `.env`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pitchlab)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Pitchlab)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
