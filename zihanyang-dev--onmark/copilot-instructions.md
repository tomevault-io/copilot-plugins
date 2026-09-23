## onmark

> Onmark is a screenplay-first, browser-rendered video compiler and execution engine. Rust owns deterministic compilation and native execution; TypeScript owns authoring and the browser runtime.

# Onmark

Onmark is a screenplay-first, browser-rendered video compiler and execution engine. Rust owns deterministic compilation and native execution; TypeScript owns authoring and the browser runtime.

**Current phase:** delivery gates one through nine and distributed incremental
capture reuse are complete. New language or pipeline work requires a separately
admitted gate backed by real authoring, correctness, and performance evidence.
Deployment work remains frozen: do not add a remote authoring command,
coordinator, database, queue, lease system, cloud workflow, infrastructure
definition, or another provider adapter. Do not add an interactive player,
preview server, Studio surface, source-mutation API, inferred temporal or visual
separability, or a hidden pixel-path fallback. New screenplay spelling,
including transitions and media-treatment attributes, still requires the
language-admission evidence defined below.

## Read before changing code

Read only the documents relevant to the task:

| Task | Required document |
| --- | --- |
| Elements, attributes, timing, cue semantics, diagnostics, language evals | `docs/en/language-specification.md` |
| New crate/package, dependency, process boundary, render pipeline, cache, deployment | `docs/en/architecture.md` |
| Any Rust implementation or review | `docs/en/rust-style-guide.md` |
| Any TypeScript/JavaScript implementation, generator, or review | `docs/en/typescript-style-guide.md` |
| Fuller Chinese rationale for a decision already mirrored in English | matching file under `docs/zh-CN/` |

When code and design documents disagree, stop and surface the conflict. Do not silently choose one.

## Architectural red lines

1. Source expresses intent; versioned IR records facts.
2. The compiler core is pure: no filesystem, network, wall clock, Chromium, FFmpeg, or cloud SDK.
3. TypeScript does not reimplement timing or planning. Rust does not reimplement DOM/CSS/WebGL.
4. Local and distributed execution consume the same Render Unit contract and render executor.
5. Unknown browser components are sequential by default. Random seekability must be proven.
6. Partitions follow render dependencies, not blindly chosen shot boundaries.
7. Authored mistakes are structured diagnostics; infrastructure faults are typed errors.
8. Expected queues, subprocesses, memory, and temporary storage are bounded.
9. No `utils`, `shared`, or `common` dumping-ground module, crate, or package.
10. AWS, queue, database, and vendor types do not enter `onmark-core`.

## Crate and package split rule

Start with a module. Split a crate/package only when at least one is real:

- a different runtime environment;
- a different dependency budget;
- an independent external consumer;
- a different deployment or release artifact.

Every split must document which criterion it satisfies, its allowed dependencies, and its consumers. Code volume and hypothetical reuse are not reasons.

The engine and desktop product use four Rust crates:

- `onmark-core`: pure compiler, domain model, diagnostics, IR, protocol;
- `onmark-media`: asset probing and normalized metadata without Chromium;
- `onmark-render`: browser/FFmpeg control and local unit executor;
- `onmark-cli`: arguments, terminal presentation, and composition root.

Do not create coordinator, Lambda, render-graph, planner, syntax, or diagnostics crates merely because those concepts have names. Add modules first. The Lambda deployment surface appears at delivery gate three and wraps `onmark-render`; it never forks the engine.

`onmark-core` module dependencies are enforced as a DAG: model is foundational; syntax/diagnostics/timeline may depend on model; render_graph may depend on model/timeline; compiler may depend on model/syntax/diagnostics/timeline; protocol may depend on model/diagnostics/timeline. Domain modules never depend on protocol. New edges require an architecture-doc change and dependency-law test update.

`@onmark/runtime` is below authoring and bundler. Runtime owns frame hooks and temporal capability declarations. Authoring may import its types-only entrypoint; runtime never imports authoring or bundler.

Rust wire types generate checked-in schemas and TypeScript codecs. Generated files are never hand-edited; CI regeneration must produce no diff.

## Working conventions

- Code, identifiers, comments, public docs, and commit subjects use English. Chinese mirrors may carry fuller early design discussion.
- Rust follows `docs/en/rust-style-guide.md`: rectangular functions, tree-shaped modules, linear pipelines.
- TypeScript and JavaScript follow `docs/en/typescript-style-guide.md`: block-shaped control flow, explicit effects, bounded async work.
- A language behavior change updates the language specification and adds conformance fixtures.
- A pipeline or package-boundary change updates the architecture document.
- The first step of a bug fix is a failing focused test or fixture.
- Generated protocol code and golden artifacts are never edited manually.
- Keep commits small and use imperative English subjects.

## Definition of done

A change is done when formatting, linting, unit tests, and relevant conformance fixtures pass; public behavior has documentation; dependency direction remains valid; expected failures remain structured; and no later delivery gate was implemented incidentally.

---
> Source: [zihanyang-dev/onmark](https://github.com/zihanyang-dev/onmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
