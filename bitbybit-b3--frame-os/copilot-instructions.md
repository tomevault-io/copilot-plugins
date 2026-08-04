## frame-os

> > instructions for any AI coding agent (cursor, claude code, openai codex, copilot, gemini cli, whatever) working in this repo. **read this BEFORE generating any code.** if there's a conflict between this file and your default behavior, this file wins.

# AGENTS.md

> instructions for any AI coding agent (cursor, claude code, openai codex, copilot, gemini cli, whatever) working in this repo. **read this BEFORE generating any code.** if there's a conflict between this file and your default behavior, this file wins.

---

## what we're building

**FrameOS** — a desktop-native, chat-first AI video editor. tauri 2 rust desktop talking to a python fastapi backend that runs in k8s (kind locally). all AI is gemini's media stack: gemini 2.5/3.x + veo 3.1 + imagen 4 + nano banana + lyria 3.

killer feature: **auto-edit on upload** — drop raw footage, the backend automatically fires three jobs in parallel (groq whisper transcript, ffmpeg + gemma 3 frame captions into qdrant, gemini 2.5 pro multimodal timeline build). asset is gated until the timeline is ready; one click and the editable JSON timeline materializes in ~45-60 seconds. refine with chat tool calls or mouse, export MP4.

bonus reveal: **crew mode** — toggle the Crew button in the chat header to flip from single-agent chat to a multi-agent pipeline. a planner agent decomposes the user goal, then dispatches to specialists (editor for timeline mutations, captioner for copy). each agent streams its own thinking deltas live as a separate card. backed by `/crew` SSE endpoint + `apps/backend/app/core/crew.py`. crewai-style orchestration without pulling the heavy pypi lib (would violate the gemini-only model-calls rule).

---

## read THESE before writing code

read in this order. they're load-bearing.

1. **`README.md`** — the user's vision in their own voice. constantly evolving. **re-read every time you start a task.**
2. **`docs/PLAN.md`** — shipped-state snapshot + what's left. start here on a fresh checkout.
3. **`docs/DEMO.md`** — recorded-walkthrough script + judging hooks + crew mode reveal in step 5.5 + "do not click X" list.
4. **`docs/DESIGN.md`** — the spec. architecture, feature tiers, auto-edit pipeline, demo script, 20-hr timebox. canonical source for "what we're building" — note: the original spec has an Auto-Edit button but that's been killed; pipeline auto-fires on upload now.
5. **`docs/NOT-DOING.md`** — explicit cuts. if you're tempted to add something, check here first. if your idea is in this file, don't build it without asking the human.
6. **`docs/PRODUCT-DESIGN.md`** — UI / UX brief. design language, visual identity, component inventory, interaction patterns. read this if you're touching the frontend or generating mockups. structured so it can be pasted into claude.ai/design / v0 / figma make / lovable directly.

---

## the stack (locked)

| layer | tech |
|---|---|
| desktop shell | **tauri 2** (rust) |
| desktop UI | vite + react + typescript + tailwind + shadcn/ui + zustand + **remotion player** |
| desktop chat | vercel ai sdk v5 with custom transport hitting backend `/chat` |
| backend service | **python 3.12 + fastapi + pydantic v2 + google-genai** |
| video processing | ffmpeg-python (server-side, in the backend container) |
| storage | **SeaweedFS** in-cluster, S3-compatible (minio CE was archived apr 2026 — seaweedfs is the production successor used by Kubeflow Pipelines) |
| local orchestration | **kind** + **skaffold** + plain k8s yaml (no helm) |
| ci | github actions; `tauri-apps/tauri-action` for cross-platform builds |
| external APIs | gemini 2.5/3.x · veo 3.1 fast · imagen 4 fast · nano banana · lyria 3 realtime · groq whisper · pexels videos |

if you want to swap any of these, **ask the human first**. do not silently introduce a new dependency.

---

## one-time local setup

```bash
# prerequisites (macOS — adapt for linux)
brew install kind kubectl skaffold docker rustup uv pnpm
rustup default stable
docker login   # if your kind config pulls private images

# clone + enter
git clone <this repo>
cd frameos

# spin up the cluster
kind create cluster --name frameos --config infra/kind-config.yaml
kubectl create namespace frameos

# load secrets (replace with your real keys)
kubectl -n frameos create secret generic api-keys \
  --from-literal=GEMINI_API_KEY=$GEMINI_API_KEY \
  --from-literal=GROQ_API_KEY=$GROQ_API_KEY \
  --from-literal=PEXELS_API_KEY=$PEXELS_API_KEY

# apply manifests
kubectl apply -f infra/k8s/

# install backend + desktop deps
cd apps/backend && uv sync && cd ../..
cd apps/desktop && pnpm install && cd ../..
```

---

## day-to-day commands

```bash
# terminal 1 — backend live-reload in the cluster
cd infra && skaffold dev

# terminal 2 — desktop app
cd apps/desktop && pnpm tauri dev

# terminal 3 — tail backend logs
kubectl -n frameos logs -f deploy/backend

# run tests
cd apps/backend && uv run pytest -x                     # python
cd apps/desktop && pnpm exec playwright test            # e2e (chromium against vite)
cd apps/desktop/src-tauri && cargo test                 # rust

# lint + format
cd apps/backend && uv run ruff check . && uv run ruff format .
cd apps/desktop/src-tauri && cargo fmt && cargo clippy -- -D warnings

# typecheck
cd apps/desktop && pnpm exec tsc --noEmit
```

---

## coding standards — Google style guides

### non-negotiable cross-language rules

- **zero comments.** code is self-explanatory through naming, type signatures, and structure. no module docstrings, function docstrings, inline `#` / `//` / `/* */` / `/** */`, JSDoc, Python docstrings, Rust `///` doc comments. shebangs in shell scripts and `$schema` keys in JSON are exempt. applies to python, ts, rust, sh, yaml, dockerfile, BUILD.bazel. if you want to comment, rename the function or extract a helper.
- **zero emojis** anywhere in the codebase — source files, UI strings, shell output, commit messages, PR titles, branch names, file names. use lucide-react icons for the frontend; plain text labels everywhere else.

these override anything else in this file. if a snippet below shows a comment or emoji, treat that as a doc-tool artifact and don't reproduce it in actual code.

### per-language baseline

we baseline on **Google's style guides** for python, typescript, and shell. tools enforce them. rust uses the standard rustfmt + clippy because google doesn't publish a rust guide.

### python (backend)

- **style:** [google python style guide](https://google.github.io/styleguide/pyguide.html)
- **formatter:** `ruff format` (line length 100 — readability wins over google's 80 on modern monitors)
- **linter:** `ruff check` with rules that mirror the google guide
- **types:** required on all public functions. pydantic v2 models for every JSON-shaped thing
- **imports:** ruff's isort: stdlib → third-party → first-party, sorted
- **docstrings:** google style — `Args:`, `Returns:`, `Raises:` blocks
- **logging:** structured via `structlog`. **no `print()` in production code.**
- **async:** I/O is async by default. use `httpx.AsyncClient`, never blocking `requests`
- **errors:** raise typed exceptions (`AssetNotFoundError`, `GeminiError`). map to HTTP at the route layer only

```python
async def analyze_video(
    asset_id: str,
    goal: str,
    *,
    model: str = "gemini-2.5-pro",
) -> Analysis:
    """Run multimodal analysis on an uploaded asset.

    Args:
        asset_id: SeaweedFS key for the source video.
        goal: Natural-language editing goal from the user.
        model: Gemini model override (defaults to 2.5 Pro).

    Returns:
        Structured analysis with scored segments, retention curve,
        heatmap, chapters, and b-roll keywords.

    Raises:
        AssetNotFoundError: if asset_id is unknown.
        GeminiError: if the API call fails after retries.
    """
    ...
```

### typescript (desktop webview)

- **style:** [google typescript style guide](https://google.github.io/styleguide/tsguide.html)
- **formatter + linter:** `biome` (one tool, faster than prettier+eslint)
- **types:** `tsc --strict`. **no `any` ever** — use `unknown` + narrowing
- **naming:** lowerCamelCase for variables/functions, UpperCamelCase for types/classes/components
- **mutability:** `const` always. `let` only when actually mutated
- **exports:** named exports for components (no `export default`)
- **imports:** `@/` absolute alias for `src/`, sorted by biome
- **fp over loops:** prefer `.map / .filter / .reduce` when the intent is transformation
- **react:** no `useEffect` for derived state — compute it inline or memoize

```tsx
import { useEditorStore } from '@/stores/editor-store';

export function TimelineClip({ clipId }: { clipId: string }) {
  const clip = useEditorStore((s) =>
    s.timeline.clips.find((c) => c.id === clipId),
  );
  if (!clip) return null;
  return <div data-clip-id={clipId}>{clip.label}</div>;
}
```

### rust (tauri core)

- **style:** rust standard (no google guide for rust). rustfmt + clippy enforce
- **formatter:** `cargo fmt`
- **linter:** `cargo clippy -- -D warnings` — clippy warnings are build errors
- **errors:** `anyhow::Result` at the boundary, `thiserror` for typed library errors
- **never `unwrap()` in production paths** — use `?` with context: `.context("opening project file")?`
- **async runtime:** tokio (tauri 2 default)
- **`unsafe`:** prohibited without a `// SAFETY:` comment explaining the invariant being upheld

### shell scripts

- **style:** [google shell style guide](https://google.github.io/styleguide/shellguide.html)
- always start with `set -euo pipefail`
- `shellcheck` must pass clean
- `shfmt -i 2` for formatting
- bash, not sh, unless we need posix portability

---

## testing

minimum bar for the hackathon (lower than our long-term 80% target — this is realistic):

- **unit tests** for every public function in `apps/backend/app/core/*`
- **integration tests** for every backend endpoint (fastapi TestClient + a real local seaweedfs container)
- **one e2e smoke test** for the auto-edit flow against a tiny pre-bundled sample video
- coverage target: **70% backend, 50% frontend**

frameworks:
- python: `pytest` + `pytest-asyncio` + `httpx` test client
- typescript: `vitest` + `@testing-library/react`
- rust: built-in `#[test]` + `cargo test`

testing rules:
- never mock gemini at the unit-test boundary for the auto-edit pipeline — record a real response and replay it (use `vcr.py` or hand-built fixtures)
- always test with the actual JSON timeline schema, not a simplified mock — schema drift is the bug we'll hit at hour 18

---

## git workflow

- branch from `main`
- one feature per PR (no megacommits)
- **commits: conventional commits** — `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`, `perf:`, `ci:`
- subject line ≤72 chars, imperative mood, lowercase
- body explains **why**, not what (the diff shows what)
- **no co-author trailers** unless explicitly set up
- **no `--no-verify`** to skip hooks. fix the underlying issue.

```
feat(backend): wire auto-edit pipeline end-to-end

routes /auto-edit through gemini 2.5 pro for analysis, then 2.5 flash
for timeline assembly. emits job_ids for async background tasks so the
client can render the timeline immediately and stream b-roll + music
in as they finish.
```

---

## things AI agents commonly get wrong in this repo

- **adding features not in docs/DESIGN.md tier 1/2/3** — check `docs/NOT-DOING.md` first
- **adding new dependencies without justification** — every npm/pypi/crate add is a build-time cost. justify in the PR
- **writing helm charts** — plain k8s yaml is locked. no helm
- **using openai / claude / mistral APIs for model calls** — gemini is the sponsor. all model calls route through `apps/backend/app/core/llm/providers.py` (gemini cloud + local ollama both supported via `X-LLM-Provider` request header)
- **bypassing the backend for gemini calls** — desktop never sees the API key
- **adding auth** — single-user local mode this hackathon. see docs/NOT-DOING.md
- **migrating to a database** — zustand on the client, in-process dict on the server. db is post-hackathon
- **shipping without re-reading `README.md`** — the user keeps adding ideas

---

## the user's voice

casual lowercase. "sorta", "ykwim", "shit like that", "man thanks". when you write user-facing copy, docs, commit messages, or PR descriptions — **match this voice**. don't be precious. `README.md` and `docs/NOT-DOING.md` are the reference.

precise commands and code stay precise. casual voice is for prose.

---

## when in doubt

1. re-read `README.md`
2. check `docs/DESIGN.md` for the canonical decision
3. check `docs/NOT-DOING.md` for explicit cuts
4. if still unclear — **ask the human before generating code**. an unanswered question costs less than a wrong implementation

---
> Source: [BitByBit-B3/frame-os](https://github.com/BitByBit-B3/frame-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
