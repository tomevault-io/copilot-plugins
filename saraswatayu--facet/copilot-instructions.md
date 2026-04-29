## facet

> Pre-launch simulation engine: generates research-grounded personas and simulates them through product studies. See [README.md](README.md) for user-facing docs, positioning, and limitations.

# Facet — Codebase Guide

Pre-launch simulation engine: generates research-grounded personas and simulates them through product studies. See [README.md](README.md) for user-facing docs, positioning, and limitations.

## Architecture

Six-command pipeline:

```
sim.sh init:       Plan (Opus) → Generate (Sonnet, parallel waves)
sim.sh study:      Simulate (Sonnet, parallel) → Analyze (Opus)    [--runs N for stability]
sim.sh synthesize: Cross-Synthesis (Opus) — unified findings across studies
sim.sh run:        Full lifecycle — init panel + all studies + synthesize (single command)
sim.sh compare:    Diff findings between two panels
sim.sh status:     Panel progress, study completion, cross-synthesis readiness
```

Primary interface:
- **sim.sh** — bash orchestrator for terminal/CI use. Each phase is a `claude --print` subprocess.

Key design: personas are generated ONCE and reused across multiple studies. The `study` subcommand reads a single study config containing all studies and runs the full pipeline.

### Model Routing

| Phase | Model | Why |
|-------|-------|-----|
| Plan | default (Opus) | Reasoning-heavy: segment design, constraint vectors, diversity matrix |
| Generate | `--model sonnet` | Cost efficiency at scale (50 parallel personas) |
| Simulate | `--model sonnet` | Cost efficiency at scale (50 parallel simulations) |
| Analyze | default (Opus) | Synthesis quality: reads all personas + simulations, produces recommendations |
| Cross-Synthesize | default (Opus) | Reads all per-study syntheses + persona summaries, produces unified findings |

### Tool Restrictions

All `claude` invocations are restricted to `Read,Write,Glob,Grep`. No Bash — no escape hatch. The one exception: calibration directory mode adds Glob and Grep to persona generation (normally Read,Write only).

## Wave-Based Generation

Personas are generated in waves of 5 to prevent homogeneity.

```
Wave 1: personas 1-5   → no diversity context
Wave 2: personas 6-10  → sees summaries of 1-5
Wave 3: personas 11-15 → sees summaries of 1-10
...
```

After each wave completes, `extract_persona_summary()` in sim.sh reads each generated persona and builds a one-line summary (name, segment, key traits). This summary block is injected into the next wave's prompt with explicit instructions: "your persona must sound, think, and decide differently from ALL of the above."

Each wave must fully complete before the next starts — the diversity context depends on it.

## Template Patterns

All four templates (`plan.md`, `persona.md`, `simulation.md`, `analysis.md`) follow consistent patterns:

1. **Integrity rules** — anti-sycophancy guardrails. Every template explicitly permits rejection, skepticism, indifference. "This persona is NOT obligated to like the product."
2. **Quality bar** — specific > generic. "$38,500/year" not "moderate salary." Named neighborhoods, not "urban area."
3. **Thinking steps** — brainstorm/analysis sections marked "do NOT include in output." These prompt Claude to reason before writing but keep the output clean.
4. **Consistency self-check** — templates end with verification that stated facts, numbers, and decisions are internally consistent.

### Study-Type Rules

Each study type in `study-types/` specifies which behavioral economics frameworks apply:

| Study Type | Frameworks | Key Metrics |
|-----------|------------|-------------|
| `pricing.md` | Prospect theory, mental accounting, loss aversion, flat-rate bias, zero-price effect | Signup decision, 12-month usage table, renewal, NPS, referral |
| `copy.md` | ELM (central/peripheral routes), construal level, reactance, framing effects | Clarity, trust, motivation, shareability (0-10 each) |
| `features.md` | Kano model (must-be/performance/attractive/indifferent), feature interaction | Per-feature importance, excitement, WTP delta, usage frequency |
| `onboarding.md` | Endowment effect, IKEA effect, psychological ownership, status quo bias, default effect, Fogg B=MAP | Completion funnel, time-to-value, ownership score, status quo shift rate |
| `retention.md` | Hedonic adaptation, sunk cost, peak-end rule, post-purchase rationalization, trust decay | 12-month satisfaction arc, churn taxonomy, honest vs. passive retention rate |
| `custom.md` | None (persona uses natural decision-making) | Gut reaction, reasoning, verdict, concerns per option |

Each study type also has outcome requirements (e.g., "at least 1 persona should churn," "features study is the weakest use case — surface reactions, not ranked lists"). The custom study type has no framework injection — for research questions that don't fit the standard types.

## Template Version Locking

Templates are copied into `.templates/` directories at init and study time:
- `{panel}/.templates/` gets `plan.md` + `persona.md` + `cross-synthesis.md` + `comparison.md` at init
- `{panel}/studies/{study}/.templates/` gets `simulation.md` + `analysis.md` + `stability.md` + `{study-type}.md` at study time

This means existing studies are reproducible even if source templates change. Contributors modifying templates don't break past studies.

## Calibration Data

The `--calibration` flag grounds personas in real-world data instead of LLM priors.

**Single file mode:** File content is injected directly into the planning prompt.

**Directory mode:** Claude uses Glob to discover files. If `manifest.md` exists at the directory root, it's read first — it describes each file's purpose. Claude selectively reads the most relevant files. The plan output includes a "Calibration Sources" section listing what was read and extracted. Directory mode enables Glob/Grep tools for persona generation (normally Read/Write only).

Supported file types: `.md`, `.csv`, `.txt`, `.json`, `.yaml`, `.yml`.

## Key Files

```
sim.sh                  — orchestrator (init, study, synthesize, run, compare, status subcommands)
stream_filter.py        — real-time progress display, parses stream-json, uses FACET_PHASE env var
templates/
  plan.md               — planning: segments, constraint vectors, diversity matrix, name registry
  persona.md            — persona background generation (identity, psychology, domain, discovery)
  simulation.md         — per-persona study simulation (Chain-of-Feeling, BDI verdicts)
  analysis.md           — unified analysis → synthesis.md + artifacts.md (TWO files)
  cross-synthesis.md    — cross-study synthesis → unified findings across all studies
  comparison.md         — study comparison → stable vs fragile findings
  stability.md          — simulation stability report → per-persona consistency
study-types/
  pricing.md            — prospect theory, mental accounting, 12-month usage tables
  copy.md               — ELM, construal level, framing effects, per-variant scoring
  features.md           — Kano model, feature interaction, prioritization caveats
  onboarding.md         — endowment/IKEA effect, psychological ownership, Fogg B=MAP
  retention.md          — hedonic adaptation, sunk cost, peak-end rule, churn taxonomy
  custom.md             — no framework injection, for non-standard research questions
examples/
  superhuman-product.md    — example product config (10 segments × 5 personas)
  superhuman-pricing.md    — example pricing study (3 tier models)
  superhuman-copy.md       — example copy study (6 positioning variants)
  superhuman-features.md   — example features study
  superhuman-onboarding.md — example onboarding study (3 flows)
  superhuman-retention.md  — example retention study (3 strategies)
research/               — ~490-source research reports informing template design
setup                   — dependency check and quickstart guide
parse_config.py         — YAML frontmatter parser (replaces sed-based parsing)
test_utils.py           — unit tests for parse_config.py
study-templates/        — 5 pre-built study configs (concept, pricing, activation, positioning, lifecycle)
```

## Config Format

**Product config** (for `init`): Markdown with YAML frontmatter containing `segments` and `personas_per_segment`. Body describes product, key details, target market.

**Study config** (for `study`): Markdown with YAML frontmatter containing `study_name`, `study_type`, `options`, and optionally `copy_variants`. Body has options detail and copy variants.

**Full run config** (for `run`): Markdown with YAML frontmatter containing `segments`, `personas_per_segment`, optional `calibration`, and `studies` array. Each study entry has a `config` path (relative to the config file). Body is the product description (doubles as product config for init).

See `examples/` for study/product configs, `study-templates/` for study configs.

## Output Structure

The panel directory contains personas (shared across studies) and per-study results. When used via the `/facet` skill, the panel root is `.facet/` in the user's project. When used via CLI, it defaults to `output/{name}/`.

```
{panel}/                                # .facet/ (skill) or output/{name}/ (CLI)
├── .templates/                         # version-locked init + cross-synthesis templates
├── plan.md                             # segment matrix, constraint vectors, diversity matrix
├── personas/
│   ├── persona-001.md                  # background ONLY (identity, psychology, domain, discovery)
│   └── ...
├── .status                             # JSON-line phase completion tracking
├── cross-synthesis.md                  # unified findings across all studies
└── studies/
    └── {study-name}/
        ├── .templates/                 # version-locked study templates
        ├── study-config.md             # copy of study config
        ├── simulations/
        │   ├── persona-001.md          # Chain-of-Feeling arcs, BDI verdicts, 12-month tables
        │   ├── persona-001-summary.md  # structured sidecar (verdict, quotes, numbers)
        │   └── ...
        ├── synthesis.md                # analysis + recommendation + counterargument
        ├── artifacts.md                # actionable deliverables + validation plan
        └── verification.md             # spot-check results (separate, never appended to synthesis)
```

## Conventions

These are invariants — do not break them:

- Personas contain ONLY backgrounds — never simulations, verdicts, or copy reactions
- Persona outlines in plan.md use `Persona #N` numbered format (sim.sh regex depends on this)
- Persona files are zero-padded: `persona-001.md`, `persona-023.md`
- Analysis produces TWO files: `synthesis.md` and `artifacts.md` (not one combined file)
- Names must be unique across the entire study (enforced by name registry in plan.md)
- All numbers in personas/simulations must be specific and internally consistent
- Template sections adapt to product domain (not hardcoded to travel/flights)
- Templates are version-locked into `.templates/` at init and study time
- No Bash tool in `claude` invocations — Read, Write, Glob, Grep only

## Adding a New Study Type

1. Create `study-types/{name}.md` following the pattern in existing study types
2. Include: caveat section, what it tests, simulation framework, per-persona metrics, outcome requirements
3. Create an example study config in `examples/` with matching `study_type` in frontmatter
4. Test: `./sim.sh init --config examples/superhuman-product.md --name test` then `./sim.sh study --panel output/test/ --config examples/your-study-config.md`
5. Review the synthesis — does it produce actionable recommendations?

No code changes needed in sim.sh — it reads `study_type` from frontmatter and loads `study-types/{type}.md` automatically.

## Modifying Templates

1. Read the template you want to change — understand the full structure first
2. Make your change — maintain the integrity rules, quality bar, and thinking step patterns
3. Test with a full run (init + study) using example configs
4. Check that output format hasn't changed (downstream phases depend on it)
5. Note: your change won't affect existing studies — templates are version-locked at init/study time

See [CONTRIBUTING.md](CONTRIBUTING.md) for submission guidelines.

## Skill routing

When the user's request matches an available skill, ALWAYS invoke it using the Skill
tool as your FIRST action. Do NOT answer directly, do NOT use other tools first.
The skill has specialized workflows that produce better results than ad-hoc answers.

Key routing rules:
- Product ideas, "is this worth building", brainstorming → invoke office-hours
- Bugs, errors, "why is this broken", 500 errors → invoke investigate
- Ship, deploy, push, create PR → invoke ship
- QA, test the site, find bugs → invoke qa
- Code review, check my diff → invoke review
- Update docs after shipping → invoke document-release
- Weekly retro → invoke retro
- Design system, brand → invoke design-consultation
- Visual audit, design polish → invoke design-review
- Architecture review → invoke plan-eng-review
- Save progress, checkpoint, resume → invoke checkpoint
- Code quality, health check → invoke health

---
> Source: [saraswatayu/facet](https://github.com/saraswatayu/facet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
