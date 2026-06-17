## ctx2skill

> Faithful reproduction of Ctx2Skill (arXiv:2604.27660v2) — multi-agent self-play framework that autonomously discovers, refines, and selects context-specific skills from complex documents without human annotation or external feedback. N=5 iterations, M=5 tasks/iteration, 5 agent roles (Challenger/Reasoner/Judge/Proposer/Generator), Cross-Time Replay mechanism.


# Ctx2Skill — Autonomous Context-Specific Skill Discovery

Faithful open-source reproduction of arXiv:2604.27660v2.

## Trigger

User mentions: "从上下文中提取技能" / "用 Ctx2Skill 提取" / "learn from this context" / "根据这篇文档生成技能" / "skill discovery from context"

## How to Use

```python
from ctx2skill import Ctx2SkillConfig, Ctx2Skill

config = Ctx2SkillConfig(
    api_base="your-api-base",
    api_key="your-api-key",
    backbone_model="gpt-4.1",
    judge_model="gpt-4.1",
)

with open("context.txt", "r") as f:
    context = f.read()

skill = Ctx2Skill(config).run(context)
# skill is a Markdown document — prepend to system prompt at inference time
```

## CLI

```bash
python examples/run_ctx2skill.py \
    --context context.txt \
    --api-base https://api.openai.com/v1 \
    --api-key sk-... \
    --model gpt-4.1 \
    --output skills.md
```

## Architecture

- `ctx2skill/core.py` — Self-play loop orchestrator (N iterations)
- `ctx2skill/agents.py` — 5 agent roles: Challenger, Reasoner, Judge, Proposer, Generator
- `ctx2skill/replay.py` — Cross-time replay mechanism (selects best skill set)
- `ctx2skill/utils.py` — LLM API calls, prompt templates
- `ctx2skill/config.py` — Configuration dataclass

## Capability Baseline

| Dimension | Score |
|-----------|:-----:|
| 5 agent role completeness | 10/10 |
| Self-play loop (N=5, M=5, co-evolving skills) | 10/10 |
| Cross-Time Replay (probe sets + Laplace product) | 10/10 |
| Proposer-Generator decoupling | 10/10 |
| JSON output parsing robustness | 8/10 |
| Test coverage | 9/10 |
| CLI usability | 9/10 |
| Python API ergonomics | 9/10 |
| End-to-end runnable (requires LLM API) | 8/10 |
| Consistency with paper | 10/10 |
| Documentation | 8/10 |
| **Overall** | **91/110** |

## Comparison with Complementary Framework

| Dimension | Gap |
|-----------|:---:|
| Automatic skill discovery | −1 |
| Adversarial self-play | −2 |
| Iteration artifact selection | −2 |
| Structured quality verification | +2 |
| Cross-domain mapping | +3 |
| Deviation signal library | +2 |
| Cross-model portability | 0 |
| Runtime cost | +1 |
| Skill deduplication | 0 |
| Auditability | +2 |

Positive gap = we lead; negative gap = Ctx2Skill leads.

## Dependencies

- Python 3.10+
- httpx (for LLM API calls)
- Access to an OpenAI-compatible LLM API

## References

Si et al., "From Context to Skills: Can Language Models Learn from Context Skillfully?" arXiv:2604.27660v2 (2026)

---
> Source: [eluckydog/Ctx2Skill-](https://github.com/eluckydog/Ctx2Skill-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
