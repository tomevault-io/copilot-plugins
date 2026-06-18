## scrollytelling-generator

> Convert presentations (PPTX) into scrollytelling HTML with editorial narrative, visual beats, and motion design


# Scrollytelling generator

## Skill structure

```
scrollytelling-generator/
├── references/
│   ├── discover-subagent-prompt.md  # discover — subagent task prompt with placeholders
│   ├── beat-types.md        # explore, storyboard — layout patterns, usage, anti-patterns
│   ├── storyboard-spec.md   # storyboard — BEATS JSON structure, format, examples
│   ├── build-guide.md       # build — constraints, scaffold customization, script usage
│   └── quality-check.md     # build — post-build review checklist
├── templates/               # Copy to project directory before editing
│   ├── storyboard-scaffold.html  # storyboard — empty template with render function
│   └── base-scaffold.html        # build — full shell: CSS, JS, progress bar
├── scripts/                 # Run only — not read into context
│   ├── extract.py           # discover — PPTX → slide images + metadata
│   ├── build.py             # build — storyboard BEATS JSON → final HTML
│   └── .venv/               # Python venv for extract.py
├── setup.sh                 # Install dependencies (run once)
└── SKILL.md
```

### Dependencies

- Python 3.8+
- `python-pptx` in venv at `scripts/.venv/` (`python3 -m venv .venv && .venv/bin/pip install python-pptx`)
- LibreOffice (`brew install --cask libreoffice`)

---

## Workflow summary

Four sequential phases. Each phase produces artifacts the next phase consumes. The summary below defines the goal and outputs; the linked process section has all execution steps.

### Routing (`/scrollytelling-generator`)

1. Identify what the user wants to work on and which presentation file to use
2. Determine project state and suggest the next phase:

   | State | Condition | Next phase |
   |---|---|---|
   | Fresh | No artifacts exist | Discover |
   | Discovered | `discover-analysis.md` exists, no `explore-test.html` | Explore |
   | Explored | `explore-test.html` exists, no `storyboard.html` | Storyboard |
   | Storyboarded | `storyboard.html` exists, no `scrollytelling.html` | Build |
   | Built | `scrollytelling.html` exists | Review / iterate |

   If artifacts exist but the user references a different source file, treat as Fresh.

3. Confirm the source file and direction before proceeding

### 1. Discover (`/scrollytelling-generator discover`)

Produces the narrative analysis that all later phases build on. Slide images are read in a subagent so they don't persist in the parent context.

Outputs: `discover-analysis.md` (with visual reference notes table). Follow the [Discover process](#discover) for all execution steps.

### 2. Explore (`/scrollytelling-generator explore`)

Establishes the visual design system and maps slides to beat types. Three stages: design system, beat mapping, proof-of-concept.

Outputs: `explore-test.html`, confirmed design system and layout vocabulary. Follow the [Explore process](#explore) for all execution steps.

### 3. Storyboard (`/scrollytelling-generator storyboard`)

Plans every beat: layout type, editorial copy, sequence, and rationale.

Outputs: `storyboard.html` with BEATS array, all beats confirmed. Follow the [Storyboard process](#storyboard) for all execution steps.

### 4. Build (`/scrollytelling-generator build`)

Generates production HTML from the approved storyboard using the build script.

Outputs: `scrollytelling.html`. Follow the [Build process](#build) for all execution steps.

---

## Guidelines

These apply across all phases.

### Context cost

Slide images dominate context cost. The subagent boundary in discover exists to contain this. Phases that re-read slide images (explore reads 8-10, storyboard should read 0-3) should do so deliberately.

### File handling

- All generated and edited files go in the **project directory** (where the user's presentation lives)
- Never create, edit, or overwrite files inside the skill directory (`~/.claude/skills/scrollytelling-generator/`)
- Skill files are read-only references and shared templates — copy them to the project directory before modifying

### Interaction patterns

- **Quick prompts.** Use the `AskUserQuestion` tool whenever there's a natural next action or decision point. This gives the user clickable options instead of requiring typed responses. Users can always select "Other" to provide custom input. Examples: after generating a file ("Open in browser?" / "Continue to next phase"), after presenting options ("Option A" / "Option B"), after completing a phase ("Proceed to explore" / "Review changes first").
- **Phase kickoff.** After entering a new phase, present the goal and key outputs, then follow the process section for execution steps.

### Tone and approach

- The storyboard is a collaborative planning tool, not a specification document
- Beat rationales should be concise and opinionated (strong opinions, held loosely)
- When suggesting layouts for slides, explain the reasoning so the user can evaluate the editorial judgment, not just the visual result
- Treat text-only beats as first-class content — they do the heaviest narrative lifting
- The goal is always: make the presentation's narrative *work* as a reading experience

---

## Process

### Discover

**Purpose:** Read and understand the source material. Produce a structured summary with enough visual detail that later phases can work without re-reading every slide image.

**Subagent boundary:** Steps 1-3 below run inside a subagent launched with the Agent tool. The subagent reads all slide images and writes the analysis — when it completes, those images are released from context. The parent never reads slide images directly.

**CRITICAL: The discover analysis MUST run inside a subagent. Slide images are the largest context cost in the entire workflow — reading them in the parent context will dominate the token budget for every subsequent phase.**

1. **Launch the subagent.** Read `references/discover-subagent-prompt.md`. Replace `{{PROJECT_DIR}}` with the project directory path and `{{PPTX_FILENAME}}` with the presentation filename. Pass the entire resulting text as the `task` parameter to the Agent tool. Do not summarize, abbreviate, or rewrite any part of the prompt.

2. **Verify the subagent completed.** Check that `discover-analysis.md` exists in the project directory and contains a "Visual reference notes" table. If the file is missing or incomplete, retry the Agent tool call — do NOT proceed by reading slides in the parent context.

3. **Subagent exits.** All slide images are released from context.

**Parent steps (after subagent completes):**

4. **Present brief summary in conversation.** Read `discover-analysis.md` and present ~8-10 lines:
   - Topic and thesis (1-2 sentences)
   - Section count and slide count
   - 2-3 most notable structural features that will matter for scrollytelling conversion

   Offer to open the file: *"Full analysis saved to `discover-analysis.md`. (open?)"*

5. **Discuss voice and audience.** Propose 2-3 voice options with a brief example of each (use actual content from the source material). Then ask:
   - Which voice direction feels right?
   - Who is the audience?
   - What should be emphasized or de-emphasized vs. the original presentation?

6. **Persist decisions.** Write the agreed voice, audience, and emphasis decisions as YAML frontmatter at the top of `discover-analysis.md`:
   ```yaml
   ---
   voice: [chosen voice direction]
   audience: [identified audience]
   emphasis: [what to emphasize/de-emphasize]
   decided: [date]
   ---
   ```

7. **Checkpoint.** Confirm the summary and voice direction before proceeding. The discover output is the foundation for explore and storyboard.

8. **Exit.** Suggest running explore next.

---

### Explore

**Purpose:** Establish the visual design system and map slides to beat types.

**Prerequisite:** `discover-analysis.md` must exist. If missing, suggest running discover first.

**Stage 1: Design system**

1. **Read discover analysis.** Read `discover-analysis.md` — the visual reference notes, narrative structure, and beat type candidates are your primary reference for understanding the slides.

2. **Check for existing design system.** Ask: "Do you have an existing design system to reuse — a storyboard, CSS file, or style guide?" If yes:
   - Read the source
   - Extract the design system: typography, palette, beat type vocabulary, aesthetic notes
   - Present what was extracted and confirm it applies
   - Skip step 3

3. **Propose aesthetic options.** If no existing design system: propose 2-3 distinct aesthetic directions. For each: typography pairing, color palette (hex values), tone/mood, and how it serves the content.

**Stage 2: Beat mapping**

4. **Read `references/beat-types.md`.** Understand the layout vocabulary.

5. **Read selected slide images.** Read only the slides identified as beat type candidates in the discover analysis — typically 8-10 slides. Do not read all slides. Use the visual reference notes table to identify which slides matter for each beat type decision.

6. **Map slides to beat types.** Using the visual reference notes and selected slide images, suggest specific slides as candidates for each layout type, with reasoning. Example:
   - "Slides 8-9 show the same interface before and after — good crossfade pair"
   - "Slide 20 is a full-bleed hero shot — good cinematic candidate"
   - "Slides 12-15-18 each show a workflow stage — good triptych"

**Stage 3: Proof-of-concept**

7. **Generate test HTML.** Build `explore-test.html` showing each proposed layout type applied to its best-fit slide. Design proof-of-concept, not a full build.

8. **Iterate.** Refine based on feedback: typography, palette, beat type assignments.

9. **Checkpoint.** Confirm the design system and layout vocabulary. This becomes the input for storyboard.

10. **Exit.** Suggest running storyboard next.

---

### Storyboard

**Purpose:** Generate, iterate, or reverse-engineer the storyboard — the source of truth for beat sequence, layout choices, and editorial copy.

**Prerequisite:** `discover-analysis.md` must exist. If missing, suggest running discover first.

1. **Read references.** Re-read `references/storyboard-spec.md` and `references/beat-types.md` (the explore conversation may have pushed beat type definitions far enough back in context that they're no longer reliable from memory). Copy `templates/storyboard-scaffold.html` to the project directory. Work from discover analysis, explore outputs, and visual reference notes — read individual slide images only to verify specific visual details. Do not batch-read slides.

2. **Determine storyboard action:**
   - No storyboard exists → generate from scratch
   - Storyboard exists, user wants changes → iterate based on feedback
   - Existing HTML build, no storyboard → reverse-engineer from HTML

3. **Generate storyboard (from scratch):**
   - Apply the agreed design system and layout vocabulary (from explore) to all slides
   - Write the BEATS array with section groupings, beat types, editorial copy, and rationale
   - Set status to `"proposed"` for new beats, `"confirmed"` for user-approved beats
   - Generate the storyboard HTML file using the scaffold template
   - **In conversation:** present a brief stats summary and offer to open. Do NOT list beats inline — the storyboard HTML is the artifact. Example: *"Generated `storyboard.html` — 34 beats across 4 sections, all proposed. (open in browser?)"*

4. **Iterate based on feedback:**

   | Feedback type | Action |
   |---|---|
   | Remove a beat | Remove from BEATS array |
   | Add a text transition/synthesis beat | Add to BEATS with suggested copy |
   | Change a layout type | Update beat `type` field |
   | Adjust copy only | Update `headline`/`body` text, no layout changes |
   | Reorder beats | Update sequence in BEATS array |
   | Confirm beats | Change `status` to `"confirmed"` |

   After each round, regenerate the storyboard file.

5. **Reverse-engineer from existing HTML:**
   - Read the HTML, identify each beat by CSS class
   - Extract beat type, slide references, editorial copy, section grouping
   - Generate BEATS array with all statuses `"confirmed"`
   - Render the storyboard

6. **Confirm beats.** Default to section-by-section confirmation — present each section's beats and ask the user to confirm, request changes, or flag beats for discussion. Alternatively, offer beat-by-beat (for careful review) or all-at-once (if the user is ready to ship). After each confirmation round, update `status` fields to `"confirmed"` and regenerate the storyboard file. All beats must be `"confirmed"` before build.

7. **Exit.** Suggest running build next.

---

### Build

**Purpose:** Generate the full scrollytelling HTML from an approved storyboard.

The build script (`scripts/build.py`) reads the BEATS JSON from the storyboard and assembles final HTML. Claude's role: confirm readiness, run the script, review the output — not hand-write HTML.

1. **Read build guide.** Read `references/build-guide.md` for constraints and scaffold customization rules.

2. **Confirm prerequisites.** Storyboard exists, all beats confirmed. If not, suggest running storyboard first.

3. **Confirm build plan with user.** Present:
   - Design system to apply (fonts, palette from storyboard header)
   - Scaffold customizations needed
   - Image paths check

4. **Customize scaffold.** Copy `templates/base-scaffold.html` to the project directory as `base-scaffold-custom.html`, then edit the copy:
   - Replace the Google Fonts `<link>` with the project's fonts
   - Replace `:root` CSS variables with the project's palette
   - **NEVER edit files in the skill directory** — always copy first

5. **Run build script:**
   ```
   python3 ~/.claude/skills/scrollytelling-generator/scripts/build.py storyboard.html --output scrollytelling.html --scaffold base-scaffold-custom.html
   ```

6. **Review output.** Read `references/quality-check.md` and run the full checklist against the generated HTML. Present findings to the user in two categories: "I can fix" (will fix immediately) and "need your input" (requires a decision). Always show the check results even if everything passes — the user needs to see that the review happened. The build script handles structural correctness, so focus especially on the editorial review items.

7. **Iterate.** If changes are needed:
   - **Copy/content** → update BEATS in storyboard, re-run script
   - **Layout/beat type** → update beat `type` in storyboard, re-run script
   - **Design system** → update `base-scaffold-custom.html` CSS variables, re-run script

8. **Checkpoint.** Confirm the output looks right.

9. **Exit.** When all sections are built, suggest reviewing the complete piece.

**Section-by-section:** For storyboards exceeding ~35 beats or when the user requests it, build incrementally. Name partial storyboards `storyboard-s1.html`, `storyboard-s2.html`, etc. Each includes a rhythm strip for its section. Combine into a full storyboard when all sections are complete.

---
> Source: [lmanheim/scrollytelling-generator](https://github.com/lmanheim/scrollytelling-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
