## comnix

> Comnix is a Kotlin Multiplatform and Compose multi-project workspace for reusable graphics, image,

# AGENTS.md

## Mission

Comnix is a Kotlin Multiplatform and Compose multi-project workspace for reusable graphics, image,
resource, and SVGA components with Android and iOS demos.

Keep shared APIs platform-neutral, keep engine implementations behind typed runtime contracts, and
do not trade runtime performance for implementation convenience.

## Reading Strategy

Read this file first. Load only the documentation required by the task:

1. For code changes under `libs/*`, `apps/*`, `plugins/*`, `benchmark`, or root Gradle files, always
   load and use `akit-repo-standards`.
2. For non-trivial architecture or module-boundary work, also read
   `.agents/skills/akit-repo-standards/references/architecture-and-conventions.md`.
3. For coroutine, Flow, rendering, decoding, loading, animation, or other performance-sensitive
   work, read `.agents/harness/performance-coding-guidelines.md`.
4. For versioning, artifact publication, changelog, tags, or GitHub Releases, read
   `.agents/harness/release-process.md`.
5. For class, function, field, and complex-branch comment style, follow `AGENTS_CODE_DEMO.kt`.
6. Before editing a subtree, read any more specific `AGENTS.md` found in that subtree.

Lightweight read-only tasks such as locating a symbol, explaining an existing code path, or checking
Git status do not require unrelated reference documents.

## Project Map

- `libs/graph`: graphics, NinePatch, Lottie painters, shadows, and native toolkit integration.
- `libs/image/image`: engine-agnostic Compose image API.
- `libs/resource/runtime`: typed resources plus shared image and binary request contracts.
- `libs/resource/loader-engine-coil`: Android/iOS Coil request engines.
- `libs/resource/loader-engine-glide`: Android Glide request engines.
- `libs/resource/gradle-plugin`: `cn.szkug.comnix.resources` generation and iOS sync plugin.
- `libs/svga`: cross-platform SVGA decoding, rendering, playback, and dynamic content.
- `apps/*`: demos and integration hosts; do not place reusable library behavior here.
- `plugins/modules`: local Android library convention plugin.
- `benchmark`: Android macrobenchmark verification.

## Global Guardrails

1. Preserve the dependency boundary: `libs/image:image` stays engine-agnostic; Coil and Glide logic
   stays in `libs/resource:loader-engine-*`.
2. Put portable contracts and logic in `commonMain`; isolate platform behavior in
   `androidMain`/`iosMain` with explicit `expect`/`actual` APIs when required.
3. Prefer typed models, value classes, and sealed results over primitive leakage.
4. Keep public Android/iOS behavior equivalent or document intentional platform gaps.
5. Keep user-facing integration docs aligned with public API or behavior changes.
6. Do not commit build outputs, generated caches, local IDE state, credentials, or logs.
7. Preserve user changes and unrelated work already present in the working tree.
8. Do not claim completion without relevant compile, test, or inspection evidence.

## Performance Guardrails

The complete mandatory rules are in `.agents/harness/performance-coding-guidelines.md`.

1. Do not execute expensive work on the main thread.
2. Do not switch coroutine dispatchers repeatedly on a hot path.
3. Do not post frequent, redundant, or unbounded events to the main thread.

Performance-sensitive changes must identify execution contexts, dispatcher transitions, main-thread
event frequency, cancellation, and backpressure before implementation.

## Code Design

1. Write English comments for class, function, and field definitions. Demo code is exempt.
2. Explain responsibility, implementation approach when non-trivial, and related business context.
3. For complex branching, document the branch set and the purpose of each branch at the owning
   function.
4. For bug, crash, or ANR fixes, document the triggering scenario, root cause, and why the fix works.
5. Prefer reuse and clear single-responsibility abstractions over duplicated platform or engine logic.

## Default Workflow

1. Classify the task and read only the routed instructions.
2. Search existing implementations and tests before designing a new abstraction.
3. Confirm module ownership, dependency direction, threading, and event flow.
4. Make focused changes without rewriting unrelated code.
5. Run the narrowest relevant compile/test checks for every touched module.
6. Review diffs for architecture, performance, documentation, and accidental generated files.
7. Report the result, validation evidence, and any remaining risk.

## Release Workflow

1. Keep user-visible release history in `CHANGELOG.md`.
2. Commit all source changes before starting the release flow.
3. Update only the version fields, then publish from that committed source state.
4. After publication succeeds, update `CHANGELOG.md` and commit it together with the version files.
5. Create the matching annotated `v<version>` tag on that release commit, then push the commit and
   tag together.
6. Let `.github/workflows/release.yml` synchronize that version's changelog section to GitHub
   Release notes.
7. Follow `.agents/harness/release-process.md` for the complete mandatory sequence and commands.

## Definition Of Done

A task is complete only when:

1. Requested behavior and documentation are complete.
2. Triggered skill instructions and architecture boundaries were followed.
3. Applicable performance rules were reviewed.
4. Relevant module-level compile or test checks passed, or the blocker is explicitly reported.
5. Public behavior changes updated the corresponding README, or no documentation impact was
   explicitly confirmed.

## Conflict Priority

1. Data, credential, and runtime safety.
2. Build correctness and public API compatibility.
3. Architecture and module ownership.
4. Performance guardrails.
5. Documentation and code-style consistency.
6. Verification evidence over assumption.

---
> Source: [szkug/comnix](https://github.com/szkug/comnix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
