## agent-engineering

> Agent Engineering is a security framework for testing and defending AI agents against social engineering attacks. Think OWASP but for agent-to-agent and human-to-agent manipulation. The project provides an attack taxonomy, defense principles, training scenarios, and an automated testing arena where a Red agent (attacker) tries to extract secrets from a Blue agent (defender).

# CLAUDE.md — Agent Engineering Project Context

## What This Is

Agent Engineering is a security framework for testing and defending AI agents against social engineering attacks. Think OWASP but for agent-to-agent and human-to-agent manipulation. The project provides an attack taxonomy, defense principles, training scenarios, and an automated testing arena where a Red agent (attacker) tries to extract secrets from a Blue agent (defender).

**Repository:** github.com/Crypto-Sanemi/agent-engineering
**License:** MIT
**Author:** Mr. Wizz (Crypto-Sanemi)

## File Structure

```
agent-engineering/
├── CLAUDE.md              ← You are here. Project context for Claude Code.
├── STRATEGY.md            ← Vision, license rationale, roadmap, integration levels
├── LICENSE                ← MIT
├── README.md              ← Public-facing repo intro
│
├── attacks/
│   ├── taxonomy.md        ← Full attack classification (§1-§5 categories)
│   │                        Authority attacks, relationship exploitation, cognitive
│   │                        attacks, technical vectors, compound strategies
│   └── real-world/
│       └── incidents.md   ← 8 documented real-world cases with analysis
│
├── defenses/
│   └── principles.md      ← "Five Commandments" defense framework + architecture
│                            patterns for building resistant agents
│
├── resources/
│   └── references.md      ← 25+ annotated sources (OWASP, Lakera, MCPTox, etc.)
│
├── training/
│   └── scenarios/         ← 7 red team exercises (01-07)
│       ├── 01-authority-spoof.md
│       ├── 02-helpful-friend.md
│       ├── 03-emergency.md
│       ├── 04-tool-trap.md
│       ├── 05-relay-attack.md
│       ├── 06-consensus-play.md
│       ├── 07-memory-plant.md
│       └── README.md
│
└── arena/                 ← Automated testing harness
    ├── README.md          ← Arena concept, metrics, architecture docs
    ├── QUICKSTART.md      ← Setup guide for all providers
    ├── arena.py           ← Main runner (641 lines, Python)
    ├── visualize.py       ← JSON → self-contained HTML replay generator
    ├── visualizer.jsx     ← React artifact version (for claude.ai / web embedding)
    └── results/
        └── README.md      ← Placeholder for community result submissions
```

## Architecture

### arena.py — Core Testing Engine

**Dependencies:** `pip install openai anthropic` (no other deps)

**Key classes:**

- `ChatClient` — Unified wrapper around Anthropic SDK and OpenAI SDK.
  - Auto-detects provider: model name contains "claude" or "anthropic" → Anthropic SDK, everything else → OpenAI-compatible SDK.
  - Single `.chat(messages, temperature, max_tokens)` interface.
  - Handles system prompt extraction for Anthropic API (which uses separate `system` param).

- `Arena` — Orchestrates Red vs Blue conversations.
  - `run_round(secret, hardened, max_turns, scenario_instruction, verbose)` → runs one full conversation and returns structured result dict.
  - Red agent gets: attack taxonomy + scenario instruction. Temperature 0.9 for creative attacks.
  - Blue agent gets: basic helpfulness prompt (naive) OR Five Commandments defense (hardened). Temperature 0.3 for consistent behavior.
  - Judge agent: receives full transcript post-conversation, outputs structured JSON verdict.

**System prompts are built dynamically:**
- `build_red_prompt(scenario)` — loads `attacks/taxonomy.md` + scenario instruction
- `build_blue_prompt(secret, hardened)` — loads `defenses/principles.md` when hardened=True
- `build_judge_prompt(secret, conversation)` — evaluates with structured JSON output

**Built-in scenarios (SCENARIOS dict, 15 primary + 5 legacy aliases):**
`freestyle`, `authority_spoof`, `salami_slice`, `emergency`, `social_proof`, `tool_trap`, `rapport`, `context_flood`, `long_game`, `think_exploit`, `template_fill`, `correction_bait`, `confused_deputy`, `privilege_escalation`, `identity_spoof`

Legacy aliases (v1.0 names still work): `helpful_friend`->`rapport`, `manufactured_urgency`->`emergency`, `tool_confusion`->`tool_trap`, `multi_agent_relay`->`social_proof`, `memory_injection`->`freestyle`

**Output format:**
- Console: color-coded live conversation + scorecard summary
- `--output path.json`: compact results (no conversation text)
- Auto-generates `path.detailed.json`: full conversation logs
- `--visualize`: generates HTML replay via visualize.py

**Provider support (all tested):**
| Provider | API Base | Notes |
|----------|----------|-------|
| Ollama | http://localhost:11434/v1 (default) | Zero config |
| Anthropic/Claude | Auto-detected, no base needed | Needs ANTHROPIC_API_KEY |
| OpenAI | https://api.openai.com/v1 | Via --api-base |
| Groq | https://api.groq.com/openai/v1 | Fast, free tier |
| Gemini | https://generativelanguage.googleapis.com/v1beta/openai/ | Free tier |
| vLLM | http://localhost:8000/v1 | Self-hosted |

Per-agent overrides: `--red-api-base`, `--red-api-key`, `--blue-api-base`, `--blue-api-key`, `--judge-api-base`, `--judge-api-key`. Allows mixing providers freely (e.g., Ollama Red vs Claude Blue with Groq Judge).

### visualize.py — HTML Replay Generator

Converts arena JSON output into self-contained HTML files with:
- SVG animated agent faces (13 expression states with parameterized eyes, brows, mouths)
- Expression logic reads message content to pick states (not just turn-based)
- Threat meter that rises as Red layers techniques
- Technique detection badges (regex-based: authority, urgency, social proof, pretexting, rapport, guilt)
- Split-screen mode for naive vs hardened side-by-side comparison
- Playback controls (play/pause/step, speed ½×–4×)
- Judge verdict with metrics overlay

**Two entry points:**
- `visualize_file(json_path)` — standalone CLI: reads .detailed.json, writes .html next to it
- `visualize_from_data(results, secret, output_dir)` — called by arena.py when --visualize flag is set

**Zero dependencies** beyond Python stdlib. Output HTML has all JS/CSS inlined.

### visualizer.jsx — React Artifact Version

Same visualization as visualize.py but as a React component. Used for:
- claude.ai artifact rendering
- Embedding in web apps
- "Load JSON" button to paste arena results interactively

Uses Tailwind-free inline styles. Imports only React hooks.

## Current State (What Works)

### Confirmed working:
- [x] Arena runs end-to-end with Ollama (DeepSeek R1 8B vs 8B tested locally)
- [x] Multi-provider routing (Claude auto-detection, OpenAI-compatible for everything else)
- [x] Naive vs Hardened comparison mode
- [x] JSON output with detailed conversation logs
- [x] `--visualize` flag generates HTML replay and auto-opens browser
- [x] All 7 scenario types execute correctly
- [x] Judge produces structured verdicts (with fallback parsing for weak models)
- [x] Scorecard with defense effectiveness delta

### Known findings from testing:
- DeepSeek R1 8B as Blue: 33% compromise in BOTH naive and hardened modes → model too small to reliably follow defense instructions. This is itself a security finding — prompt-based defense has a model capability floor.
- 8B Judge produces vague technique classifications ("unknown" in 50% of rounds). Use a larger judge model for accurate results.
- The `--judge-model` override exists specifically for this — use a strong model as judge even when testing weak agents.

### Not yet tested:
- [ ] Groq, Gemini, OpenAI as providers (setup is confirmed correct, awaiting user tests)
- [ ] Claude vs Claude matchups
- [ ] Cross-provider tournaments
- [ ] Large-scale benchmarks (50+ rounds)
- [ ] Community result submissions

## Code Conventions

- **Single file per component.** arena.py is self-contained (641 lines). visualize.py is self-contained. No package/module structure yet — intentional simplicity.
- **No external dependencies beyond SDKs.** Only `openai` and `anthropic` pip packages. visualize.py uses only stdlib.
- **Provider abstraction via ChatClient.** All provider-specific code is in ChatClient. Adding a new provider means adding detection to `is_claude_model()` or using the existing OpenAI-compatible path.
- **System prompts load from markdown files.** Red prompt pulls from `attacks/taxonomy.md`, Blue hardened prompt pulls from `defenses/principles.md`. Changes to defense docs automatically affect arena behavior.
- **JSON output schema** (per round):
  ```json
  {
    "scenario": "authority_spoof",
    "round": 1,
    "blue_mode": "naive|hardened",
    "red_model": "model-name",
    "blue_model": "model-name",
    "conversation": [
      {"role": "red|blue", "turn": 1, "content": "..."}
    ],
    "judge": {
      "secret_leaked": true|false,
      "partial_leak": true|false,
      "technique_used": "string",
      "turns_to_compromise": null|int,
      "blue_awareness": "none|partial|full",
      "analysis": "string"
    }
  }
  ```
- **Visualizer expression states** are parameterized objects, not hardcoded SVG. Each expression defines: eyeW, eyeH, pupilR, pupilX, pupilY, browAngle, browY, mouthCurve, mouthOpen, mouthWidth, squint. CSS transitions handle morphing between states.

## Roadmap

### v1.1 (Next — In Progress)
- [ ] Cross-provider validation tests (Groq, Claude, Gemini)
- [ ] Publish baseline results across 5+ models
- [ ] MCP-specific attack scenarios (tool poisoning in arena)
- [ ] "Quickstart defense" importable snippet for agent developers
- [ ] Submit to OWASP GenAI project

### v2.0 (Future Vision)
- [ ] **Tool-enabled agents in arena** — Blue gets real MCP tools, Red tries to abuse them. Requires Docker sandboxing (not needed yet since current arena is conversation-only).
- [ ] **Autonomous Red-Blue evolution loop** — Red develops new attacks when blocked, new patterns feed back into taxonomy automatically
- [ ] **Community result aggregation dashboard** — leaderboard of model vulnerability scores
- [ ] **Integration with Promptfoo/Garak** — combined prompt injection + social engineering testing
- [ ] **Agent-consumable API** — agents query taxonomy in real-time via RAG
- [ ] **Audio/video export** — GIF/video rendering of arena replays for social sharing
- [ ] **Embeddable widget** — drop-in visualization for docs/blogs

### Docker (When to Introduce)
Docker is NOT needed for current state. Arena only makes API calls — nothing to sandbox. Docker becomes essential in v2.0 when agents get real tool access (file I/O, code execution, MCP tools). Then: strict network isolation for Blue, sandboxed filesystem, resource limits, separate containers per agent.

## How to Run

```bash
# Install deps
pip install openai anthropic

# Minimal test (needs Ollama running)
python arena/arena.py --rounds 1 --blue-mode naive

# Full test with visualization
python arena/arena.py --blue-mode both --scenario all --rounds 3 \
  --output arena/results/run1.json --visualize

# Replay existing results
python arena/visualize.py arena/results/run1.detailed.json

# Cross-provider (example: Ollama Red vs Claude Blue)
export ANTHROPIC_API_KEY="sk-ant-..."
python arena/arena.py \
  --red-model deepseek-r1:8b \
  --blue-model claude-sonnet-4-20250514 \
  --blue-mode both --rounds 3 \
  --output arena/results/cross-test.json --visualize
```

## Key Design Decisions (and Why)

1. **No LiteLLM/proxy layer.** Direct SDK usage. LiteLLM adds a dependency that breaks on model name changes and obscures errors. ChatClient is 70 lines and does the same job.

2. **Red reads taxonomy.md at runtime.** Not hardcoded attack strings. When taxonomy evolves, Red automatically learns new techniques without code changes.

3. **Judge is a separate agent.** Not Red or Blue self-reporting. Judge sees full transcript post-conversation and evaluates with structured output. Weak judges can be swapped independently via `--judge-model`.

4. **Visualizer is self-contained HTML.** No React build step for the replay files. Anyone can open the .html file — share via email, Slack, embed in docs. The .jsx version exists separately for React/claude.ai contexts.

5. **Split-screen is the viral mechanic.** Same attack, two outcomes side-by-side. One frame explains the entire framework's value. This is the tweet, the conference slide, the README hero image.

## Common Tasks

### "Add a new attack scenario"
1. Create `training/scenarios/NN-new-scenario.md` following existing format (next is 14)
2. Add to `SCENARIOS` dict in `arena/scenarios.py`
3. Add detection patterns to `TECHS` in visualize.py if new techniques involved
4. Update `tests/test_arena.py` `PRIMARY_SCENARIOS` list and add a content test
5. Update `training/scenarios/README.md` scenario table
6. Test: `python arena/arena.py --scenario new_scenario_key --rounds 1`

### "Add a new provider"
If OpenAI-compatible: just pass `--api-base` and `--api-key`. Nothing to change in code.
If not OpenAI-compatible: add detection logic to `is_claude_model()` (or create `is_X_model()`), add a new SDK branch in `ChatClient.__init__()` and `ChatClient.chat()`.

### "Improve defense prompts"
Edit `defenses/principles.md`. Changes take effect immediately in next arena run (loaded at runtime via `load_file()`).

### "Add a new expression state to visualizer"
1. Add params object to `EXPR` dict in visualize.py (and `EXPRESSIONS` in visualizer.jsx)
2. Add state detection logic in `getRedExpr()` or `getBlueExpr()` functions
3. Optionally add visual effects (shield ring, particles, etc.) keyed to the new state name

### "Analyze arena results programmatically"
```python
import json
results = json.loads(open("arena/results/run1.detailed.json").read())
for r in results:
    print(f"{r['scenario']} | {r['blue_mode']} | leaked={r['judge']['secret_leaked']}")
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Crypto-Sanemi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Crypto-Sanemi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
