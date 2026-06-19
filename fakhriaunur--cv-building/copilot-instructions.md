## cv-building

> >


# CV Building Pipeline

End-to-end CV creation: from any input format → semantic accumulation → normalized master doc
→ quality-gated CV YAML → professional PDFs → strategic interview materials.

**Core Philosophy: Premix Storage & Molds**

Think of the master doc as **premix storage** — a neutral, ever-growing repository of your entire career.
Think of each CV as a **mold** — a specific shape poured from the premix for a particular target.

- **Premix (master.md)** — infinite potential, neutral, always accumulating. No tailoring lives here.
- **Molds (cv.yaml)** — specific tailoring pulled from premix. Quality-gated, industry-appropriate, role-targeted.
- **Zero-assumption input** — accepts anything: full biography, draft CV, fractals, LinkedIn paste, conversation. No format required.
- **Sidecar changelog** — all additions/changes/removals tracked in `cv_files/CHANGELOG.md`. No VCS assumed.

**Entry Points:**
- `/cv-building` or "build my CV" → Full pipeline (accumulate → normalize → quality gate → render → package)
- "just want to add to my master" or "just want to fill/populate the master" → Accumulation only (add to master, update changelog)
- "remove X from my master" or "take out this role" → Content removal (delete from master, update changelog)
- "just render my CV" → Quality gate + render only (requires existing master)
- "prepare interview materials" → Strategic package generation only

**Prerequisites:**
- `rendercv` Python package: `uv tool install "rendercv[full]"` (or `pip install "rendercv[full]"`)
- `typst` compiler (installed with rendercv[full])

---

## Pipeline Overview

```
Any Input → Semantic Accumulation → Master Normalization → Quality Gate (cv.yaml) → Package
     ↓              ↓                      ↓                      ↓                      ↓
Free-form     Merge with existing     Organized, neutral     Tailored, criteria-    PDFs + strategic
CV/fractals/  master doc              master doc             matched YAML           materials
conversation
```

---

## Step 1: Semantic Accumulation

Accept any input and merge it into the growing knowledge base.

### First-Time Setup

If `cv_files/master.md` doesn't exist:
1. Create the `cv_files/` directory: `mkdir -p cv_files`
2. Initialize `cv_files/master.md` from the template: [references/master/master.template.md](references/master/master.template.md)
3. Create `cv_files/CHANGELOG.md` with a header and first entry
4. Proceed with accumulation as normal

### Multi-Persona (Optional)

If the user needs separate identities (stage name, anon web3 identity, pen name),
use suffixed master docs: `cv_files/master_<persona>.md`.

- `cv_files/master_stage-name.md` — Actor's public identity
- `cv_files/master_anon.md` — Anon web3/DAO resume
- `cv_files/master_real-name.md` — Personal identity

Each persona gets its own master and changelog. Cross-persona accumulation is never assumed.
If in doubt, default to single `master.md`.

### Input Types (No Assumptions)

| Input | Example | How to Handle |
|-------|---------|---------------|
| **Full biography** | "Here's my complete career history..." | Parse all sections into master |
| **Existing CV** | User pastes or uploads a resume | Extract all data points |
| **Draft tailored CV** | User has a CV for a specific role | Extract raw facts, strip tailoring |
| **Fragments** | "I also led a team of 5 at Company X" | Add to existing role or create new |
| **LinkedIn export** | JSON or text from LinkedIn | Parse structured data |
| **Conversation** | User describes achievements in chat | Extract and structure in real-time |
| **Correction** | "Actually my end date was March, not February" | Confirm, update existing, log in changelog |

### Accumulation Rules

1. **Never discard** — all user-provided data stays in master unless explicitly asked to remove
2. **Always merge** — new input supplements existing master, doesn't replace it
3. **Track everything** — every addition/change/removal logged in changelog
4. **No completeness assumption** — master can be partial, sparse, or empty. Always ready for more.
5. **Explicit removal only** — if user says "remove X" or "take out this role", delete from master and log in changelog. Never remove anything without explicit instruction.
6. **Confirm corrections** — when the user corrects existing info, acknowledge the change before applying: *"Understood, updating [field] from [old value] to [new value]."* Replace the value entirely (master is current truth, not a history doc), and log the change in the changelog.
7. **Detect and merge recurring details** — when new input overlaps with existing entries (same company, same role, overlapping dates), enrich the existing entry instead of creating a duplicate. Confirm before merging: *"This looks like it overlaps with your [Role] at [Company]. Should I add these details to the existing entry, or is this a different position?"* This prevents accidental double-entry from typos or fragmented input (merge conflicts).

### Probing Questions

When input is vague or brief, use targeted questions to excavate career details.
**Question bank:** [references/master/question-bank.md](references/master/question-bank.md)

Pick the most relevant questions per role rather than asking all of them. Focus on:
- **Context:** What would break if you disappeared?
- **Scale:** Users, throughput, data volumes
- **Velocity:** What manual processes did you automate?
- **Innovation:** What did you build/adopt before it was standard?
- **Influence:** Who did you mentor? What patterns still stand?

### Detect Target Context (When Building CV)

Ask (or detect from user input):

1. **Target role** — Specific job title or paste a job description
2. **Industry type** — STEM, Business/Product, Arts/Creative, Social Impact
3. **Company type** — FAANG, startup, enterprise, nonprofit, agency
4. **Level** — Junior, Mid, Senior, Staff, Principal, Manager, Director
5. **Page count** — 1 or 2 pages (default: 1)

---

## Step 2: Master Doc Normalizing

Organize accumulated data into `cv_files/master.md` using the template structure.

**Read the template:** [references/master/master.template.md](references/master/master.template.md)

### What Happens Here

- **Structure only** — organize into: Experience, Skills, Education, Projects, Leadership
- **NO tone gating** — keep the user's voice and all raw data
- **NO culture fitting** — this is the neutral source of truth
- **NO criteria matching** — save tailoring for Step 3

### Master Doc Sections

1. **Career Goals & Target Context** — Role targets, industry, level
2. **Professional Experience** — Each role: context, achievements, scale, velocity, innovation, influence
3. **Technical Skills** — Mastery / Proficiency / Learning Edge tiers
4. **Education & Certifications** — Degrees, honors, certifications
5. **Projects & Open Source** — Side projects, OSS, publications
6. **Leadership & Community** — Non-work achievements
7. **Awards & Recognition** — Industry awards, patents, papers
8. **Creative Portfolio** (if applicable) — For artists/creatives

### Changelog

**Every modification to master.md MUST be logged.**

Write to `cv_files/CHANGELOG.md`:

```markdown
# Changelog

## YYYY-MM-DD HH:MM
- **Added:** [What was added, e.g., "TechCorp Inc. role with 5 achievements"]
- **Updated:** [What was changed, e.g., "StartupXYZ end date: Feb 2022 → Mar 2022"]
- **Removed:** [What was removed, if any, e.g., "Removed placeholder education entry"]
- **Source:** [Where the info came from, e.g., "user paste", "conversation", "uploaded CV"]
```

This replaces any versioning system. The changelog is the single source of truth for what changed and when.

### Output

Write/update `cv_files/master.md` and `cv_files/CHANGELOG.md`.

**Important:** This document stays neutral. All tailoring happens in Step 3.

---

## Step 3: Quality Gate Processing

This is where the magic happens. Transform master data into `cv_files/cv.yaml` with
industry-appropriate tone, culture fit, and role-specific criteria.

**Read tone guide first:** [references/build/tone-guide.md](references/build/tone-guide.md)

### 3a: Determine Gating Level

Based on the target industry, apply the appropriate tone level:

| Industry | Gating Level | Key Characteristics |
|----------|-------------|-------------------|
| **STEM** (engineering, research, data) | Level 1 — Strict | Anti-cringe, metrics-heavy, technical precision |
| **Business** (product, consulting, finance) | Level 2 — Moderate | Business impact focus, storytelling OK, professional warmth |
| **Arts/Creative** (design, media, entertainment) | Level 3 — Flexible | Personality allowed, portfolio emphasis, mission-driven |

See [references/build/tone-guide.md](references/build/tone-guide.md) for detailed gating rules per level.

### 3b: Craft CV Content

**Headline** (one-line identity under name):
- 3-4 pipe-separated positioning phrases tailored to target role
- Note: RenderCV uses `headline` field (not `label`)
- Example: `Client-Facing AI Delivery | Cross-Functional Engineering Leadership | Systems Thinking`

**Sections** — select and prioritize from master based on target:

1. **What I Bring** (0-3 bullet entries) — Top value propositions with bold headers
   - For STEM: Technical achievements with metrics
   - For Arts: Creative achievements with impact
   - For Business: Revenue/growth metrics

2. **Experience** — Most impactful roles for this target, each with:
   - company, position, location, start_date, end_date
   - highlights: metrics-driven bullets (quantity depends on page count)

3. **Education** — Degrees, honors, relevant highlights

4. **Additional sections** as needed (Projects, Skills, Beyond Work)

### 3c: Bullet Density Rules (Level 1 - STEM)

Every highlight must contain:
1. A number that matters (%, time, money, users, scale)
2. Specific technology (never "database" — always "PostgreSQL with read replicas")
3. Business impact (why would a CEO care?)
4. Temporal context when impressive ("early 2023, before industry standard")

For Level 2-3: Adapt density — numbers still matter but narrative structure is equally valued.

**Quality bar reference:** See [references/build/before-after-example.md](references/build/before-after-example.md) for concrete before/after comparisons of CV bullets.

### 3d: Write cv.yaml (The Mold)

Each mold (cv.yaml) is suffixed by target to allow multiple simultaneous versions from the same master.

**Naming convention:** `cv_files/cv_<company>_<position>.yaml`

- `<company>` — lowercase, hyphenated company name (e.g., `stripe`, `plaid`)
- `<position>` — lowercase, hyphenated role (e.g., `senior-backend`, `staff-engineer`)
- Optional variant suffix: `_v2`, `_faang`, etc. if multiple versions for same target

**Examples:**
- `cv_files/cv_stripe_senior-backend.yaml`
- `cv_files/cv_plaid_staff-engineer.yaml`
- `cv_files/cv_stripe_senior-backend_v2.yaml` (alternate version)

Write the file following the exact RenderCV schema.

**Schema Reference:** See [references/build/rendercv-schema.json](references/build/rendercv-schema.json) or the human-readable [cv-yaml-schema.md](references/build/cv-yaml-schema.md) for the complete 4-section model (cv, design, locale, settings) and all 9 entry types.

**Complete Examples:** See [references/molds/](references/molds/) for fully renderable sample CVs and design files from the official rendercv-skill.

**YAML Structure:**
```yaml
cv:
  name: "Full Name"
  headline: "Positioning Phrase 1 | Phrase 2 | Phrase 3"
  location: "City, State/Country"
  email: user@example.com
  phone: "+1 555 123 4567"
  social_networks:
    - network: LinkedIn
      username: handle

  sections:
    What I Bring:
      - bullet: "**Bold Header:** Description with metrics."

    Experience:
      - company: Company Name
        position: Role Title
        location: City, Country
        start_date: 2023-07  # YYYY-MM format
        end_date: present    # or YYYY-MM
        highlights:
          - "**Category:** Achievement with numbers and tech."

    Education:
      - institution: University Name
        area: "Field (with honors)"
        degree: MEng
        start_date: 2011
        end_date: 2016
```

**Key Rules:**
- **Dates:** Use `start_date`/`end_date` with `YYYY-MM` or `YYYY`. Use `end_date: present` for current roles. `date` (free-form) and `start_date`/`end_date` are mutually exclusive. If only `start_date` is given, `end_date` defaults to `present`.
- **Markdown:** `**bold**`, `*italic*`, `[link](url)` are supported in strings. Block-level markdown (headers, lists, code blocks) is not rendered. Raw Typst commands and math (`$$f(x)$$`) also pass through.
- **Quotes:** **ALWAYS quote strings containing a colon (`:`).** This is the most common cause of invalid YAML. When in doubt, quote.
- **Phone:** E.164 international format only (`+15551234567`). Never invent — only include if user provides it.
- **Bullet characters:** Only these are valid: `●`, `•`, `◦`, `-`, `◆`, `★`, `■`, `—`, `○`. Do not use en-dash (`–`), `>`, or `*`.
- **Section titles:** `snake_case` keys auto-capitalize (`work_experience` → "Work Experience"). Keys with spaces or uppercase are used as-is.
- **Publication authors:** Use `*Name*` (single asterisks) to highlight the CV owner in author lists.
- **Nested highlights:** Sub-bullets are supported via indentation under `highlights:`.
- **No design in cv.yaml:** Colors, fonts, margins, and layout are set during rendering via `--design.theme` or a separate `design.yaml`.
- **No footer:** Default to `show_footer: false` in any design override. Rationale: footers waste precious space on 1-page CVs, repeat info already in the header, and make documents look like generated reports instead of polished artifacts.

Write `cv_files/cv_<company>_<position>.yaml`.

**For complete RenderCV schema, design patterns, CLI reference, and locale support:**
Install the official [rendercv-skill](https://github.com/rendercv/rendercv-skill/) — this skill's RenderCV integration is aligned to it for correctness.

---

## Step 4: Package Outputting

Generate all output artifacts.

### 4a: Render PDFs

**Available built-in themes** (RenderCV v2.8+):
| Theme | Style |
|-------|-------|
| `classic` | Blue accents, partial line dividers |
| `harvard` | Traditional academic, black text, centered |
| `engineeringresumes` | Maroon accents, no dividers, compact |
| `engineeringclassic` | Teal accents, full line dividers |
| `sb2nov` | Black text, full dividers, minimal |
| `moderncv` | Slate accents, moderncv-style headers |

For complete theme design options, see the [rendercv-skill](https://github.com/rendercv/rendercv-skill/) reference.

**To render a single theme:**
```bash
rendercv render cv_files/cv_<company>_<position>.yaml --design.theme classic
# Output: cv_<company>_<position>_classic.pdf in root dir
```

**Useful CLI options:**
```bash
# Watch mode: auto-re-render on file change
rendercv render cv_files/cv_<company>_<position>.yaml --watch

# Custom output directory
rendercv render cv_files/cv_<company>_<position>.yaml --output-folder ./output

# Override fields without editing YAML
rendercv render cv_files/cv_<company>_<position>.yaml --design.theme moderncv --cv.name "Jane Doe"

# Separate design file (reuse across multiple molds)
rendercv render cv_files/cv_<company>_<position>.yaml --design cv_files/design.yaml
```

**To render multiple themes:**
```bash
python references/scripts/render_cv.py --cv-yaml cv_files/cv_<company>_<position>.yaml --themes classic,moderncv --output-dir .
```

**To render all themes:**
```bash
python references/scripts/render_cv.py --cv-yaml cv_files/cv_<company>_<position>.yaml --all --output-dir .
```

Output PDFs go to the **root dir** (the directory where the skill is invoked):
- `cv_<company>_<position>_classic.pdf`
- `cv_<company>_<position>_moderncv.pdf`
- etc.

### 4b: Validate Page Count

Quick check: `qpdf --show-npages <pdf>` (if available). Otherwise open the PDF.

Read each generated PDF to check page count.

- **If pages > target:** Remove lowest-impact bullets first. Shorten verbose highlights. Remove least-relevant role if needed. Re-render.
- **If pages < target:** Expand descriptions with more detail from master.md. Add additional roles. Re-render.
- **Repeat** until page count matches target.

### 4c: Generate Strategic Package

Create supporting materials in `cv_files/strategic-package_<company>_<position>/`.

All materials are tailored to the same target role and draw from the master doc.

**Package naming:** Same suffix convention as the mold.
- `cv_files/strategic-package_stripe_senior-backend/`
- `cv_files/strategic-package_plaid_staff-engineer/`

**Application Stage:**

1. **cover-letter.md** — First written touchpoint. Hook + relevance + CTA. Under 200 words.
   - See [references/strategic/cover-letter-guide.md](references/strategic/cover-letter-guide.md)

**Interview Stage Materials:**

2. **hr-interview.md** — HR/Recruiter screen prep. Culture fit, motivation, logistics.
   - See [references/strategic/hr-interview-guide.md](references/strategic/hr-interview-guide.md)

3. **technical-interview.md** — Engineering deep-dive. System design, coding, architecture.
   - See [references/strategic/technical-interview-guide.md](references/strategic/technical-interview-guide.md)

4. **user-interview.md** — PM/Product sense interview. User empathy, cross-functional thinking.
   - See [references/strategic/user-interview-guide.md](references/strategic/user-interview-guide.md)

5. **topman-interview.md** — Leadership/Executive interview. Vision, business impact, presence.
   - See [references/strategic/topman-interview-guide.md](references/strategic/topman-interview-guide.md)

**Networking Materials:**

6. **elevator-pitch.md** — 30-second spoken pitch for networking events.
   - See [references/strategic/elevator-pitch-guide.md](references/strategic/elevator-pitch-guide.md)
7. **interview-opener.md** — 2-minute "tell me about yourself" answer.
   - See [references/strategic/interview-opener-guide.md](references/strategic/interview-opener-guide.md)
8. **deep-dive.md** — 5-minute narrative of candidate's strongest achievement.
   - See [references/strategic/deep-dive-guide.md](references/strategic/deep-dive-guide.md)
9. **outreach-email.md** — Cold email template for reaching target companies.
   - See [references/strategic/outreach-email-guide.md](references/strategic/outreach-email-guide.md)

### 4d: Update Changelog

If any new information was discovered during CV crafting, add it to master.md and log in CHANGELOG.md.

---

## Output Locations

| File | Location |
|------|----------|
| `master.md` | `cv_files/master.md` (shared across all targets) |
| `CHANGELOG.md` | `cv_files/CHANGELOG.md` (shared across all targets) |
| CV mold (YAML) | `cv_files/cv_<company>_<position>.yaml` |
| PDFs (all themes) | Root dir: `cv_<company>_<position>_classic.pdf`, etc. |
| Strategic package | `cv_files/strategic-package_<company>_<position>/` |

**Example:** Target = Senior Backend at Stripe
| File | Location |
|------|----------|
| Master | `cv_files/master.md` |
| Changelog | `cv_files/CHANGELOG.md` |
| Mold | `cv_files/cv_stripe_senior-backend.yaml` |
| PDFs | `./cv_stripe_senior-backend_classic.pdf`, etc. |
| Strategic | `cv_files/strategic-package_stripe_senior-backend/` |

---

## Troubleshooting

**`rendercv: command not found`**
→ Run `uv tool install "rendercv[full]"` (or `pip install "rendercv[full]"`)

**PDF spills to extra pages**
→ Reduce content, shorten bullets, or adjust margins in cv.yaml design section

**Schema validation errors**
→ Check YAML syntax. Strings containing `:` must be quoted. Dates must be `YYYY-MM` or `YYYY`. Phone numbers must be E.164 (`+15551234567`).

**Theme not rendering**
→ Use built-in theme names: `classic`, `harvard`, `engineeringresumes`, `engineeringclassic`, `sb2nov`, `moderncv`.

**For complete RenderCV reference**
→ Install the official [rendercv-skill](https://github.com/rendercv/rendercv-skill/) — covers schema, design patterns, CLI usage, and locale support.

---
> Source: [fakhriaunur/cv-building](https://github.com/fakhriaunur/cv-building) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
