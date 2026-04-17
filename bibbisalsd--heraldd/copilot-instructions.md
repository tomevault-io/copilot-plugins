## heraldd

> > **For**: Gemini CLI agent sessions

# Harold / Jarvis Core - Developer Handoff for Gemini CLI

> **For**: Gemini CLI agent sessions
> **Primary task**: Keep the Herald website up to date with the current Jarvis Core codebase state
> **Last updated**: 2026-04-08

## Quick Orientation

This is **Jarvis Core** (codename Harold) — a local-first AI assistant runtime built in Python. It runs entirely on the user's machine using Ollama for LLM inference, Kokoro ONNX for TTS, and faster-whisper for STT. No cloud APIs required.

The project is a merger of two codebases:
- **Esoteric v0.2** (the "Mind") — cognitive layer: world model, evidence store, belief state, judge, CRSIS self-improvement
- **CLLM** (the "Spinal Cord") — deterministic pipeline: routing cascade, tool policy, admission control, contracts, rendering

**Owner**: billybart (James)
**Hardware**: i5-13420H, 16GB RAM, RTX 4050 6GB VRAM, Pop!_OS Linux

---

## Project Paths

| What | Path |
|------|------|
| Jarvis Core (Python) | `/home/billybart/Downloads/Harold/esotericv0.2CURRENT/Harold/Jarvis/Jarviscore/` |
| Herald Website (Next.js) | `/home/billybart/Downloads/Harold/site/New folder/` |
| Detailed handoff (full architecture) | `PROJECT_HANDOFF.md` (in Jarviscore root) |
| Docs/specs | `/home/billybart/Downloads/Harold/esotericv0.2CURRENT/Harold/docs/specs/` |

---

## Website Overview

The Herald website is a Next.js 15 static-export site (`output: "export"` in next.config.ts).

### Tech Stack
- **Framework**: Next.js 15 (App Router, static export)
- **3D**: Three.js via @react-three/fiber + @react-three/drei
- **Animation**: GSAP + Lenis smooth scroll
- **Styling**: CSS Modules
- **Package manager**: npm

### Site Structure
```
site/New folder/
  src/
    app/
      layout.tsx              # Root layout
      globals.css             # Global styles
      (site)/                 # Main route group
        page.tsx              # Homepage (renders Hero)
        applications/         # Applications page
        business/             # Business case page
        classification/       # Intent classification page
        cognitive-layer/      # Concept D / world model page
        comparison/           # vs ChatGPT/Claude/etc comparison page
        crsis/                # Self-improvement engine page
        how-it-works/         # Architecture walkthrough
        model-seats/          # Model lineup page
        numbers/              # Stats/metrics page
        pattern/              # CLLM pattern page
        problem/              # Problem statement page
        roadmap/              # Future plans page
        skeptic/              # Skeptic FAQ page
        world-model/          # World model deep dive
      explore/                # Interactive explorer
    components/
      Hero.tsx                # Landing hero with 3D scene + typewriter
      Nav.tsx                 # Navigation
      Footer.tsx              # Footer
      HowItWorks.tsx          # Architecture diagram section
      ModelSeats.tsx          # Model lineup cards
      Comparison.tsx          # Competitor comparison table
      CognitiveLayer.tsx      # World model explanation
      Crsis.tsx               # CRSIS self-improvement section
      Classification.tsx      # 5-stage routing visualization
      WorldModel.tsx          # Evidence store / belief state
      Numbers.tsx             # Stats (LOC, models, tools, etc.)
      Roadmap.tsx             # Future plans
      CodeBlock.tsx           # Code snippet display
      GsapSection.tsx         # GSAP scroll animation wrapper
      ScrollReveal.tsx        # Scroll-triggered reveal
      TextReveal.tsx          # Text animation
      PinnedSection.tsx       # Scroll-pinned section
      SmoothScroll.tsx        # Lenis wrapper
      ScrollProgress.tsx      # Scroll progress indicator
      SiteComponents.tsx      # Shared component exports
      Applications.tsx        # Use cases section
      Business.tsx            # Business model section
      Skeptic.tsx             # FAQ / pushback section
      Inversion.tsx           # Design philosophy section
      Problem.tsx             # Problem statement section
      ClosingCta.tsx          # Call to action
  public/
    esoteric.html             # Static Esoteric page
  out/                        # Static export output (pre-built)
  next.config.ts              # Static export config
  package.json                # herald-site, Next.js 15
```

### Build & Run
```bash
cd "/home/billybart/Downloads/Harold/site/New folder"
npm install
npm run dev      # Dev server
npm run build    # Static export to out/
```

---

## What Needs Updating on the Website

The website was built when the model lineup was different and before recent architectural changes. Here is what is now **outdated** and needs to be updated:

### 1. Model Seats (CRITICAL - ModelSeats.tsx)

The `MODEL_SEATS` array in `src/components/ModelSeats.tsx` shows the OLD model lineup. Update to match the current `jarvis/config.py`:

**Current config.py (ground truth):**
```
renderer_model_preferred:  gemma4:e2b        (was llama3.2:1b)
renderer_model_fallback:   gemma4:e4b        (was llama3.2:3b)
vision_lite_model:         gemma4:e2b        (was qwen2.5vl:3b — shared with renderer)
vision_bg1_model:          qwen3-vl:8b       (unchanged)
code_bg1_model:            deepcoder:14b     (unchanged)
code_reviewer_model:       rnj-1:8b          (NEW — reviews deepcoder output)
logic_specialist_model:    deepseek-r1:8b    (NEW — math/logic/research)
embedding_model:           nomic-embed-text-v2-moe  (unchanged)
```

**New seats to add to the website:**
- **Code Reviewer** (rnj-1:8b): Sequential relay — deepcoder:14b generates code, rnj-1:8b reviews/profiles it. This is a unique two-pass architecture.
- **Logic Specialist** (deepseek-r1:8b): Deep reasoning for math, programming logic, and research tasks. 300s timeout.

**Seats to update:**
- Renderer: llama3.2:1b/3b -> gemma4:e2b/e4b (Google Gemma 4, multimodal, 128K context)
- Vision Lite: qwen2.5vl:3b -> gemma4:e2b (shared with renderer for VRAM efficiency)

### 2. Comparison Page (Comparison.tsx)

The comparison table may be outdated. The current competitive positioning:

| vs | Jarvis wins on | They win on |
|----|---------------|-------------|
| Claude CLI | Hallucination control, memory, self-improvement, voice, cost ($0), privacy | Raw intelligence, speed, breadth of knowledge |
| ChatGPT | Evidence-anchored rendering, deterministic routing (100% for known intents), $0 cost | Model quality, plugins ecosystem, web access |
| AutoGPT | Reliability, cost, hallucination control, self-improvement, voice | Autonomous multi-step tasks, community |
| Base Ollama | Everything (structured pipeline, 50+ intent handlers, memory, tools, CRSIS) | Simplicity |

### 3. Numbers / Stats (Numbers.tsx)

Update any hardcoded stats to current values:
- **Total LOC**: ~34,000 (jarvis package) / ~46,000 (including tests+scripts)
- **Test files**: 107 files, ~10,500 LOC
- **Python modules**: 171 files
- **Model seats**: 10 (up from 8 — added code_reviewer + logic_specialist)
- **Tools**: 40+
- **Deterministic intent handlers**: 50+
- **VRAM budget**: 6GB (RTX 4050)
- **Cost per turn**: $0

### 4. Architecture / How It Works (HowItWorks.tsx)

The BG1 code specialist now has a **two-pass sequential relay**:
```
User request -> deepcoder:14b (generate) -> rnj-1:8b (review/profile) -> output
```
If the reviewer fails, the system gracefully falls back to the initial generation.

Also, general research tasks now use deepseek-r1:8b instead of llama3.2:3b.

### 5. CRSIS Section (Crsis.tsx)

CRSIS is fully operational with:
- Intent miss detection
- Misrouting detection
- Tool result analysis
- LibCST-based code modification (AST-safe)
- Automatic rollback on validation failure
- Versioned rule snapshots

### 6. Cognitive Layer / World Model Pages

These should be accurate but verify against:
- `jarvis/world_model/evidence_store.py` — provenance chains, corroboration/contradiction
- `jarvis/world_model/judge.py` — claim tagging: observed/recalled/inferred/guessed
- `jarvis/world_model/belief_state.py` — explicit uncertainty tracking
- `jarvis/world_model/state.py` — frozen-immutable WorldState

---

## Architecture Reference (for accuracy checking)

### Core Pipeline (per turn)
```
Speech/Text Input
  -> IngressHub -> IngressNormalizer (clean text, assign speaker profile)
  -> PromptDispatcher.route() — 5-stage cascade:
      1. Exact match (hardcoded phrases)
      2. Strict deterministic (signal-based: asks_time, asks_cancel, etc.)
      3. Soft deterministic (greetings, identity, screen, codebase queries)
      4. Semantic match (embedding model, skipped if >500ms elapsed)
      5. Classifier fallback (keyword heavy/light)
  -> Lane execution:
      REALTIME: IntentHandlerRegistry (50+ handlers) or LLM chat (gemma4:e2b)
      BG1: Specialist chain:
        - Code: deepcoder:14b -> rnj-1:8b review relay
        - Vision: qwen3-vl:8b
        - Research: deepseek-r1:8b (300s timeout)
        - Web: fetch + summarize via renderer fallback
  -> ResponseCompiler + CLLMRenderer (evidence-anchored, hedge detection)
  -> OutputCoordinator -> TTS (Kokoro ONNX) or text log
```

### Hallucination Control (unique strength)
1. **Fact-anchored rendering**: LLM only sees verified evidence in its prompt
2. **Output gating**: Detects hedge markers ("I think", "probably", "as an AI") and rejects
3. **Claim tagging**: Judge classifies every claim as observed/recalled/inferred/guessed
4. **Deterministic-first**: 70-85% of turns never touch the LLM at all

### CRSIS Self-Improvement Loop
```
Observe (intent misses, misrouting, low satisfaction)
  -> Analyze (pattern detection across recent turns)
  -> Propose (phrase/threshold/synonym changes)
  -> Apply (LibCST AST modification of own Python code)
  -> Validate (import check + test run)
  -> Rollback on failure
```

### Memory Architecture
- **PocketMemory**: SQLite entity/slot/link graph with canonical vs dynamic protection
- **Flat memory**: Key-value facts with confidence scores
- **Semantic search**: nomic-embed-text-v2-moe embeddings with lexical fallback
- **Namespaces**: hot_working (volatile), session (per-session), user (persistent), task_result (BG1 outputs)

### Permission Model
Three profiles: `owner` (full access), `trusted` (no file_write/code_runner/app_ops), `guest` (read-only, no heavy tasks)

---

## Duplicate Class Situation (Merger Artifact)

The Esoteric + CLLM merger left 9 duplicate class definitions. These have been documented with canonicality markers but not yet consolidated. If you encounter import confusion:

| Class | CANONICAL location | Legacy copy |
|-------|-------------------|-------------|
| TurnArtifact | brain_core/contracts.py (30 fields) | brain_core/turn_artifact.py (42 fields) |
| EvidencePacket | brain_core/evidence_packet.py | brain_core/contracts.py |
| JobStatus | world_model/task.py | brain_core/contracts.py |
| TaskDecision | brain_core/prompt_dispatcher.py (7 fields) | brain_core/task_classifier.py (2 fields) |
| ChangeResult | crsis/contracts.py | crsis/code_modifier.py |
| EventRecord | observability/events.py (turn events) | world_model/state_builder.py (state events, different concept) |

**Rule**: The pipeline (`turn_pipeline.py`, `runtime_v2.py`) uses `contracts.py` TurnArtifact. The `brain_core/__init__.py` also exports from `contracts.py`. Always check which version you're importing.

---

## Key Files to Read First

If you need to understand the system before updating the site:

1. `jarvis/config.py` — All model seats and system parameters (121 lines)
2. `jarvis/brain_core/prompt_dispatcher.py` — 5-stage routing cascade (~615 lines)
3. `jarvis/brain_core/contracts.py` — Core data contracts (~225 lines)
4. `jarvis/brain_core/bg1_manager.py` — Background task execution + specialist routing (~380 lines)
5. `jarvis/models/code.py` — Two-pass code relay (deepcoder -> rnj-1) (~293 lines)
6. `jarvis/brain_core/cllm_renderer.py` — Fact-anchored rendering with output gating (~442 lines)
7. `jarvis/world_model/judge.py` — Claim tagging system (~238 lines)
8. `jarvis/crsis/engine.py` — CRSIS self-improvement rule engine (~220 lines)

---

## Conventions

- **Python**: 3.12, type hints throughout, `from __future__ import annotations` in every file
- **Dataclasses**: frozen where possible, mutable only for artifacts that accumulate data during a turn
- **Testing**: All tests mock TTS and Ollama — the machine crashes on heavy test runs
- **Config override**: Any `JarvisConfig` field can be overridden via `JARVIS_{FIELD_NAME}` env var
- **Website**: Next.js 15, App Router, CSS Modules, static export, GSAP animations, Three.js hero scene
- **No cloud dependencies**: Everything runs local. The website is the only thing that gets deployed.

---

## What NOT to Change

- Do not modify `jarvis/` Python source to update the website — only read it for accuracy
- Do not remove the legacy duplicate classes — they have canonical markers and will be consolidated separately
- Do not change the static export config (`output: "export"`) — the site deploys as static HTML
- Do not upgrade Next.js or major dependencies without asking — the 3D scene is sensitive to Three.js versions

---

## Useful Commands

```bash
# Check current model lineup
cat jarvis/config.py | grep -E "_model|_specialist"

# Run Jarvis text chat (verify behavior)
cd /home/billybart/Downloads/Harold/esotericv0.2CURRENT/Harold/Jarvis/Jarviscore
python3 run_chat.py

# Build the website
cd "/home/billybart/Downloads/Harold/site/New folder"
npm run build

# Dev server
npm run dev
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bibbisalsd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
