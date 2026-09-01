## fora

> This file is a guide for AI assistants (like Claude, Gemini, Antigravity, Cursor, etc.) contributing to FORA. It outlines the codebase structure, build instructions, test commands, code style guidelines, and core constraints.

# FORA — AI-Assisted Contributor Guide (CLAUDE.md)

This file is a guide for AI assistants (like Claude, Gemini, Antigravity, Cursor, etc.) contributing to FORA. It outlines the codebase structure, build instructions, test commands, code style guidelines, and core constraints.

---

## 1. Repository Structure

```
├── README.md               # Main project introduction
├── SETUP.md                # Environment setup and onboarding flow
├── ROADMAP.md              # Planned items for V1, V2, and V3
├── CHANGELOG.md            # All notable changes, versioned
├── generate.js             # Core pipeline engine (Planner, loaders, codegen, assembler)
├── brainstorm.sh           # Shell script to fetch JD and assemble brainstorm prompt
├── run.sh                  # Interactive runner orchestrating the end-to-end pipeline
├── setup.sh                # Environment validator and health checker
├── prompts/
│   ├── brainstorm-prompt.md # Prompts for Phase 1 & 2 brainstorm in LLM chat
│   └── codegen-prompt.md    # System/rules instructions used internally by generate.js
├── design-system/
│   └── default.md          # Personal design system CSS tokens and instructions
├── templates/
│   ├── sections/           # Scaffolding HTML section templates (nav, act1, etc.)
│   └── *.json              # Template config definitions (three-act, work-first)
├── test/
│   └── regression.sh       # Regression test suite for golden briefs & output validation
├── git-hooks/
│   └── pre-commit          # Git pre-commit hook preventing staging of .env
└── applications/
    └── applications.json   # Unified JSON database logging all applications
```

---

## 2. Commands & Testing

- **Dry-run generation (free, mock placeholders):**
  `node generate.js --dry-run briefs/example-brief.json`
- **Standard generation (needs API keys, runs AI codegen):**
  `node generate.js --run briefs/your-brief.json`
- **Publish generation (runs codegen + deploys to Vercel):**
  `node generate.js --publish briefs/your-brief.json`
- **Deploy existing output (skip codegen, redeploy HTML):**
  `node generate.js --deploy briefs/your-brief.json`
- **Run regression tests (dry-run mode):**
  `./test/regression.sh --dry-run`
- **Run regression tests (live AI generation mode):**
  `./test/regression.sh`
- **Run environment health check:**
  `./setup.sh --check`

---

## 3. Code Style & Rules

1. **Zero External Dependencies:** The core node engine (`generate.js`) must not import any non-native packages. Use built-in Node.js modules like `fs`, `path`, `crypto`, and native `fetch` (Node 18+).
2. **First-Person Voice for Copy:** AI-generated framing text (`{{framing_line}}`) must always be written in the first person ("I did X", not "He did X" or "The candidate did X").
3. **No em-dashes (—):** Banned from AI copy generation rules. Use plain hyphens/dashes (-) or rewrite the sentence to avoid them.
4. **HTML Slots:** Slots on sections must match variables defined in templates. When slot values are empty/null/false, they should be fully omitted rather than showing TBD/empty placeholders.
5. **Strict Local Privacy:** Personal candidate profiles (`profile.json`), saved application briefs (`briefs/*.json`), generated outputs (`output/`), and configuration `.env` are private. They are gitignored and must never be staged or committed.

---

## 4. Architecture Decisions (Do Not Alter)

- **Per-Section Codegen:** We call the LLM separately for each section rather than in one monolithic call. This isolates failures and improves section-specific instruction following. Keep this architecture intact.
- **Clipboard workflow:** Brainstorm relies on clipboard copying for external LLM chat interaction. Do not attempt to replace the CLI clipboard flow with an automated browser/API agent for brainstorm in V1.
- **Base64 media URIs:** Images are resolved to base64 data URIs so that the generated index.html is completely self-contained. Keep this to avoid complex asset path deployment management.

---
> Source: [meetshahco/FORA](https://github.com/meetshahco/FORA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
