## explainlab

> Animated Bitcoin educational explainers using React. Recorded to MP4 via Playwright + FFmpeg, with automated visual QA.

# Bitcoin Error Explainer — Animated Video Series

Animated Bitcoin educational explainers using React. Recorded to MP4 via Playwright + FFmpeg, with automated visual QA.

## Workspace
- `client/src/episodes/` — each episode folder has `VideoTemplate.tsx` + custom components
- `client/src/lib/video/` — shared hooks (`useVideoPlayer`), canvas primitives (`CE`, `morph`, `sceneRange`), `DevControls`, animation presets, diagram components
- `scripts/` — auto-episode pipeline (`auto-episode.sh`), **MP4 export (`record.mjs`)**, visual QA (`visual-qa.mjs`), scene screenshots (`screenshot-scenes.mjs`)
- `client/public/audio/` — scene voiceover MP3s
- `references/` — brand guidelines, writing style references

## Role-Specific Guides

This project splits context by agent role to reduce token cost and keep each agent focused:

- **`CLAUDE-build.md`** — Animation toolkit, GSAP utilities, VideoTemplate patterns, episode architecture. For build-phase agents only.
- **`CLAUDE-critic.md`** — Quality bar, sameness checklist, visual distinction rules, episode registry. For critique-phase agents only.
- **`CLAUDE-research.md`** — Teaching approaches, content checklist, tone/voice. For research-phase agents only.

Build agents: read `CLAUDE-build.md`. Critics: read `CLAUDE-critic.md`. Research agents: read `CLAUDE-research.md`. All agents should read this core file.

## Automated Visual QA (`scripts/visual-qa.mjs`)

**Run after building any episode.** Opens the episode in Playwright at 1920×1080, steps through every scene, checks element positions with `getBoundingClientRect()`.

```bash
node scripts/visual-qa.mjs ep11 ./visual-qa-output
```

Reports: **FAIL** (off-screen near-miss), **WARN** (clipped >40% or far off-screen elements). Generates screenshots + markdown report.

**Do NOT write manual POSITION AUDIT comments** — the automated tool replaces manual math audits.

## Scene Rules
- **One idea per scene.** One concept, one step, one point.
- **Scene 1 = Title.** Scene 2 = start from familiar ground — don't open with jargon. A cold open on "Merkle proofs" loses people instantly. Instead: "A block bundles transactions" → "How do you prove yours is inside?" The hook emerges from the progression, not from forcing a dramatic opener. Only use a punchy hook-first opening when the topic is already familiar to the audience.
- **Last scene = CTA** ("Follow @bitcoin_devs") + optional series teaser for next episode.
- **Use as many scenes as needed.** More scenes with less content each > fewer dense scenes.
- **Motivation before mechanism.** Before any scene showing HOW something works (a formula, algorithm, technique), there MUST be a preceding scene establishing WHY — the problem it solves or the question it answers. A viewer should never think "why are we doing this?" If you're about to show math, first show the problem the math solves. Example: don't jump to `7^n mod 15` — first show that "multiplying is easy, un-multiplying is hard, and Bitcoin's security depends on that gap."

## Text Rules
Text and visuals work TOGETHER. The best scenes have **text integrated into the visual** — labels on diagrams, values inside blocks, formulas next to the thing they describe. Think 3Blue1Brown: equations animate alongside the geometry, labels point to the thing they name.

Rules:
- **Visual leads, text clarifies.** Every scene that teaches a concept needs a **real animated visual that demonstrates the mechanism** plus at least one teaching anchor (label, value, caption) that clarifies what the viewer is seeing. The visual does the heavy lifting — text supports it. If a scene is just text panels with entrance animations, the visual isn't leading. Pure unlabeled animation is only OK for title cards, mood beats, or very short transitions.
- **No paragraphs on screen.** 3+ sentences = split across scenes.
- **Use real values.** "bitcoin" → `01100010...` beats "the input gets converted to binary."
- **Progressive reveal.** Each scene adds ONE piece. Like a conversation, not a lecture.
- **Breathing room.** Whitespace is content. Let animations breathe.
- **Questions and quizzes are powerful.** Use when needed, recomended to keep the user hooked.

## Tone & Voice
- Casual-educational, peer-to-peer. ELI5 ethos on deep topics.
- Direct address: "Let's see...", "Now let's look inside..."
- Conversational pacing. Never academic or stiff.
- **Never force analogies.** Only when they map naturally and illuminate the concept.

## Visual Identity

### Color Palette Modes (`--palette` flag)

The `--palette` flag on `auto-episode.sh` controls color constraints:
- **`grayscale`** — black, white, grays only. One accent color allowed for emphasis. Stark, data-focused look.
- **`brand`** — BDP brand palette only (see `references/brand-guidelines.md`). Orange, blue, green, pink, purple + neutrals.
- **`free`** (default) — no restrictions. Pick whatever serves the mood.

Every episode defines its palette in `EP_COLORS` in `constants.ts`. The `--palette` flag guides what goes in it.

### Everything Else Must Vary Per Episode

**Each episode defines its own palette** in `constants.ts`:
```ts
export const EP_COLORS = {
  bg: '#0F172A',          // dark slate — security/attack mood
  bgAlt: '#1E293B',
  accent: '#EF4444',      // danger red
  accentAlt: '#F97316',   // warning orange
  highlight: '#FDE68A',   // gold for key reveals
  muted: '#64748B',
  text: '#F1F5F9',
};
```

**Each episode defines its own motion personality** in `constants.ts`:
```ts
export const EP_SPRINGS = {
  enter: { type: 'spring', stiffness: 600, damping: 18 },  // aggressive snap-in
  morph: { type: 'spring', stiffness: 80, damping: 30 },   // slow deliberate morph
  impact: { duration: 0.1, ease: 'easeOut' },              // instant for collisions
};
```

### Motion Personality Per Episode
- **Aggressive/security topics:** `stiffness: 500+`, `damping: 15-20` — snappy, sudden, tense
- **Mathematical/crypto topics:** `stiffness: 100-150`, `damping: 25-30` — slow, precise, deliberate
- **Network/distributed topics:** `stiffness: 200-300`, `damping: 20` — flowing, wave-like, organic
- **Step-by-step processes:** mix fast setup moves (`stiffness: 400`) with slow key reveals (`stiffness: 80`)


## Engagement Techniques

### Key Insight "Highlight Scene"
One scene per episode should visually break the pattern to signal the core takeaway. Different background color, larger text, dramatic animation. Mark with `{/* HIGHLIGHT SCENE */}`.

### "Why Is This a Big Deal?" Beat
After teaching the mechanism, dedicate a scene to frame the significance.

### Cascade / Domino Consequence
Animate downstream breakage in sequence when a system has dependencies.

### Show Running State
During multi-step transformations, show full data with current step highlighted/boxed.

### Scale Comparison — Make Big Numbers Real
Decompose incomprehensible numbers into tangible comparisons.

### Copy-Move for Connections
To show two things are the same, related, or colliding: animate a copy of element A to sit next to element B. The motion itself creates the connection — more powerful than labels or arrows alone. Technique: `copy-move`. Role: `connect`.

### Generalization Through Sweep
Instead of showing 3 examples side by side, show 1 example that dynamically morphs through variations. The viewer sees the motion and understands "this works generally." Technique: `sweep`. Role: `generalize`.

### Covariation — Two Things in Tandem
When teaching an input→output relationship, animate both sides simultaneously. Change the input, watch the output change in real-time. The synchronized motion communicates dependency faster than any label. Technique: `linked-vary`. Role: `covary`.

## Teaching Approaches

an episde can combine also mutliple approach, this is just an exmaple if you find better approach feel free to do it, don't take this religiously
1. **Problem > Failure > Fix Loop** — build naive system, show how it breaks, fix. Best for protocol design.
2. **Specific > General** — concrete example first, then abstract rule.
3. **Analogy-First** — only when analogy fits naturally.
4. **Definition-Deep-Dive** — define, then layer complexity scene by scene.
5. **Wrong > Less Wrong > Right** — start wrong, refine toward correct.
6. **Dialogue-Driven** — Alice & Bob discuss conversationally. Use `Character` component.

### Emotional Arc
Curiosity > Confusion > Partial clarity > **Aha moment** > Satisfaction.

## Scene Composition — No Single-Element Episodes
- **Different scenes = different visual elements.**
- **Build, climax, clear, rebuild.** Visual "chapter breaks."
- **All content fits within the 1920×1080 viewport.** No off-screen elements.
- **Rule of thumb:** No single component visible for more than ~40% of the episode.
- **Each act gets its own visual centerpiece.**

### Didactic Role Per Scene
Every scene should serve one primary teaching role (the *why*) and use a named animation technique (the *how*). These are separate vocabularies:

**Roles:** `connect` (link two representations) · `covary` (show dependency — change input, output changes) · `visualize_structure` (reveal a concept's shape) · `visualize_process` (step through a procedure) · `symbol_sense` (make encodings/formulas intuitive through motion) · `ground_in_reality` (connect to real-world scenario) · `generalize` (show it works beyond the single example)

**Techniques:** `copy-move` · `morph` · `trace` · `rule-based-move` · `scale-vary` · `rearrange` · `decompose` · `highlight-morph` · `sweep` · `linked-vary`

## Content Checklist
- Pick a topic people have heard of but don't really understand
- **Define the episode's visual concept** — What accent colors? What layout within the viewport?
- **Build at least one custom component** for the episode's core visual
- Target the emotional arc: Curiosity > Confusion > Partial clarity > Aha > Satisfaction
- Find a natural analogy (or skip it if none fits)
- Open from what the viewer already knows
- **Every scene should have animated visuals** — text captions the animation, not the other way around
- Use a real worked example with actual values when possible
- Progressive reveal — staggered delays, never dump everything at once
- **Vary motion style** — don't use identical spring configs for every episode
- End with CTA on last scene
- Use as many scenes as needed


**What a new episode MUST do differently:**
- Use `createThemedCE(ceThemes.xxx)` — NEVER bare CE with default fade-up
- **Viewport-first layout** — all content within 1920×1080, no oversized canvases
- Use `useSceneGSAP` for at least one choreographed sequence
- Define custom EP_COLORS and EP_SPRINGS in constants.ts
- Core visual must NOT use CE — use morph(), GSAP, SVG morph, or canvas
- Background must follow the `--palette` mode
- **Multiple distinct visual compositions** — build, climax, clear, rebuild
- others (don't limit yourself to what i said above)

## Autonomous Pipeline

`./scripts/auto-episode.sh <topic> <ep_number> <slug> [--with-voice] [--full-auto]`

Planner (can't edit code) reviews and steers. Executor (can edit code) builds. Handoff via `.auto-episode/ep<N>-<slug>/` artifacts. Pipeline: Research → Merge → Creative Spec → Build → Critique Loop → Voiceover. See `scripts/auto-episode.sh` for full details.

## Build Memory

Reusable lessons from past episodes live in `.auto-episode/build-memory.md`. This is a short, curated file — NOT a growing log. Stable patterns get promoted into this CLAUDE.md or the role-specific guides. The append-only episode history lives separately in `.auto-episode/episode-history.md`.

---
> Source: [bitcoin-dev-project/ExplainLab](https://github.com/bitcoin-dev-project/ExplainLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
