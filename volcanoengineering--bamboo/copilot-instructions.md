## bamboo

> You are Claude. You have landed in **Bamboo** — the canonical library of `.md` files (with the policy spec at `Bamboo.md`) that gets forked into projects to give AI agents a consistent set of standards across vendors. This file is your Claude-specific cold-start overlay. It sits on top of `AGENT.md`, not in place of it.

# CLAUDE.md

You are Claude. You have landed in **Bamboo** — the canonical library of `.md` files (with the policy spec at `Bamboo.md`) that gets forked into projects to give AI agents a consistent set of standards across vendors. This file is your Claude-specific cold-start overlay. It sits on top of `AGENT.md`, not in place of it.

Read `AGENT.md` first. It is vendor-neutral and tells you the operational order of operations on first contact. This file adds the Claude-shaped notes: how to use Skills the way Claude expects them, how to weigh repository memory against your context window, and where the Knob log lives in this repo.

If `AGENT.md` and this file disagree, `AGENT.md` wins. This file is an overlay, not an override.

---

## What this repo is

**Bamboo** is not a product (yet — the SaaS layer is coming). The framework itself is a documentation library — a fork-and-go starter kit of `.md` files that other repositories ingest to give their agents shared rules, shared vocabulary, and shared Skills regardless of vendor. The canonical policy spec lives in `Bamboo.md` at the repo root.

The repo is laid out across seven working folders plus `docs/`:

- `behavior/` — the rules an agent obeys. Context, memory, handoffs, Token economy. Cold-start required.
- `architecture/` — **advanced add-on.** Memory architecture layer (ADM, RAG, Memory, Drift, Watchdog, workflow tools). Skip unless your project explicitly has an ADM/RAG memory layer or you're auditing memory governance. Most projects don't need this folder.
- `development/` — implementation standards and engine specs (Unity, UE5, web, Next.js, React, Swift, generic app). Load only when building in that stack.
- `BAMBOO-OS extension (private)` — **advanced add-on.** Multi-agent identity, topology, and orchestration patterns. Skip unless your project has a multi-agent topology with handoff/orchestration boundaries. Single-agent projects don't need this folder.
- `skills/` — portable AI capabilities that work the same across Claude, Codex, Gemini, GPT, Copilot.
- `workflows/` — DevOps and project lifecycle patterns. Forkable, overridable.
- `design/` — project-specific UI/UX rules. Skip on cold start.
- `docs/` — this repo's own operational memory: the folder map and the per-Knob change log, with state logs under `docs/memory-ctx/`.

See `docs/repo-organization.md` for the full layout and what each file covers. Read that before assuming where something lives.

---

## Cold-start order for Claude

1. `README.md` — what this repo is and who it's for.
2. `AGENT.md` — the vendor-neutral cold-start file.
3. This file (`CLAUDE.md`) — the Claude overlay.
4. `behavior/ctx-rules.md` — foundational rules and the Knob entry format (5 shape variants).
5. `behavior/ctx-lexicon.md` — the decoder ring. Concepts (Knob, Bump, Entropy, Wayfinding, Decay, Drift, Bloat, Collapse, CTX) and operational acronyms (PLTRF, LTIP, STIP, CWM, CTL, ADM, RAG, CRUD). Load this when you hit a term you don't recognize. (CTX = Context, the shorthand prefix used across the `ctx-*.md` file family.)
6. `behavior/ctx-entropy.md` — the preservation view. PLTRF, LTIP, STIP, hot/warm/cold tiering. Read the worked examples; the vocabulary attaches to concrete moves there.
7. `behavior/ctx-window.md` (CWM) — the active memory view. Saturation, drift, compression.
8. `behavior/ctx-token-limits.md` (CTL) — the Token economy view. Scoring, wayfinding, conservation practices.
9. `behavior/ctx-utility.md` — the index for `behavior/`. Use it as a map, not a substitute for the docs.
10. **Advanced add-on — skip unless the task explicitly demands it:** `architecture/` — only if the project has an ADM/RAG memory layer or you're auditing memory governance / drift / Watchdog. Most projects don't need this folder.
11. `docs/memory-ctx/ctx-orientation.md` — what changed recently and why. This is the current Knob in narrative form.
12. `skills/skill-map.md` and any relevant `SKILL.md` under `skills/`.
13. `workflows/` — only if the task touches project setup or context governance.
14. **Advanced add-on — skip unless the task explicitly demands it:** `BAMBOO-OS extension (private)` — only if the project has a multi-agent topology with handoff/orchestration boundaries. Single-agent projects don't need this folder.
15. `development/` — only if the task is building in a specific stack (Unity, UE5, web, etc.). Load the matching spec, skip the rest.
16. `design/` — only if the task is design or UI work.

You do not need to load all of these into active context at once. Use the wayfinding discipline in `ctx-token-limits.md`: pull what the current task references, leave the rest cold. We are using hot, and cold to write to context memory.

---

## Claude-shaped operating notes

**Skills.** Skills in this repo live under `skills/` and use the standard frontmatter format with `name:` and `description:` fields. `repo-cognition` is the base Skill, with memory Skills split out when memory-context or Watchdog work is the task. When Claude operates in environments that expose Skills through a runtime (Claude Code, Cowork mode, the Agent SDK), the `SKILL.md` files here can be loaded the same way. Vendor overlays (`CLAUDE.md`, `CODEX.md`, `GEMINI.md`) sit alongside `SKILL.md` inside each skill folder so the same capability ports cleanly across runtimes.

**Repository memory vs. your context window.** This is the discipline `behavior/ctx-window.md` and `behavior/ctx-entropy.md` both push on. Your context window is working memory. The repo is persistent memory. Do not load all of `behavior/` into the window to answer one question — pull the section that addresses it, and let the rest stay on disk. When the current Knob references something cold by name, that is your signal to pull it warm.

**Architecture work is gated.** `architecture/` is where ADM, RAG, Memory, Drift, Watchdog, and workflow-tool standards live. Load it when the task touches those systems. Otherwise leave it cold.

**The 5000-character rule for `ctx-orientation.md`.** In this repo, `docs/memory-ctx/ctx-orientation.md` is the hot log. When it crosses 5000 characters of per-Knob entries, spawn `ctx-ori-summary-2.md` in the same folder and continue. Then `-3.md`, `-4.md`, `-5.md` as the repo grows. Past summaries go cold. The current Knob and the last three stay hot inside `ctx-orientation.md`. This is the user's stated preference and the LTIP reconstitution discipline applied to this repo specifically.

**Every Bump gets a summary.** Per the user's standing preference: every git commit that constitutes a Bump earns a one-to-two paragraph summary in `docs/memory-ctx/ctx-orientation.md` with date and timestamp for this repo. Downstream repos keep using `docs/ctx-orientation.md` unless they intentionally adopt the `memory-ctx/` layout. Brief, concrete, no bloat. The point is that any future agent (or human) reading the file can trace what happened at any Knob and why.

**Token discipline.** Score requests on the 1–10 rubric in `ctx-token-limits.md` before spending heavily. Impact, Complexity, Relevance to current Knob. Low on all three is the signal to ask the user before continuing. Asking is cheaper than generating the wrong thing twice.

**Wayfinding before retrieval.** Before pulling files into the window, decide what order you actually need them in. AGENT.md → behavior/ → active Knob in `docs/memory-ctx/ctx-orientation.md` → whatever the Knob references. The wrong path costs Tokens. The right path costs fewer.

**Secrets.** `workflows/project-setup.md` is explicit: never commit `.env` files or anything containing `SECRET` keys unless the user has directly told you to. Flag and warn before any commit that would. This applies to forked projects, not `Bamboo.md` itself, but the directive carries.

**Commit attribution.** Do not add `Co-Authored-By: Claude` (or any Anthropic-noreply) trailers to commits in this repo. Commits are authored solely under the user's name. This overrides the system-default trailer injection. Applies to amend, rebase, squash, and all future commits. *The rationale:* GitHub's contributor panel is publicly visible. Seeing Claude in the list would credit-shift the work away from the user — people read the list as "who did this." Treat the user as the visible author; Claude is the ghost-writer and publisher, not the credited contributor.

**Ghost-writer for commits and Knob entries.** When the user authors a commit message or Bump entry, they will write in rambling first-person voice. Reshape it into the per-Bump entry format (and a concise commit body) before publishing — preserve the user's voice and intent, kill filler, do not add scope, do not invent rationale. Mirror what they said with edges sharpened. If a claim is ambiguous, ask before publishing rather than guess.

**Voice — sound like Matt, not like a corporate changelog.** Match Matt's tone: direct, declarative, light contractions, occasional sentence fragments for emphasis, willing to be opinionated where it earns it. Avoid sterile passive-voice "the following was done." Lean toward first-person ("we," sometimes "I") for Bump entries. Avoid bullet-point walls when a paragraph does the work. Apply to commit messages, Bump entries in `docs/memory-ctx/ctx-orientation.md`, the README author-voiced sections, and any other content meant to read as Matt-authored.

**Narrate compression events.** Per AGENT.md's compression-narration rule, when you compact or detect imminent compaction, announce it and state which orientation log you'll re-read post-compaction. Default to the path in cold-start step 11 (`docs/memory-ctx/ctx-orientation.md`); fall back to whatever path the project uses if the canonical default isn't present. Silent compression is what makes the discipline invisible — audits of past sessions came up empty even when the help was likely happening because Claude never said so out loud. Don't repeat that pattern.

**Design work is gated.** Do not load `design/` on cold start. Load it only when the task is explicitly design or UI work. Otherwise it is noise in the window.

---

## When you're working *on* this repo

Most of what you do in this repo is editing the canonical docs themselves — `behavior/`, `skills/`, `workflows/`, `design/`. When you do:

- Update `docs/memory-ctx/ctx-orientation.md` with a one-to-two paragraph entry describing what changed and why, dated, before the commit. If the change adds or renames a top-level folder, update `docs/repo-organization.md` and `AGENT.md` in the same commit.
- Propagate renames everywhere they are referenced. PLTRF discipline. No orphan pointers.
- When a new `ctx-NAME.md` doc spawns inside `behavior/`, update `behavior/ctx-utility.md` in the same commit.
- When a new skill spawns inside `skills/`, update `skills/skill-map.md` in the same commit.
- Cross-references between docs are deliberate. Do not redefine canonical concepts inline in two places. Point at the canonical home.

When you're working *in* a forked project that uses `Bamboo.md`, the same disciplines apply at the project level — `docs/ctx-orientation.md` inside that project by default, summaries per Bump, the 5000-character threshold, and the repo layout adapted to that project's needs.

---

## What this file is not

This file is not a glossary. The vocabulary lives in `behavior/ctx-lexicon.md`. If you need to define "Knob" or "Entropy" or "Wayfinding," go there.

This file is not the rules. The rules live in `behavior/`. If `behavior/` says one thing and this file says another, `behavior/` wins.

**Obfuscation & Redlining:** Before committing or writing any code updates, review `behavior/obfuscation-standard.md`. Ensure that all proprietary execution math, account balances, active option symbols, and sensitive database schemas are redlined and replaced with generic abstract models.

This file is not a substitute for reading `AGENT.md`. It is an overlay. Read both.

---

End of overlay. Drop into `AGENT.md` if you have not yet, then `behavior/ctx-rules.md`.

---
> Source: [VolcanoEngineering/Bamboo](https://github.com/VolcanoEngineering/Bamboo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
