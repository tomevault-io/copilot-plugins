## studio-video-creator

> >


# Studio Video Creator

This skill produces a single, well-defined artifact: a **60–90 second concept film**. Voiceover-driven, prototype-grounded, insight-led, aspirational close.

The film is used at the moment a studio concept becomes legible — post-research, pre-build — to frame the idea for an internal team, an LP, a launch audience, or the founder themselves. It is not marketing collateral. It is a thinking artifact rendered as video.

## Philosophy: Guide like software, not chat

The user wants a tool, not a conversation. Behave accordingly:

- **Decide silently where you can.** Variant selection, structural choices, design defaults — make the call, then name it briefly. Do not present menus when you can present a recommendation.
- **Ask only what you genuinely need.** Hard cap: three questions in the entire intake phase. Each must change the output materially. If a question can be answered from the source material, do not ask it.
- **Gate every phase.** Do not advance to script without a legible concept summary. Do not render without a chosen voice. State the gate; don't quiz the user about it.
- **Show progress.** Each phase ends with a short status line ("Concept locked → moving to script") so the user always knows where they are.
- **Withhold options.** Curation is the value-add. The voice list is 8, not 100. The framework variants are 3, not 12. If the user wants more, they ask.
- **Recover gracefully.** Missing prototype? Suggest the minimum surface needed. Vague brief? Restate what you can extract and ask the user to confirm or correct.

## Execution context — interactive sessions only

This skill requires an interactive session where the user can respond between phases. The workflow is a series of human-judgment gates; without the human, the output can look complete before the important decisions have actually been judged.

- If running inside an autonomous task, batch prompt, single-shot delegation, scheduled job, or any context where the user cannot provide input between phases: **STOP after Phase 1**, present the brief, and tell the user to run the skill in an interactive Claude Code session to continue.
- The skill's value is in its gated process. Skipping gates usually produces output that appears finished before the concept, design, voice, or hook has been properly reviewed.
- Do not delegate this skill to an autonomous agent. Do not pre-answer the gates in a single prompt. Each gate exists because the downstream phase depends on human judgment at that gate.

## When to use this skill

**Use when:**
- The user has source material (brief, research, prototype, deck, URL, PDF) for a product concept or studio project AND wants a video
- The user is at the "this needs to become legible" moment in a concept's life
- The video's purpose is framing/pitching/introducing — not marketing or tutorial
- The target length is short (60–120s)

**Do not use when:**
- The user wants a long-form video, tutorial, course, ad creative, social post, talking-head, or recorded demo
- The user has no source material and just wants to brainstorm — that's a different mode
- The user explicitly wants something other than the concept-film genre

If unsure, ask one disambiguating question and let the user steer.

## What you produce

By the end of a complete run, the user has, in their project directory:

```
<project>/
├── brief.md           — interpreted concept, audience, vision statement
├── script.md          — final script with section timing
├── motion-board.md    — beat-by-beat visual causality plan
├── design.md          — chosen visual/motion language, brand interpretation, cover strategy
├── voice.json         — selected ElevenLabs voice ID + audition notes
├── renderer.json      — selected local renderer (`remotion` by default, `hyperframes` when warranted)
├── remotion/ or hyperframes/
│                      — renderer project, scenes filled with project content
│   ├── package.json
│   ├── data/ or src/
│   └── public/
└── out/
    ├── design-thumbnail.png — Phase 4 title-frame / style-frame artifact
    ├── cover-frame.png — actual frame-0 poster / preview image from the hook
    ├── hook.mp4       — first 10–15s rendered (round-one deliverable)
    └── final.mp4      — full film (rendered on explicit request)
```

The design thumbnail is the aesthetic iteration unit. The motion board is the explainer-quality gate. The cover frame is the silent first-impression check. The hook render is the film iteration unit. The full render is the publication unit.

---

## Workflow

The workflow has seven numbered phases plus a Phase 3B motion-board gate. Each phase is a gate. Do not skip ahead.

### Phase 1 — Intake & legibility gate

Receive the user's source material. Inputs may include:

- A brief, research doc, or memo (PDF, MD, DOCX, plaintext)
- A prototype reference (Figma link, deployed URL, screenshots)
- A deck (PPTX, PDF, Keynote export)
- A website URL
- A loose description in the user's own words

**Steps:**

1. Read everything provided. Use `web_fetch` for URLs, the file-reading skill conventions for documents, image analysis for screenshots.
2. Produce a **two-sentence concept summary** — what it is and why it matters. Show this to the user.
3. **Gate:** Ask the user to confirm or correct the summary. Do not proceed until the concept is legible in two sentences. If you cannot summarize the concept in two sentences from the inputs, the video will fail. Tell the user this directly and ask for the missing piece.
4. State what minimum prototype surface the video will need ("To do this concept justice, the prototype should show: the home/landing view, the core ontology screen, and the action moment"). If the prototype provided already covers this, say so. If not, flag the gap — the user can either provide it now or accept that the film will use schematic representations.

End Phase 1 with: `Concept locked. → Phase 2: clarifying questions.`

**HARD STOP.** Present the two-sentence summary and wait for user confirmation. Do not proceed to Phase 2 until the user explicitly confirms or corrects. Silence is not confirmation.

Read `references/intake-checklist.md` for the full checklist of what to extract from source materials and how to handle each input type.

### Phase 2 — Three sharp questions (and not one more)

Ask at most three questions. Skip any whose answer is already in the source material.

**The three slots:**

1. **Audience + emotional state.** Who watches this, and what state are they in when the film starts? (Examples: "Skeptical LP who's seen 20 pitches this week." "Internal team that needs to believe.") This determines tone.
2. **The single insight.** If the viewer remembers one thing 10 minutes after watching, what is it? Force the answer into one sentence. This becomes the spine.
3. **The vision statement.** "In a world where… / We believe… / What if…" — one sentence the film can land on. If the user is unsure, propose two options from the source material.

If the source material answers any of these, skip the question and state your inference: "I'm reading the audience as institutional LPs. Correct me if not."

End Phase 2 with: `Brief assembled. → Phase 3: structure and script.`

**HARD STOP.** Present the assembled brief (audience, insight, vision statement) and wait for confirmation.

### Phase 3 — Variant selection and script

You have three structural variants. Pick one silently based on the brief.

**Customer-led** — opens on a person living the problem. Use when the concept is grounded in a human moment (mobile medical care, education, housing, services). The viewer enters through empathy.

**Insight-led** — opens on the idea itself. "What if X." Use when the concept's power is conceptual and the human moment is harder to dramatize (B2B tools, infrastructure, platform plays). The viewer enters through curiosity.

**Demo-led** — opens cold on the product, voiceover catches up. Use when the product is visually striking and self-evident. The viewer enters through "wait, what's that."

Read `references/frameworks.md` for the full structural template, beat-by-beat timing, and worked examples per variant.

**Name your choice in one sentence** ("This wants to be customer-led — opening on a real moment with a clinician making a house call"), then write the script.

**Script writing rules — read `references/script-rules.md` before drafting.** Key points:
- 60–90 seconds at standard VO pace ≈ 150–220 words
- Section structure: Cold Open → Problem → Insight → Product Walk → Vision Close
- Each section has a timing budget
- Voice is warm-confident, never salesy, never breathless
- Concrete > abstract. Specific > generic. Verbs > adjectives.
- The product is named explicitly at least once
- The vision close earns the aspiration through what came before

The Cold Open's first on-screen direction must also include a cover-frame intent: what frame 0 shows before voiceover and motion help. The first frame should be a planned poster image, not a fade-in accident.

Output the script in `script.md` with timing per section and per beat:

```markdown
## Cold Open (0:00–0:08)
[On-screen: ...]
**VO:** ...

## Problem (0:08–0:22)
...
```

End Phase 3 with: `Script drafted. → Phase 3B: motion board.`

**HARD STOP.** Present the full script with on-screen directions and section timing. Wait for the user to approve, edit, or redirect. Do not proceed to design until the script is locked.

### Phase 3B — Motion board and explainer grammar gate

After the script is approved and before visual design, prove that the film explains through motion instead of becoming illustrated voiceover.

Read `references/motion-board.md`, then create `<project>/motion-board.md`.

For every 5-8 second beat, specify:

- Beat/time range
- VO line
- Visual action
- Before state
- After state
- Product proof
- Motion mode (`human-action`, `object-flow`, `product-state-change`, `camera-move`, `kinetic-type`, or `lockup-hold`)
- Scene/component implication
- Static risk

Each beat must contain a visible state change: an object moves, information transforms, a product state changes, a user decides, the system responds, or an output appears. A new sentence appearing on screen is not enough.

Use the board to catch wrong-format translations. A visual reference like a deck, brand system, or screenshot may define taste, but it must not force the whole film into deck grammar. The film still needs cinematic causality.

For product explainers, the Product Walk must be a causal chain, not a feature tour:

```text
input arrives -> system reads -> judgment is made -> output appears -> proof remains
```

If a beat has no action, no before/after state, or no product proof, revise the script's on-screen direction before moving on. Do not proceed if more than one consecutive beat is `kinetic-type` or `lockup-hold`.

End Phase 3B with: `Motion board approved. → Phase 4: design direction and thumbnail.`

**HARD STOP.** Present `motion-board.md` and wait for approval or correction. Do not proceed to design until the user approves the explainer grammar.

### Phase 4 — Design direction and thumbnail

The film has a visual language. You decide it, then show the user as an artifact, not just prose.

Start with a **visual source checkpoint** before attempting design. The source material used for intake may be incomplete, visually weak, or intentionally different from the desired outcome. Briefly state the visual sources you currently have and ask for any design-specific additions:

> "Before I define the look, I'm using [website/Figma/deck/screenshots] as the visual source of truth. If there's a deck, screenshot, brand guide, Figma, reference site, moodboard, past video, or desired aesthetic that should steer the look, add it now. Otherwise say 'use these' and I'll thumbnail the direction."

This is a compact gate, not a new questionnaire. If the user adds material, read it before choosing a direction. If it conflicts with earlier inputs, treat the newest design-specific material as the source of truth and note that choice in one sentence.

Two paths:

**If the brand exists** (logo, deployed site, established palette in the inputs): extract the brand. Identify typography, color, tone of imagery, motion sensibility. Document it in `design.md`.

**If the brand does not exist yet** (early concept): propose two directions. Each direction is a one-paragraph description plus three concrete UI references. Use Mobbin, Refero, or an equivalent UI reference source if available to fetch real inspiration screens. Read `references/design-language.md` for how to query references effectively and how to structure the proposal. Recommend one direction and make the recommendation tangible with a rendered thumbnail.

Either way, end with concrete tokens recorded in `design.md`:

- Primary typeface (with weight choices)
- Display typeface (if different)
- Color palette (background, ink, accent, support)
- Motion principle (e.g., "deliberate slow-in, snap-out; nothing bounces; sub-300ms transitions")
- Imagery direction (photographic, illustrated, screen-cap, kinetic typography mix)

Also define the **cover frame strategy** in `design.md`. Read `references/cover-frame-strategy.md` if the best first frame is not obvious. Choose the archetype (title-card / product-first / human-moment / thesis), describe the actual frame-0 image, specify any on-screen text, and state why it still reads silently at small size.

These tokens flow into the selected renderer template. Remotion is the default and recommended renderer; use HyperFrames when the user asks for an HTML/CSS/GSAP artifact or when that project shape is clearly better. If renderer choice materially matters, read `references/renderers.md`.

**Thumbnail artifact:**

1. Run `scripts/init-project.sh <project>` if the project has not already been initialized.
2. Write the chosen tokens to the selected renderer's token contract:
   - HyperFrames: `<project>/hyperframes/data/tokens.json`
   - Remotion: `<project>/remotion/src/compositions/shared/BrandTokens.ts`
3. Add the cover frame strategy to `design.md`.
4. Write or update placeholder film data with the project name, chosen variant, first script lines, vision-close tagline, and frame-0 cover intent. It only needs enough content for the thumbnail.
   - HyperFrames: `<project>/hyperframes/data/film.json`
   - Remotion: `<project>/remotion/src/data/film.ts`
5. Run `scripts/render-design-thumbnail.sh <project>`.
6. Present `<project>/out/design-thumbnail.png` to the user.
7. If the user gives aesthetic feedback, update `design.md` and the selected renderer's token file, then re-render `design-thumbnail.png`. Repeat until the user approves.

The thumbnail should feel like the film's opening title card or a representative style frame: type, palette, screen treatment, spacing, and tone in one glance. It is not final film content; it is the design decision made visible.

The design thumbnail and the cover frame are related but distinct. If the film opens on a title card, they may be nearly identical. If the film opens on a person, product state, or concrete problem object, the thumbnail proves the aesthetic while the cover frame proves the actual first frame viewers will see in embeds and previews.

End Phase 4 with: `Design thumbnail approved. → Phase 5: voice audition.`

**HARD STOP.** Present the design tokens and `design-thumbnail.png`. Wait for the user to lock the direction. If the user says it feels wrong, revise and re-render the thumbnail before proceeding. Do not advance to voice audition on design prose alone.

### Phase 5 — Voice audition

Use the ElevenLabs Player MCP (`ElevenLabs Player:generate_tts`) to play 4 voice samples reading the first ~10 seconds of the script.

**Choose 4 voices from the curated shortlist in `references/voice-shortlist.md`.** Do not present all 8. Match by the audience and tone established in Phase 2:
- Aspirational/cinematic → Adam, Brian, Rachel
- Intimate/human-moment → Charlotte, Antoni
- Authoritative/institutional → Daniel, Lily
- Warm/optimistic → Rachel, Antoni, Lily
- Contrarian/thoughtful → Will

Name your reasoning in one line ("Pulling four warm-but-grounded options given this is an LP-facing pitch") then call the TTS tool four times with the first 10–15s of the script and the chosen voice IDs.

After the user picks (or asks for "more options" → surface the remaining shortlist), save the selected voice ID and notes to `voice.json`:

```json
{
  "voice_id": "21m00Tcm4TlvDq8ikWAM",
  "voice_name": "Rachel",
  "model": "eleven_v3",
  "audition_notes": "Chose for warmth without softness; viewer is institutional but the concept is human.",
  "settings": { "stability": 0.5, "similarity_boost": 0.75 }
}
```

End Phase 5 with: `Voice selected. → Phase 6: hook render.`

**HARD STOP.** Play the four voice samples. Wait for the user to pick one. Do not proceed to render until a voice is selected.

### Phase 6 — Hook render (round-one deliverable)

This is the critical UX moment. Do NOT render the full 90s film yet. Render only the **first 10–15 seconds** — the cold open plus the first beat. This is the iteration unit.

**Steps:**

1. Run `scripts/init-project.sh` to copy the selected renderer template if Phase 4 did not already initialize it. Default to Remotion. Use `--renderer hyperframes` only for the cases in `references/renderers.md`.
2. Generate the project's content files — populate scene props, design tokens, script timing, and motion-board intent — using the selected renderer's data contract. See `references/hyperframes-integration.md` or `references/remotion-integration.md` for exact paths.
3. Generate the voiceover audio for the hook section via ElevenLabs (full TTS call with the selected voice_id and the hook script). Save via `scripts/generate-voiceover.sh <project> --hook-only`; it writes to the selected renderer's `public/audio/hook.mp3`.
4. Run `scripts/render-hook.sh <project>` which calls the selected renderer to render the Hook composition and export frame 0 as `<project>/out/cover-frame.png`.
5. Communicate render time honestly ("This will take ~2–4 minutes depending on hardware"). Don't pretend it's instant.
6. Inspect `<project>/out/cover-frame.png` before presenting. It must work as a silent poster frame at small size: not blank, not loading, not dependent on VO, and aligned with the film's promise. If it fails, adjust the cold open in the renderer data file or the tokens and re-render.
7. Run `scripts/check-static-video.sh <project>/out/hook.mp4`. If it flags unplanned freeze spans, revise the renderer data file or add the needed scene treatment and re-render.
8. Inspect the hook as an explainer before presenting it: with audio muted, does a meaningful state change happen in the first 10-15 seconds? Does the product, object, or user action cause that change? If it feels like a held slide under VO, revise the renderer data file or add the needed scene treatment and re-render.
9. Present `out/cover-frame.png` and `out/hook.mp4` to the user.

End Phase 6 with: `Hook delivered. → Phase 7: iterate or render full.`

**HARD STOP.** Present hook.mp4 to the user. Wait for explicit approval ("looks good", "render full", or similar). Do not render the full film on inferred consent, silence, or assumption. The user must explicitly greenlight the full render.

### Phase 7 — Iterate, then render full

The default state after Phase 6 is **conversational iteration**. The user will say things like:
- "Punchier open"
- "Swap the voice to something warmer"
- "The second line should land harder"
- "Make the product walk feel slower, more deliberate"
- "Change the accent color to deeper teal"

For each edit, identify the smallest re-render needed (hook only? specific scene? the whole thing?) and execute. Communicate what you're doing in one line.

**When the user is ready** ("looks good, render the whole thing" / "let's go full"):

1. Generate full voiceover audio with `scripts/generate-voiceover.sh <project>`; it writes to the selected renderer's `public/audio/voiceover.mp3`
2. Run `scripts/render-full.sh <project>`
3. Communicate the expected time (~5–10 minutes for a 90s film at 1080p)
4. Present `out/final.mp4`

End Phase 7 with: `Film complete. → Files in <project>/.`

---

## Subfile reference map

When you need depth, read the relevant subfile. Do not load these preemptively — load when the phase requires it.

| Subfile | When to read |
|---|---|
| `references/intake-checklist.md` | Phase 1 — comprehensive intake guidance per input type |
| `references/frameworks.md` | Phase 3 — full structural templates, variant deep-dives, worked examples |
| `references/script-rules.md` | Phase 3 — voice, pacing, vocabulary, do/don't, common failure modes |
| `references/motion-board.md` | Phase 3B/6 — beat-by-beat visual causality, motion modes, anti-slideshow checks |
| `references/design-language.md` | Phase 4 — UI reference querying, brand extraction, token specification |
| `references/cover-frame-strategy.md` | Phase 4/6 — cover/poster-frame archetypes and review checklist |
| `references/voice-shortlist.md` | Phase 5 — the 8 curated voices with IDs, personas, when to pick each |
| `references/renderers.md` | Phase 4/6 — renderer choice; Remotion default, HyperFrames option, other tools to watch |
| `references/hyperframes-integration.md` | Phase 6/7 — HyperFrames file layout, data shape, render commands, debugging |
| `references/remotion-integration.md` | Phase 6/7 — Remotion file layout and React component contract when selected |
| `references/example-signatures-law.md` | Anytime — worked example walking through a real concept-film run |

## Scripts and tools

The skill ships with a set of executable scripts in `scripts/`. Use them rather than inventing equivalents.

| Script | Purpose | When to run |
|---|---|---|
| `scripts/init-project.sh <project> [--renderer hyperframes\|remotion]` | Scaffold a new project directory (copies selected renderer template and creates placeholder files) | Phase 4 before thumbnail render; Phase 6 only if not already initialized |
| `scripts/audition.sh --script TEXT --voices NAMES --output DIR` | Generate ElevenLabs voice samples via direct API (fallback when ElevenLabs MCP isn't available). Produces MP3s + an `index.html` audition page. | Phase 5, only if MCP unavailable |
| `scripts/render-design-thumbnail.sh <project>` | Render the Phase 4 design thumbnail to `<project>/out/design-thumbnail.png`. | Phase 4, before voice audition |
| `scripts/generate-voiceover.sh <project> [--hook-only]` | Generate voiceover MP3 for a project's script using the voice_id in `voice.json`. Use `--hook-only` for hook render. | Phase 6 (hook), Phase 7 (full) |
| `scripts/measure-audio.sh <audio-file>` | Return audio duration in seconds. Use to align scene timing to actual TTS output length. | When debugging audio sync |
| `scripts/check-static-video.sh <video-file>` | Detect likely held-frame/freeze spans in a rendered hook or full film. | Phase 6 before showing hook; Phase 7 before final delivery |
| `scripts/render-hook.sh <project>` | Render the Hook composition (first ~15s) and export `out/cover-frame.png` from frame 0. | Phase 6 |
| `scripts/render-full.sh <project>` | Render the full ConceptFilm composition. | Phase 7, only after hook approval |

**Environment dependencies for the scripts:**

- `audition.sh`, `generate-voiceover.sh`: require `ELEVENLABS_API_KEY` env var, `curl`, `jq`
- `measure-audio.sh`: requires `ffprobe` (from ffmpeg)
- `check-static-video.sh`: requires `ffmpeg`
- `render-*.sh`: require Node.js 22+ and FFmpeg; Remotion projects also install npm dependencies on first render

**Preferred path when ElevenLabs MCP is available** (Clark's environment): use `ElevenLabs Player:generate_tts` for inline audition playback in Phase 5. Use `scripts/generate-voiceover.sh` for the actual MP3 files that feed the selected renderer.

---

## Critical rules — do not violate

1. **Never produce a full render before a hook render.** The hook is the iteration unit. Going straight to full wastes 10 minutes of compute and slows the loop.
2. **Never present more than 4 voice options at once.** Curation is the product.
3. **Never ask the user a question whose answer is already in the source material.** It signals you didn't read carefully.
4. **Never advance past the legibility gate (Phase 1) without a confirmed two-sentence summary.** Every downstream failure traces back to skipping this gate.
5. **Never produce a script over 220 words for a 90s film.** Voiceover pace is a hard constraint, not a suggestion.
6. **Never use the word "revolutionary," "game-changing," "leverage," "unlock," "synergy," or "best-in-class" in a script.** Read `references/script-rules.md` for the full banned-words list and why.
7. **Always state the chosen variant before writing the script.** One sentence. Lets the user redirect if you've read the brief wrong.
8. **Always name the gate when transitioning phases.** It tells the user where they are and signals confidence in the structure.
9. **Never run the full workflow without user interaction between gates.** Each phase gate requires human judgment. An autonomous pass through the workflow produces output that looks complete but fails on craft. If you find yourself advancing past a HARD STOP without user input, you are violating the skill's core contract.
10. **Never render the full film without explicit user approval of the hook.** "Looks good" or "render full" constitutes approval. Silence, inferred consent, or "seems fine" from a delegating agent does not.
11. **Never move from design direction to voice audition without a rendered design thumbnail.** The user cannot reliably approve an aesthetic from tokens alone. Thumbnail it, show it, revise it, then proceed.
12. **Never define the design direction without a visual source checkpoint.** A website, deck, or prototype supplied for concept intake may not be the desired design target. Give the user a chance to add visual references before you decide.
13. **Never let the cover frame be accidental.** Frame 0 must be planned, visible, readable as a still, and honest to the film. No blank fade-ins, loading states, tiny unreadable UI, or first frames that require voiceover to make sense.
14. **Never let the film become illustrated voiceover.** Every 5-8 second beat must contain a visible state change: object moves, product acts, information transforms, user decides, system responds, or output appears. Static title cards are allowed only for the cold open, insight turn, or final lockup, and never for consecutive sections.
15. **Never use `motion: 'static'` as a default.** Static holds require an explicit reason in the motion board. If the available scene components cannot honor the board's action, add a project-specific scene component or stop and tell the user the template cannot yet make the film.

## Format principles

The concept-film format this skill produces has a few defining qualities:

- The product is the protagonist's tool, not the protagonist's identity
- The voiceover speaks *to* the viewer, not *at* them
- Real screens, not abstract metaphors — viewers should leave knowing what the product looks like
- The music builds; the film lands
- Aspiration is earned through specificity, not asserted through superlatives

These principles keep the output consistent. Treat them as operating constraints when choosing structure, writing the script, and rendering the hook.

---

## Recovery patterns

**No prototype provided.** Generate schematic representations in the renderer scenes — clean wireframe-style screens that convey the ontology without pretending to be the real product. Note the constraint in `brief.md`: "Schematic screens used; real prototype to swap in for final render."

**Concept summary fails (Phase 1 gate).** Restate what you can extract and ask: "I can see [X, Y]. I can't tell [the key thing]. What am I missing?" Do not loop in the dark. One direct question, then proceed when answered.

**User wants something the framework can't handle** (e.g., a 5-minute video, an ad, a tutorial). Surface the mismatch in one line: "This wants to be a different format than the concept film — I'm built for ~60–90s pitch films. Want to proceed in this format or switch to something else?"

**Render fails.** Read the renderer error. Common causes: missing audio file, malformed data, oversized asset, or a missing timeline/composition registration. Fix at the source (the data file), don't fork the template.

**User asks for a voice not in the shortlist.** Offer to add it for this project only, with a one-line caveat: "I'm using a curated shortlist for consistency — happy to use [voice] just for this film. Do you want me to add it to your default shortlist for future runs as well?"

---

That's the operating manual. The depth lives in the subfiles. Read them when each phase asks for it.

---
> Source: [clarkvalberg/studio-video-creator](https://github.com/clarkvalberg/studio-video-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
