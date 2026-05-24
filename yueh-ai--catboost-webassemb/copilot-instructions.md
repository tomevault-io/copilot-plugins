## catboost-webassemb

> - `plan_v1.md` - Original project plan with 5 core phases + 2 stretch goals (phases 6-7)

# CatBoost WebAssembly Demo

## Developer Documentation

- `plan_v1.md` - Original project plan with 5 core phases + 2 stretch goals (phases 6-7)
- `PROGRESS.md` - Current progress tracking and implementation status (tracks 5 core phases)

**IMPORTANT: Documentation hierarchy**

- When conflicts arise between documents, follow this precedence: CLAUDE.md (highest) → plan_v1.md → PROGRESS.md (lowest)
- CLAUDE.md contains the most up-to-date implementation guidance
- PROGRESS.md is a living document meant to be updated as work progresses

## Bash Commands

### Python workspace

```bash
cd python/
uv sync                    # Install/update dependencies
uv run train.py           # Train CatBoost model
uv run validate.py        # Validate Python vs WASM predictions
```

### WASM workspace

```bash
cd wasm/
source ~/emsdk/emsdk_env.sh  # Activate Emscripten
./build.sh                    # Build WASM module (output: wasm/build/)
```

### Web application

```bash
cd webapp/
pnpm install              # Install dependencies
pnpm run dev             # Start dev server (http://localhost:5173)
pnpm run build           # Production build
pnpm run typecheck       # Run TypeScript type checking
```

## Core Files

- `python/train.py` - Model training with UCI Adult Income dataset
- `python/validate.py` - Compare Python vs WASM predictions
- `wasm/catboost_wrapper.cpp` - C++ wrapper for CatBoost inference
- `webapp/src/CatBoostWorker.ts` - Web Worker for WASM model execution
- `webapp/src/model_data.ts` - Model and categorical mappings

## Code Style

- Use TypeScript strict mode in webapp
- Follow existing import patterns (ES modules)
- Prefer async/await over callbacks
- Handle errors explicitly
- NEVER commit model files (.cbm) to git
- Keep concurrency simple: use async/await and event loops instead of threads or processes

## Documentation Guidelines

- Write in clear, simple English (project maintainer is non-native speaker)
- Avoid complex sentences and idioms
- Be explicit rather than implicit
- Use examples when explaining concepts

## Testing

IMPORTANT: Always run typecheck after making changes to TypeScript code:

```bash
cd webapp && pnpm run typecheck
```

## Environment Setup

- Python 3.10+ with uv package manager
- Node.js 18+ with pnpm
- Emscripten 3.1.51 for WASM builds
- Modern browser (Chrome 90+, Firefox 88+)

---
> Source: [yueh-ai/catboost-webassemb](https://github.com/yueh-ai/catboost-webassemb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
