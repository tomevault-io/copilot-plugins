## ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is **not an application** — it is a library of portable **AI-assistant tooling artifacts** (skills, rules, hooks, agents) authored to be **copied into a consuming Java project's `.claude/` directory**. There is no application build here. Every artifact targets the same stack: **Java 25 + Spring Boot 4** (Spring Framework 7, Maven, Jackson 3, JSpecify). All prose for end users is in **Brazilian Portuguese**; meta-docs (READMEs, hook comments) are in English.

The four artifact types and how a consuming repo loads them:

| Type | Folder | Installed to (target repo) | Loaded |
|---|---|---|---|
| Skills | `skills/<name>/` | (consumed by the agent host, e.g. Copilot/Claude) | On demand — triggered by task |
| Rules | `rules/<name>.md` | `.claude/rules/` | Always-on baseline |
| Hooks | `hooks/<name>/` | `.claude/hooks/` + merge `settings.example.json` into `.claude/settings.json` | Agent lifecycle events |
| Agents | `agents/<name>.md` | `.claude/agents/` | Invoked for bounded tasks |

## The artifacts form one coherent system — keep them in sync

The central invariant: **`rules/java-standards.md` is the source of truth**, and the other artifacts reference or enforce it. When you change a standard, propagate it:

- **Skills** generate code that must obey `java-standards` (records for DTOs, self-constructing objects, no MapStruct, restricted Lombok, sealed interfaces, domain exceptions, etc.).
- **`hooks/java-standards-guard`** is the *hard* enforcement of the high-confidence subset of that rule — it blocks (exit 2) `@Data`, MapStruct/`@Mapper`, `System.out/err`/`printStackTrace`, and generic `throw new RuntimeException(...)` before a `.java` write hits disk. If you add a bannable, low-false-positive pattern to the rule, add it here too.
- **`agents/spring-java-reviewer.md`** reviews diffs against `java-standards` *plus* the relevant skill and its `references/`. It is read-only — it reports, never edits.
- **`hooks/java-build-check`** is a `Stop` hook (not per-edit): runs `mvn` once per turn (`$MVN_GOALS`, default `-q -B test`), silent on success, exit 2 + truncated errors to block the stop on failure. Loop-guarded via `stop_hook_active`. Heavy verification belongs here, not in a per-edit hook.

## Anatomy of a skill (the main thing you'll author)

```
skills/<name>/
  SKILL.md          # YAML frontmatter (name, description) + the instruction body
  references/*.md    # deep-dive topics, loaded only when the SKILL.md routes to them
  evals/evals.json   # output-quality cases: {prompt, expected_output, files}
```

Authoring conventions, learned from the existing four skills:

- **The frontmatter `description` is the trigger contract** — it is long and deliberate, listing concrete user phrasings (including plain-language ones) that *should* fire the skill **and an explicit "NÃO acione para…" (do not trigger for) list** to fence it off from sibling skills. Treat this as the highest-leverage text in the skill; it is what gets optimized (see workspaces below).
- **Keep `SKILL.md` lean; push depth into `references/`.** The body has a routing table ("Quando … / Leia …") that tells the agent which reference file to read for advanced topics. Don't inline what belongs in a reference.
- Skills open with a **"Stack definida"** table and a **"Passo 0 — Perguntar antes de gerar"** section: ask the user for the specifics (base-url, endpoints, DTOs, auth) before generating, with documented defaults for unattended runs.
- Skills cross-reference each other (e.g. the HTTP-client skill defers logging to `spring-java-log-standardization`, fine resilience policy to a Resilience4J skill). Preserve these handoffs instead of duplicating.
- `spring-java-http-client.skill` is a **packaged zip** of the skill directory — a build output, not a source file to edit. Edit the directory, not the `.skill`.

## Evals and the workspaces

Skill quality has **two axes**, each with its own eval format:

1. **Triggering** — `*-workspace/trigger-eval.json`: a list of `{query, should_trigger}` cases that test whether the `description` fires on the right prompts and stays silent on the wrong ones.
2. **Output quality** — `skills/<name>/evals/evals.json`: `{prompt, expected_output, files}` cases describing what the generated code must (and must not) contain.

The `*-workspace/` directories (e.g. `spring-java-http-client-workspace/`) are **untracked scratch areas** produced by running/optimizing skills — description-optimization runs (`desc-opt*`), timestamped iterations, `grade.py`, HTML reports, logs. They are working artifacts, not part of the shipped library; don't treat them as source and don't commit them unless asked.

Use the **`skill-creator`** skill to create, edit, optimize, run evals, and benchmark skills — that is the supported workflow for the eval/optimization loop, rather than ad-hoc scripts.

## Commands

There is no project-level build or test here. Verification commands live inside the artifacts and run **in the consuming repo**, against its `pom.xml`:

- `hooks/java-build-check/java-build-check.sh` → runs `mvn $MVN_GOALS` (default `-q -B test`; override up to `-q -B clean install`). No-ops when there is no `pom.xml`. Requires `jq`, Maven, a JDK on PATH.

---
> Source: [Joao-Victorsg/ai](https://github.com/Joao-Victorsg/ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
