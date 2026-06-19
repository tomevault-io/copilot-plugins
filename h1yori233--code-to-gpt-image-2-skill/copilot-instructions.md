## code-to-gpt-image-2-skill

> Generate a structured GPT Image 2 prompt JSON from any codebase. Use when Codex needs to read a repository, source file, app, CLI, library, or architecture and produce a prompt.json for a technical infographic, architecture diagram, pipeline explainer, module map, data-flow diagram, or educational codebase visualization.


# Codebase To GPT Image 2 Prompt

Turn a codebase into a structured GPT Image 2 prompt JSON for a high-quality technical infographic. The goal is not to document every file or wrapper; the goal is to extract the most important mechanism in the code and encode it as controllable image-generation JSON.

## Workflow

1. **Resolve scope**
   - Use the provided folder, file, GitHub repo, or current working directory.
   - If the user names a specific entry point, treat it as the anchor.
   - If no entry point is named, infer it from README, package/project config, CLI files, routes, app bootstrap files, or common framework conventions.

2. **Read strategically**
   - Start with README or docs, dependency/config files, and primary entry points.
   - Treat README/docs as orientation only: scope, terminology, supported modes, documented defaults, and user-facing examples. Unless the user explicitly asks for a README-only diagram, verify P0 mechanics from implementation files.
   - Use `rg` / `rg --files` to locate important modules, commands, routes, classes, and public APIs.
   - Prefer targeted batches of 2-5 files. Do not bulk-load a large file set and then summarize everything at once.
   - Read past wrapper code into the downstream modules that do the real work.
   - Classify files and functions as **core mechanism**, **orchestration**, **boundary adapter**, or **runtime glue**.
   - Read enough core code to explain what the system transforms, which components do the transformation, and why the design is interesting.
   - After finding the likely P0 mechanism, perform a second depth pass on the implementation files behind it. Do not stop at the entry-point call site.
   - For tensor, media, ML, compiler, database, or protocol code, derive a concrete worked example when the user gives a scenario or the repo has a clear default. Include formulas and numeric values, not only symbolic shapes.
   - Do not exhaustively analyze unrelated implementation details unless they affect the visual story.

3. **Build an internal evidence ledger**
   - Maintain the ledger internally during the task. Do not save `evidence.json`, do not make the ledger the final answer, and do not stop after code analysis.
   - First pass: build `source_map` from docs, configs, entry points, and quick searches. Record P0/P1/P2 candidates and the next implementation files to read.
   - Second pass: read only the selected P0 implementation files in small batches. After each batch, update `mechanism_cards` with claim, source, priority, `importance_score`, `why_it_matters`, `visual_budget`, `compression_strategy`, visual role, and unresolved questions.
   - Third pass: for numeric systems, update `shape_ledger` with scenario, formula, concrete value, source, and `attach_to_panel`. Keep only numbers that explain representation changes, compute scale, state flow, cache/chunk behavior, or output shape; not every shape deserves visible space.
   - Track data lineage in the ledger. When a later tensor, state, or object uses an earlier value, record the producing panel and consuming panel.
   - Before writing the final storyboard, perform a `budget_pass`: move canvas from low-value setup, input branching, repeated formulas, and easy arrows toward core loops, model architecture, state updates, and decode/materialization.
   - Stop exploring when the ledger is sufficient to draw the core mechanism. Then generate the prompt JSON from the ledger.

4. **Rank information by visual importance**
   - **P0 - Core mechanism**: domain concepts, algorithms, model architecture, data transformations, state machines, request/data flow, core loops, storage semantics, rendering logic, or protocol behavior.
   - **P1 - Supporting architecture**: orchestration branches, component boundaries, adapters, external services, persistence, output creation, extension points, and failure handling.
   - **P2 - Runtime glue**: CLI argument parsing, environment setup, logging setup, process rank setup, config defaults, file naming, and generic save/load code.
   - Build the infographic from P0 and P1. P2 should usually be omitted; if it appears, it may only be a tiny caption, footer, or compact callout.
   - Never spend the first 2-3 main panels on CLI/runtime setup unless the codebase is itself a CLI framework or runtime tool.
   - Allocate more space to mechanisms with hidden complexity, state mutation, repeated execution, performance/quality impact, or representation changes.
   - Compress details that are simple input branching, routing, default choices, repeated formulas, or one-step combinations that an arrow can explain.
   - For each panel, ask: if this panel were deleted, would the viewer still understand the core mechanism? If yes, shrink it, merge it, or turn it into an inline callout.
   - For ML/video/diffusion systems, conditioning, input setup, and runtime setup together should usually use only 15-25% of the canvas. Denoising/model architecture/decode should usually use at least 70%.
   - For diffusion systems, CFG and schedule equations rarely deserve standalone panels. Prefer curves, mixers, and state arrows unless the formula itself is the novel idea.
   - For model-centric code, a DiT/Transformer/engine/compiler core should be a real architecture cutaway with its internal blocks, not a thumbnail.
   - For decode/materialization code, expand it when it contains real reconstruction logic such as mean/std transforms, causal caches, chunking, overlap, tiling, rendering, or clamping. Name only behaviors verified in code.
   - When the image output varies too much across generations, reduce degrees of freedom with an explicit `layout_contract`: fixed zones, percent ranges, fixed order, and forbidden visible extras.
   - If one core mechanism is more central than another, give it a larger fixed zone. For example, when DiT architecture is the core of a video diffusion diagram, it must be larger than the denoising state loop.

5. **Choose the dominant core**
   - Select one dominant P0 mechanism that should be the visual hero.
   - Allocate the largest region of the image to that mechanism. For algorithmic, ML, compiler, rendering, database, protocol, or agentic codebases, the dominant core should occupy 45-65% of the main diagram.
   - Supporting inputs, routing, loading, and outputs should orbit the dominant core instead of consuming equal-height panels.
   - If the dominant core is a loop, model, state machine, scheduler, transformer, attention block, compiler pass, renderer, query engine, or planner, expand it into subcomponents. A single box labeled with the core name is not sufficient.
   - Use fields such as `focus_area`, `visual_weight`, or `expanded_details` in the JSON when helpful to force GPT Image 2 to allocate space correctly.
   - For complex technical diagrams, use a `layout_contract` as well as `visual_weight`. Specify top-to-bottom zones, each zone's intended canvas share, and which zones must be larger than others.
   - Define content invariants for the dominant core. These are mandatory visible nodes that must not be dropped, merged away, or replaced with generic labels.

6. **Extract the visual story**
   - Identify the system purpose in one sentence.
   - Identify the main mechanism journey: source/input artifact -> representation -> core transformation -> output artifact.
   - Identify the main actors: entry point, orchestrator, services, models, data stores, adapters, UI/API layers, background workers, or external systems.
   - Identify notable mechanics: validation, configuration, branching modes, caching, async/concurrency, distributed execution, model inference, persistence, retries, error handling, or output serialization.
   - Identify 3-6 high-signal insights, then attach each insight directly to the panel where it is produced or consumed.
   - Default to no right-side insight sidebar. Use only inline callouts, shape badges, lineage arrows, compact legends, and tiny footnotes.
   - Use an `insight_sidebar` only if the user explicitly requests a sidebar or a separate notes column.
   - When a formula can be communicated by a graph, loop arrow, mixer, or data-flow join, prefer the visual form. Keep equations as tiny labels only when they clarify a non-obvious transformation.
   - Default to no visible legend, source footer, or source path captions. Put color/category meaning directly in the relevant labels, and keep `source_hint` as machine-readable grounding only.

7. **Choose a diagram pattern**
   - **Pipeline**: best for CLIs, inference scripts, ETL, compilers, request handlers, build systems, and workflows with clear stages.
   - **Architecture map**: best for web apps, services, monorepos, API systems, and multi-component projects.
   - **State/lifecycle diagram**: best for event loops, protocols, agents, jobs, media processing, and long-running systems.
   - **Module atlas**: best for libraries, SDKs, frameworks, and codebases where public modules are the learning target.
   - **Debug/ops board**: best when failure modes, deployment, performance, or runtime configuration are the most important story.
   - When the entry point is mostly a wrapper, make the diagram pattern follow the deeper subsystem, not the wrapper shape.

8. **Build the prompt JSON**
   - Read `references/prompt-json-schema.md` when designing the JSON.
   - Produce valid JSON, not Markdown, when the user asks for a file.
   - Do not set `subject.codebase_name`, title, or subtitle to a README/docs workflow unless the user explicitly requested documentation visualization.
   - Include `worked_example` / `shape_trace` when concrete shapes, counts, schedules, cache sizes, or loop counts can be derived.
   - Mirror every `worked_example.shape_trace` value in the relevant `main_diagram` panel via `shape_badges`, `inline_callouts`, or `lineage_links`.
   - Put formulas and numbers near the visual object they explain. Do not put the only concrete numbers in a detached table or sidebar.
   - Encode the `budget_pass` result in the JSON through `visual_weight`, `focus_area`, `layout`, and panel content. Do not give equal space to every true fact.
   - Include `layout_contract` for complex diagrams where stable composition matters.
   - Keep `source_hint` fields for grounding, but explicitly forbid rendering source paths as visible text.
   - Omit visible legends unless the user asks for a legend. If a legend is truly needed, make it tiny and never a major region.
   - Keep the JSON visual and instruction-oriented. It should tell GPT Image 2 what to draw, not become a long code review.
   - Prefer short labels, numbered sections, clear arrows, modules, callouts, legends, and negative constraints.
   - Include only source-grounded claims. If a point is inferred, phrase it generically or mark it as a visual simplification.

9. **Validate before delivery**
   - Parse the JSON or visually inspect syntax for valid quotes, commas, and brackets.
   - Check that all title and label text is concise enough to render.
   - Check that no module names, command names, or file names are hallucinated.
   - Check that the layout has a clear first-view hierarchy: title, main flow/map, inline callouts, and a tiny legend if useful.
   - Check that `negative_prompt` prevents generic AI imagery, unreadable microtext, and irrelevant decorative elements.
   - Check that at least 80% of the main diagram describes core mechanism or supporting architecture, not setup code.
   - Check that the dominant core is the largest visual region.
   - Check that `layout_contract` exists for complex dense infographics and that its area rules match the stated visual priorities.
   - Check that there is at most one P1 setup/routing/loading section before the first P0 section.
   - Check that every P0 module with substantial internal behavior includes 3-7 `expanded_details` or equivalent sub-elements.
   - Check that numeric shape/schedule traces are present when the code exposes enough formulas to derive them.
   - Check that each numeric trace has `attach_to_step` and appears inline in that step's panel.
   - Check that every later use of a variable, tensor, state, or object has a visible lineage from its producer.
   - For ML/video/array systems, check for details such as timestep/sigma schedule, conditioning/modulation, patchify purpose, token count, encoder/decoder stride, and chunk/cache/overlap behavior when applicable.
   - Check that conditioning/input/setup do not take over the page when the model loop, model architecture, or decode path is the harder idea.
   - Check that formulas are not treated as the main story when a visual mixer, schedule curve, state arrow, or architecture diagram would explain the mechanism better.
   - Check that decode/materialization is not reduced to a save/output box when code contains meaningful decoder, cache, chunk, normalization, or reconstruction logic.
   - Check that P0 `source_hint` values point to implementation files, classes, or functions. README/docs may support defaults and examples, but must not be the only evidence for the core mechanism.
   - Check that the largest panels are not downloads, model-choice tables, CLI commands, prompt-extension services, examples, or frontends when deeper implementation code is available.
   - Check that there is no right-side insight sidebar, detached worked tensor trace, or notes column unless explicitly requested.
   - Check that there is no visible source footer, source path caption, legend box, or mux legend unless explicitly requested.
   - Check that `source_hint` is treated as grounding metadata, not visible text.
   - Check that all content invariants for the dominant core are present. For Wan2.1-style DiT diagrams, missing AdaLN/time modulation is a factual failure.
   - Check that the title names the system's interesting mechanism, not just the filename or CLI command.

## Output Rules

- Default output name: `prompt.json` when the user asks to save a file.
- If the user asks only for analysis, provide the JSON inline first, followed by a short rationale.
- Use the user's language for explanatory prose, but use whatever language best fits the requested image labels. For codebase diagrams, English labels are usually safest unless the user requests Chinese.
- Use exact code identifiers sparingly. Favor human-readable labels, with code identifiers in small captions only when useful.
- Do not include huge code snippets in the image prompt. Convert code behavior into diagram modules.
- Do not run expensive builds, model downloads, training, or generation commands unless the user explicitly asks. Static code reading is usually enough.
- Do not make CLI flags, runtime initialization, logging, or default parameters the hero unless the user specifically asks for an operations/runtime diagram.
- Do not stop after presenting the internal evidence ledger. The deliverable is still the requested prompt JSON.
- Do not ask GPT Image 2 to render file paths, source footers, or source captions by default.

## Anti-Patterns

Avoid these common failures:

- **Wrapper worship**: over-explaining `argparse`, bootstrapping, logging, env vars, or file naming because they appear near the entry point.
- **Default dumping**: filling sidebars with sampling steps, config defaults, ports, flags, or dimensions without explaining why they matter.
- **Flat call graph posters**: drawing every invoked function as equal instead of emphasizing the domain transformation.
- **Filename-first titles**: titles like "`generate.py` Flow" when the real story is model conditioning, request routing, compilation, rendering, indexing, or state transitions.
- **Source caption clutter**: placing long file paths everywhere. Use tiny source hints only for important anchors.
- **Core as a label**: naming the important subsystem but not explaining its internal mechanics.
- **Equal-weight pipeline**: giving input parsing, loading, core algorithm, and saving the same visual weight.
- **Implementation trivia sidebar**: using the sidebar for facts that are technically true but do not help the viewer understand the dominant core.
- **Generic insight title**: using "Implementation Insights" for any sidebar, which invites random facts instead of a curated mental model.
- **README workflow poster**: turning public docs, download choices, examples, or command invocations into the main story when the codebase contains deeper implementation mechanics.
- **Detached sidebar**: placing important mechanics in a separate right-side notes column instead of attaching them to the flow nodes they explain.
- **Detached tensor trace**: putting shapes or formulas only in a worked-example table far from the tensors that produce or consume them.
- **Broken lineage**: using derived values such as `y`, `cache`, `tokens`, or `state` later in the diagram without a visible producer-to-consumer connection.
- **Conditioning takeover**: letting input branches, T2V/I2V lanes, prompt encoders, or setup consume the canvas that should explain the core model/loop/decode.
- **Formula worship**: making CFG, schedules, or equations large panels when a curve, mixer, or state-transition diagram explains the idea better.
- **Architecture thumbnail**: showing a model, DiT, compiler, renderer, or engine as a tiny box instead of a real internal architecture when it is the hard part.
- **Decode afterthought**: reducing a meaningful decoder/materializer to a save/output box despite cache, chunk, normalization, reconstruction, or rendering logic in code.
- **Layout drift**: relying on vague visual weights when the output needs stable area allocation across generations.
- **Visible source clutter**: drawing source paths, citations, source footers, or legend boxes that do not teach the core mechanism.
- **Missing invariant**: omitting mandatory architecture facts such as AdaLN/time modulation in a DiT diagram.

## Evidence Discipline

When reading a codebase, keep a small internal evidence map:

- product/system purpose
- `source_map`: entry points, docs/config defaults, core implementation files, and key functions/classes/modules by P0/P1/P2
- `mechanism_cards`: claim, source, priority, `importance_score`, `why_it_matters`, `visual_budget`, `compression_strategy`, visual role, and panel candidate for each important mechanism
- `shape_ledger`: scenario, formula, value, source, and `attach_to_panel` for concrete shapes, counts, schedules, cache/chunk behavior, and output shapes that explain representation changes or compute scale
- `lineage_links`: producer panel -> derived value -> consumer panel
- `budget_pass`: final decision about what deserves hero, large, medium, small, or tiny space and what should be merged or omitted
- `layout_contract`: fixed zones, percent ranges, relative area constraints, forbidden visible elements, and mandatory content invariants
- dominant core and its internal subcomponents
- main data/control flow and representation changes
- source priority: implementation evidence for P0, docs/README evidence only for scope, defaults, examples, or explicit documentation diagrams
- external dependencies and integrations
- output artifacts
- constraints, bottlenecks, tradeoffs, and failure modes

Use this evidence to generate the prompt JSON. The ledger is an internal staging tool, not a separate output file. The final image prompt should be aesthetically directed and source-grounded, not a generic software poster.

## Reference Files

- `references/prompt-json-schema.md` - JSON schema, field meanings, diagram pattern guidance, and validation checklist. Read this before writing or saving the final `prompt.json`.

---
> Source: [H1yori233/code-to-gpt-image-2-skill](https://github.com/H1yori233/code-to-gpt-image-2-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
