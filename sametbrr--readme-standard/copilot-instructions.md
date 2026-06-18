## readme-standard

> |


## Standard structure

### README.md (English)

```
[badges]
[nav line]            ← include only if ≥8 sections or ≥200 lines (Rule 14)

# Project Name

One-line description.

> 🇹🇷 Türkçe için [README.tr.md](README.tr.md)

[value paragraph]     ← include only if published package/product (Rule 20)

---

## Quick Start
## Features           ← table form if ≥7 features (Rule 19)
## Requirements
## Installation
   ### Uninstall      ← include only if project installs something persistent
## Usage              ← per-command template if CLI/plugin (Rule 21)
## Configuration      ← include only if user-facing config exists
## How It Works       ← include only if internal mechanics are non-obvious
## Project Structure  ← include only for multi-component projects
## Limitations        ← include only if known user-facing constraints exist
## Troubleshooting    ← include only if predictable failure scenarios exist
## Contributing       ← include only if contributions are accepted
## Community          ← include only if community channels exist
## Documentation      ← include only if external docs site exists
## License
[footer]              ← include only if GitHub repo and ≥200 lines (Rule 23)
```

Section selection is governed by **Conditional sections** below: a conditional
section is included only when its detection signal is present, and is never
emitted empty or as a placeholder.

### README.tr.md (Turkish)

Mirror of README.md with these rules:
- First content line (after badges): `> 🇬🇧 For English see [README.md](README.md)`
- All prose and section headings: translated — see [references/tr-translations.md](references/tr-translations.md)
- Badges: identical (same shields.io URLs)
- Code blocks: never translated

## Conditional sections

Each optional section has a defined inclusion condition and detection signal.
Check the signals during Create and Fix modes; include the section only when
the signal is present.

| Section | Placement | Include when | Detection signal |
|---|---|---|---|
| Value paragraph | After TR link, before first `---` | Published package/product | npm/NuGet/PyPI badge generated; `package.json` not `private` |
| `### Uninstall` | Subsection of Installation | Project installs something persistent | global install (`-g`), installer script, hook/MCP config writes, PATH changes |
| Configuration | After Usage | User-facing config exists | config file schema, env vars, settings file in code |
| How It Works | After Configuration | Internal mechanics non-obvious | multi-step pipeline, background process, index/cache mechanism |
| Project Structure | After How It Works | Multi-component project | multiple top-level modules/packages |
| Limitations | After Project Structure | Known user-facing constraint exists | documented constraint, known issue, platform limit |
| Troubleshooting | After Limitations | Predictable failure scenarios exist | external dependency (gh, docker, API key), auth/network step, multi-step install |
| Contributing | After Troubleshooting | Contributions accepted | `CONTRIBUTING.md`, public repo + dev/test scripts |
| Community | After Contributing | Community channels exist | Discord/Slack/forum/discussions link |
| Documentation | After Community | External docs site exists | docs URL (homepage field, published docs/) |
| Footer | After License, last element | GitHub repo + README ≥200 lines | git remote is GitHub + line count |

Rules:

- **Placement is relative order, not a dependency.** The "Placement" column
  defines the canonical order of the table; "After X" does not require X to
  exist. When a predecessor is omitted, the section slots in after the nearest
  *present* section that precedes it in the table (e.g. if How It Works is
  omitted, Project Structure goes directly after Configuration — or after
  Usage if Configuration is also omitted).
- **User instruction always overrides detection** — "uninstall ekleme" / "skip
  troubleshooting" removes the section even when the signal is present;
  "troubleshooting ekle" adds it even without a signal (ask the user for
  content if none is detectable).
- A conditional section whose condition is not met is **omitted entirely** —
  never emitted empty, as "N/A", or as a placeholder (that is a FAIL, Rule 17).
- In Audit mode, conditional sections produce **⚠️ WARN only, never ❌ FAIL**:
  - section present but signal absent → `⚠️ 'Uninstall' present but project installs nothing persistent`
  - signal present but section missing → `⚠️ Project installs globally; 'Uninstall' section recommended`
- Section names and order are fixed (Rule 11 discipline applies to conditional
  sections too). README.tr.md must contain exactly the same section set.

## Mode workflows

### Audit mode

1. Read README.md and README.tr.md (if exists)
2. Check each rule against [references/rules.md](references/rules.md)
3. Check conditional sections two-way (section↔signal, WARN only — see **Conditional sections**)
4. Print checklist — ✅ PASS / ❌ FAIL / ⚠️ WARN per rule, with line reference where it fails
5. Do NOT write any files

### Create mode

1. Detect project type and read metadata → see **Project type detection** below
2. Evaluate conditional section signals → see **Conditional sections** (user instructions override)
3. Build badge line → see [references/badge-patterns.md](references/badge-patterns.md)
4. Generate README.md from [assets/templates/README.md.tmpl](assets/templates/README.md.tmpl) — for CLI/plugin projects build Usage from [assets/templates/usage-cli-command.tmpl](assets/templates/usage-cli-command.tmpl)
5. Generate README.tr.md from [assets/templates/README.tr.md.tmpl](assets/templates/README.tr.md.tmpl)
6. Write both to the current directory

### Fix mode

1. Read existing README.md (required — error if missing)
2. Detect project type and read metadata
3. Evaluate conditional section signals → see **Conditional sections** (user instructions override; add sections whose signal is present, remove sections whose condition cannot be met)
4. Rewrite README.md: remap existing content to correct sections, apply all rules
5. Generate README.tr.md as a full Turkish translation of the new README.md
6. Confirm before overwriting existing files: "README.md ve README.tr.md üzerine yazılacak. Devam?"

### TR Sync mode

1. Read README.md (must exist — error if missing)
2. Translate fully to Turkish following [references/tr-translations.md](references/tr-translations.md)
3. Overwrite README.tr.md only

## Project type detection

Read these files in order (first match wins):

| File | Project type | Extract |
|---|---|---|
| `package.json` | npm package | `name`, `description`, `version`, `repository.url` |
| `*.csproj` | NuGet package | `<PackageId>` or `<AssemblyName>`, `<Version>`, `<Description>` |
| `pyproject.toml` | Python package | `[project] name`, `version`, `description` |
| `SKILL.md` | Claude Code skill | `name`, `description` frontmatter fields |
| No match | Generic/library | directory name + git remote |

GitHub username and repo: `git remote get-url origin` → parse from URL.
If no git remote exists, ask the user explicitly.

**Monorepo / multiple manifests (tiebreakers):**

1. A manifest at the repo root always beats one in a subdirectory.
2. Multiple *different* manifest types at the root → the table order above
   decides (`package.json` beats `pyproject.toml`, etc.).
3. Multiple *same-type* manifests in subdirectories and none at the root →
   the README is being written for the repo, not one package: use the
   directory name + git remote (Generic) and ask the user which package, if
   any, should drive the badges.

## Badge generation

Exact markdown for each type: [references/badge-patterns.md](references/badge-patterns.md)

| Project type | Badges |
|---|---|
| NuGet package | NuGet version + downloads + MIT license |
| npm package | npm version + MIT license |
| CLI / skill | GitHub release + MIT + agentskills.io + Claude Code |
| Generic | GitHub release + MIT license |

## Key rules summary

Full rules with correct/wrong examples: [references/rules.md](references/rules.md)

1. No `**EN:** ... **TR:** ...` inline bilingual patterns — ever
2. Quick Start is always the FIRST section after description + TR link
3. Quick Start: install command + minimal working example, ≤15 lines total
4. TR link in README.md: `> 🇹🇷 Türkçe için [README.tr.md](README.tr.md)`
5. EN link in README.tr.md: `> 🇬🇧 For English see [README.md](README.md)`
6. Code blocks: never translated
7. Tables: monolingual — EN-only in README.md, TR-only in README.tr.md
8. Section heading emojis: only if the project already uses them consistently
9. License section: always last `##` section — `MIT — see [LICENSE](LICENSE).`
10. All 5 core sections mandatory in order: Quick Start → Features → Requirements → Installation → Usage
11. Section names must match exactly — approved aliases only: "Commands" (CLI/plugin) and "Get Started" (multi-step install)
12. `---` horizontal rule between every `##` section (blank line above and below)
13. Description is plain text, one line — no blockquote (`>`), no multi-paragraph
14. Nav line below badges when ≥8 sections or ≥200 lines — max 6 anchor links, `•` separator
15. Troubleshooting = `**Symptom** — solution` pattern; Limitations = constraint + reason + workaround
16. `<details>` only for secondary depth (alternative paths, methodology) — primary path never collapsed
17. Conditional sections: omit entirely when condition unmet — empty/placeholder sections are FAIL
18. Callouts: only `> **⚠️ Important:**`, `> **Note:**`, `> **Heads up:**` — no freeform variants
19. Features: 2-column table mandatory at ≥7 features, recommended at ≥4 — never mix table and bullets
20. Value paragraph (≤4 sentences, bold opener) after TR link — only for published packages/products
21. CLI/plugin Usage: command cheat-sheet block + per-command `###` subsections
22. Hero block / banner / demo GIF only if visual assets exist in `assets/`
23. Footer (`Report Bug · Request Feature`, centered) after License — GitHub repos ≥200 lines only

## Section heading translations

Quick lookup: [references/tr-translations.md](references/tr-translations.md)

---
> Source: [sametbrr/readme-standard](https://github.com/sametbrr/readme-standard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
