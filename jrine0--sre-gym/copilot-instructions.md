## sre-gym

> An RL training environment for AI agents to diagnose and fix Kubernetes production errors using kubectl tools. Trains agents on realistic K8s failure scenarios with dense PBRS rewards.

# Self-Healing Kubernetes SRE Gym

## Overview
An RL training environment for AI agents to diagnose and fix Kubernetes production errors using kubectl tools. Trains agents on realistic K8s failure scenarios with dense PBRS rewards.

## Architecture
```
sre-gym/
├── CLAUDE.md              # This file
├── pyproject.toml         # Python package
├── Dockerfile             # Lightweight container image
├── sre_gym/
│   ├── __init__.py
│   ├── env.py             # OpenEnv-compliant environment (step/reset)
│   ├── models.py          # Pydantic models: K8sAction, K8sObservation, K8sState
│   ├── grader.py          # AssertionEngine for pod status checks
│   ├── tasks/
│   │   ├── __init__.py
│   │   ├── easy.py        # Pod CrashLoopBackOff (ConfigMap missing)
│   │   ├── medium.py      # OOMKilled triage (RAM limits)
│   │   └── hard.py        # Cascading latency failure (3 microservices)
│   ├── mcp_server.py      # MCP tool definitions for Claude Code
│   ├── rewards.py         # PBRS reward shaping engine
│   └── inference.py       # OpenAI client with structured <tool_call> logs
├── .claude/
│   └── skills/
│       ├── SKILL.md       # sre-env-builder
│       ├── reward-engineer.md
│       └── references/
│           ├── pbrs-stability.md
│           └── sre-metrics.md
└── tests/
    ├── test_env.py
    └── test_tasks.py
```

## Technical Requirements
- Runs on 2 vCPUs, 8GB RAM
- Isolated Docker sandbox (kind/k3s)
- All rewards scaled 0.0–1.0
- PBRS dense feedback + sparse terminal reward
- MCP tool definitions for IDE integration

## Quick Start
```bash
pip install -e sre-gym
python -m sre_gym.env --task easy --episodes 10
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jrine0)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Jrine0)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
