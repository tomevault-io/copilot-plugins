## evolving

> Conventions for maintaining this wiki. Read before editing any file under `wiki/`.

# wiki/CLAUDE.md

Conventions for maintaining this wiki. Read before editing any file under `wiki/`.

The root `CLAUDE.md` describes the project; this file describes how we keep the project's documentation healthy as it grows. For the current list of pages, see [`index.md`](./index.md) — keep it in sync whenever you add or rename a page.

## Principles

The wiki is a **living knowledge base** — the project's current model of itself — not a historical archive of past ideas. Two failure modes to avoid:

- **Fossilization.** Dated filenames, "v1" / "v2" copies, append-only specs. Git already stores history; the wiki stores *what is true now*.
- **Fragmentation.** Ten micro-pages that drift apart. A single well-organized page beats a directory of stubs.

Everything below is in service of those two principles.

## Conventions

### 1. Edit in place. Do not append dated copies.

If the design changes, update `architecture.md` or `specification.md` directly. Do not create `architecture-2026-05.md` or `specification-v2.md`. The git log is the history; the wiki is the present.

### 2. Read the full page before editing it.

Before changing a wiki page, read it end to end. The spec is dense and cross-referential — local edits that contradict a distant section are a common failure mode. Non-negotiable for any substantive edit.

### 3. Capture non-obvious choices as ADRs under `decisions/`.

When you make a design decision that a future reader could reasonably question ("why didn't you do X instead?"), add an ADR under `wiki/decisions/`. Format and when-to-write rules are in `wiki/decisions/README.md`.

**The split:** core wiki pages describe *what the system is and how it works*; ADRs describe *why we chose this approach over the alternatives*. A reader asking "how does X work?" should always find the answer in a core page, not in an ADR.

Two directions this rule cuts:

- **Don't bury decision rationale in core pages.** Alternatives and rejected reasons belong in ADRs, not scattered through `architecture.md`, `specification.md`, or other core pages.
- **Don't use ADRs as primary documentation.** If an ADR's "Decision" section is describing the chosen thing in more than a sentence or two — invocation shapes, full APIs, module layouts — that content belongs in a core page. The ADR states what was chosen and links out.

Concrete test for the second direction: if you deleted the ADR file tomorrow, would a contributor still be able to understand how the system works by reading the wiki? If no, the ADR is carrying load that belongs in the wiki.

### 4. Default to extending existing pages. Promote to a new page only when earned.

Pre-decomposition is the failure mode — creating `worker.md`, `gatekeeper.md`, `safety-model.md` before any of them has enough content to stand alone. Fragmentation is harder to unwind than consolidation.

Start with a section inside `architecture.md` or `specification.md`. Promote that section into its own wiki page **only when at least one of these is true**:

- **The section is drowning its host page.** A subsection has grown past roughly 20% of the parent page, or has significantly more depth than sections around it.
- **Multiple other sections link into it.** Three or more places reference "see the X discussion below" — X has earned its own page and a stable link target.
- **A genuinely new concept arrives.** A new agent role, a new subsystem, a new top-level product surface. Not a refinement of something that already exists.

**Concrete-noun test:** can you complete "X is a ___" with a non-generic answer? "Retrospector is a read-only agent that proposes prompt edits from log patterns" → yes, page-worthy. "Good error handling is important" → no, not page-worthy.

**No pre-created directories.** The wiki stays flat (`wiki/*.md`) until flat stops working. Don't invent `wiki/agents/` before there's enough per-agent content to warrant it. Taxonomy emerges from the material; don't impose it in advance.

**Whenever you add, rename, or remove a page, update [`index.md`](./index.md) in the same edit.** The index is the navigation layer — a page not in the index is effectively invisible. If you forget, the next page-add will compound the drift.

### 5. Don't create empty pages.

If you can't write at least three meaningful sentences about a topic, don't create a page for it. A stub with a `TODO` is worse than a missing page: the stub gets indexed and read; the missing page prompts you to add it where it belongs. Applies to ADRs too — don't file an ADR for a decision that's still in flux.

### 6. Cross-reference, don't duplicate.

If `architecture.md` describes principle #5 and `specification.md` needs to refer to it, **link to it** — don't restate it. Duplication causes drift: when one copy updates and the other doesn't, the wiki lies.

### 7. Write in the present. Don't narrate refactors.

The wiki describes the system *as it is now*, not *how it got here*. After a refactor, the core pages should read as if the removed concept never existed. A fresh reader should not be able to tell whether a concept was removed yesterday or never existed.

Anti-patterns to delete on sight in `architecture.md`, `specification.md`, `README.md`, and the root `CLAUDE.md`:

- "X was removed — see ADR NNN."
- "The original design had Y; we now do Z."
- `~~struck-through~~` items in any list.
- "Previously…", "No longer…", "Used to be…", "Replaced by…" framing.
- A non-goal bullet whose removal would be the news ("No multi-tag runs — v2 only" after multi-tag ships). Just delete the line.

History belongs in the git log and in ADRs (especially their Context sections). ADRs are the one exception to this convention — narration of "what changed and why" is precisely what they're for.

**On ADR cross-links from core pages.** This rule does *not* contradict Convention #6. A link to an ADR is legitimate when it points at the *why* behind a currently-true statement — the surrounding prose already makes sense without clicking through. A link is a tombstone when the prose only makes sense as a reference to the removed concept.

Discriminator: delete the link and re-read the paragraph. If it reads as normal present-tense description, the link was a legitimate rationale reference. If it now reads as dangling or evasive, the link was carrying the narration and the paragraph still needs rewriting.

## Chunk completion checklist

Run this at the close of every implementation chunk. The whole point of the checklist is to keep the wiki in sync with `src/` *as work lands*, not as a quarterly cleanup. A wiki that is stale by even a few days erodes trust faster than it can be rebuilt — every fresh agent who reads a fossilized claim and acts on it makes the next agent doubt the wiki by default.

Do these in order. Do them in the same branch as the chunk's code, not in a follow-up.

1. **Update [`status.md`](./status.md).** Move every newly-built thing from "Not yet built" to "Built", with a file pointer (e.g., `src/schema/agent-output.ts`). If a file pointer no longer points at a real file, the page is wrong — fix the page, not the rule. This is the most important step: `status.md` is the page a fresh agent reads to know where the project is.
2. **Update the relevant core pages** (`architecture.md`, `specification.md`, `runtime.md`). Edit in place per Convention #1. Apply Convention #7 — write the new state in present tense; delete obsolete prose, don't narrate that you deleted it.
3. **Add ADRs for non-obvious decisions made during the chunk.** Format per [`decisions/README.md`](./decisions/README.md). Update [`decisions/README.md`](./decisions/README.md) with the new ADR's index entry in the same edit.
4. **If a new page was added or removed, update [`index.md`](./index.md)** (per Convention #4).
5. **Discard intermediate scratch docs.** Anything under `docs/superpowers/`, `docs/plans/`, brainstorm files, or other ignored scratch (per the root `CLAUDE.md` working rule "intermediate docs are scratch") gets deleted from the working tree. Their durable content has already been promoted by steps 1–4.
6. **Resolve cross-reference drift.** Search the touched pages for links and confirm each target still exists. A broken `./decisions/00X-foo.md` reference is the most common rot signal.
7. **Sweep for drift — any duplicate fact, anywhere.** Drift isn't only `src/` ↔ wiki; it's any place where the same fact lives in two locations and one can rot independently of the other. Three classes to check at chunk close:
    - **`src/` ↔ wiki.** For each module the chunk touched, the wiki section that describes it should be readable as a description of the code that's actually there.
    - **Intra-`src/`.** A constant in one file that mirrors a value in another file (e.g. a hardcoded version in CLI code that mirrors `package.json`); a type definition copy-pasted instead of imported. Single-source-of-truth or import; never both.
    - **`src/` ↔ config.** Constants in code that mirror values in `package.json`, `tsconfig.json`, `biome.json`, `bunfig.toml`, etc. The config is canonical; code reads from it.

    If you find a divergence you don't have time to fix, write it down at the bottom of `status.md` under "Known drift" — never silently leave two facts disagreeing.

A chunk is not complete until this checklist is. If a fresh agent picking up the project tomorrow couldn't reconstruct what's built, what isn't, and where to start next from `index.md` + `status.md` + the touched core pages, the chunk is unfinished — regardless of whether the code is merged.

## Wiki health (aspirational)

As the project grows, these become worth enforcing:

- **Cross-reference minimum.** Substantive pages should link to at least three other wiki pages. Isolated pages that no other page links to are suspicious.
- **Health checks.** Periodically read the wiki end to end, looking for orphan sections, stale claims, and contradictions. Treat this like a code review of the docs.
- **No chronological dumping.** Organize pages by concept (Worker, Safety Model, History Summarization), not by when things were added. The git log is chronological; the wiki is thematic.

## What does *not* live in the wiki

- **Conversation notes, brainstorms, exploratory drafts.** These belong in git commit messages, or nowhere. If a brainstorm produces a real design decision, extract it into an ADR — don't check the brainstorm in.
- **Tutorials, quickstarts, user guides.** These go in `README.md`. The wiki is the project's internal model of itself; the README is for external users.
- **Runtime state, logs, or output.** The agents produce `evolving.log.jsonl` and `.evolving/retrospectives/` at runtime. Those are not documentation.

## If you get stuck

If a wiki page says one thing and `src/` says another — and there's no ADR explaining the gap — that's a drift signal. Flag it to the human rather than picking a side silently.

---
> Source: [JINGBANZ/evolving](https://github.com/JINGBANZ/evolving) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
