## docs-structure

> description: Documentation chapter structure and style for docs/

---
description: Documentation chapter structure and style for docs/
globs: docs/**/*.md
alwaysApply: false
---

# Docs — Chapter Structure & Style

Documentation in `docs/` is the homelab "journey": part journal, part technical guide. Keep it consistent and documentation-first (include "why").

## File Naming

- **Chapters:** `ChapterN-topic.md` or `ChapterNX-topic.md` for sub-chapters.
  - Examples: `Chapter0-hardware.md`, `Chapter1-proxmox.md`, `Chapter2-vms.md`, `Chapter2a-core.md`, `Chapter2c-media.md`
- Use lowercase, hyphen-separated topic names. Sub-chapters use a letter after the number (2a, 2b, 2c, 2d).

## Document Structure

1. **Title** — Clear chapter title, optionally with emoji (e.g. `# Chapter 2: VM Overview — How the Lab is Separated (and Why)`).
2. **Introduction** — What this chapter covers and why it matters.
3. **Philosophy / reasoning blocks** — Use blockquotes for design notes and "why":
   ```markdown
   > ### 🧠 Philosophy: Short Title
   > Explanation of the reasoning or tradeoff.
   ```
4. **Sections** — Use `##` for major sections, `###` for subsections. Use horizontal rules `---` to separate major blocks when it improves scanability.
5. **Tables** — Use for inventories, quick reference, "what runs where", VMID mappings, app roles.
6. **Steps** — Numbered lists for procedures (e.g. "Steps (Proxmox)", "Steps to Prepare"). Include verification commands where relevant.

## Style

- Write for "future me" and readers: explain decisions, not just steps.
- Prefer concise, scannable sections; put deep "why did I choose this app" in follow-up chapter files (2a, 2b, 2c).
- Use **bold** for key terms in tables and lists. Keep quick-reference sections compact; expand in dedicated subsections.
- Code blocks: use bash for shell commands, yaml for config snippets. Specify language when it helps.

## Cross-References

- Point to other chapters by name and file (e.g. "See Chapter 2A (`core`)", "Full bootstrap design lives in the Docker/Compose chapter (planned)").
- Keep README as the map; docs chapters are the detailed narrative.

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
