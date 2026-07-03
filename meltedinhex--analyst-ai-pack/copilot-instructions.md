## analyst-ai-pack

> This workspace is **AnalystAIPack**, a library of 118 security skills for **malware analysis,

# AnalystAIPack — GitHub Copilot instructions

This workspace is **AnalystAIPack**, a library of 118 security skills for **malware analysis,
reverse engineering, and threat hunting**. Each skill is a folder under `skills/` containing a
`SKILL.md` (procedure) and a runnable `scripts/analyst.py` (a static, read-only analysis tool).

## How to help in this repo

- When the user asks an analysis/hunting task, **find the matching skill first** instead of
  improvising. Discover skills with:
  - `python tools/analyst-pack.py search <topic>`
  - `python tools/analyst-pack.py list [--subdomain malware-analysis|reverse-engineering|threat-hunting|lab-foundations]`
  - `python tools/analyst-pack.py show <skill-name>`
- Read the chosen `skills/<name>/SKILL.md` and follow its **Workflow**. Honor its
  `When to Use` and especially the **Do not use** boundary.
- Run a skill's tool via:
  `python tools/analyst-pack.py run <skill-name> -- <subcommand> <args>`
  or directly `python skills/<skill-name>/scripts/analyst.py <subcommand> <args>`.
  Output is structured JSON — pass it forward to the next skill or into a report.

## Safety rules (do not violate)

- **Never execute a malware sample.** All scripts are static and read-only. Do not write code
  that runs, detonates, or connects to a sample's infrastructure.
- **Assume an isolated lab** for any live-sample step; follow each skill's `Safety & Handling`.
- **Keep IOCs defanged** in output (`hxxp://`, `1[.]2[.]3[.]4`). The scripts already do this —
  preserve it.
- **No live samples in the repo** (see `.gitignore`). Some scripts contain malware *signature
  strings* for detection; antivirus may flag the file text — that is static data, not behavior.

## Authoring or editing a skill

Keep the repo green and follow the conventions:

- Each skill needs: `SKILL.md` + `scripts/analyst.py` + `references/api-reference.md` + `LICENSE`.
- Frontmatter: `name` (kebab == folder), `description` (ends with an "Activates for …" trigger),
  `domain: cybersecurity`, `subdomain` (one of the four), `tags`, `version`, `author`, `license`,
  and framework IDs (`mitre_attack`, `d3fend`, optional `car`).
- Body sections in order: `When to Use` (with **Do not use**), `Prerequisites`,
  `Safety & Handling` (sample-handling skills), `Workflow`, `Validation`, `Pitfalls`, `References`.
- Scripts are **standard-library only** (optional deps degrade gracefully) and never execute the
  sample. Validate with:

```bash
python tools/lint-skills.py --all
python tools/build-index.py && python tools/build-catalog.py && python tools/build-navigator-layer.py
python tools/smoke-test.py
```

See `AGENTS.md` (full operating guide) and `CONTRIBUTING.md` (originality rules).

---
> Source: [meltedinhex/analyst-ai-pack](https://github.com/meltedinhex/analyst-ai-pack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
