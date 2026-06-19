## visual-eng-with-agy-skill

> >-


# Visual Engineering With agy

Route complex styling-focused visual engineering work through a low-cost general agent that runs `agy` with Gemini 3.1 Pro, while keeping the lead agent responsible for scope, review, and verification.

## agy and Antigravity Self-Ignore Rule

This skill is only for OpenCode/Sisyphus/Oh My OpenAgent routing decisions before launching `agy`. If this file is loaded inside `agy`, Antigravity CLI, or an `agy`-spawned agent, ignore the entire skill and continue with the actual user prompt. Do not apply this skill recursively, do not launch another `agy` process because of this skill, and do not treat `MUST LOAD` or routing instructions in this file as applicable inside `agy` itself.

## Mandatory Loading

This skill is mandatory for complex styling-focused visual engineering work that matches the conditions below. When the task qualifies, the orchestrator should load `visual-eng-with-agy` before delegating implementation.

Critical routing rule: do not route qualifying work directly to OMO's `visual-engineering` category first. That bypasses the `agy` workflow by selecting the OMO visual engineering model before this skill can run. For qualifying work, delegate to a low-cost general-purpose agent and load this skill explicitly with `load_skills=["visual-eng-with-agy"]`.

The mandatory marker is intentionally narrow:

- Mandatory for complex visual/styling work that needs design judgment, multi-surface consistency, screenshot-driven QA, or broad visual decomposition.
- Not mandatory for frontend logic, state, data flow, API wiring, event handling, tests, or behavior changes.
- Not mandatory for simple styling tweaks that a regular general agent can complete directly.

If the harness supports explicit skill loading, pass `load_skills=["visual-eng-with-agy"]` for qualifying tasks and use a low/general implementation category rather than `visual-engineering`.

## Correct Task Routing Examples

Wrong: this loads the skill text but still routes execution through OMO's visual-engineering model, so `agy` may never run.

```text
task(category="visual-engineering", load_skills=["visual-eng-with-agy"], run_in_background=false, prompt="...")
```

Correct for complex styling work: use a low/general category and load this skill so the wrapper agent follows the `agy` workflow.

```text
task(category="unspecified-low", load_skills=["visual-eng-with-agy"], run_in_background=false, prompt="Launch agy with Gemini 3.1 Pro per the visual-eng-with-agy skill. First decompose the visual task, then execute one atomic visual change per agy run.")
```

Use `unspecified-high` instead of `unspecified-low` only when the wrapper work itself needs more reasoning or broader project context. Do not use `visual-engineering` as the first category for this skill; reserve OMO visual-engineering for the fallback chain when `agy` is unavailable.

Correct for complex visual review or static audit tasks: still require the wrapper agent to launch `agy` first. Review-only does not mean agy-optional.

```text
task(category="unspecified-low", load_skills=["visual-eng-with-agy"], run_in_background=false, prompt="Review this visual change without editing files. You MUST launch agy with Gemini 3.1 Pro first and use its output as the review basis. If agy is unavailable, report why before using any fallback.")
```

## Lead Route Accountability

The lead agent must always state the intended route before delegating a qualifying frontend design task:

- If using `agy`, say that the subagent must launch `agy` first and include the exact `agy` requirement in the subagent prompt.
- If not using `agy`, explain why this route does not use `agy` before delegating: simple styling tweak, frontend logic, `agy` unavailable, unauthenticated, timeout, or explicit user override.
- The subagent prompt must require the subagent to report whether it launched `agy` and, if not, the exact reason.
- A subagent response that omits whether `agy` was launched is incomplete and must be followed up before accepting the result.
- Review-only and no-edit tasks still need this route statement.

## When To Use

Use this skill when all of these are true:

- The task is primarily about styling, CSS, visual layout, spacing, typography, color, animation, screenshots, responsive presentation, or browser-visible polish.
- The visual change is complex enough to benefit from a dedicated visual pass: multi-component styling, design-system alignment, responsive layout work, animation choreography, screenshot-driven polish, or a broad visual redesign.
- The user wants Oh My OpenAgent/OhMyOpenCode orchestration rather than direct implementation by the lead agent.
- A low-cost general-purpose agent is acceptable for execution, with Gemini 3.1 Pro providing the visual engineering reasoning inside `agy`.

Do not use this skill for backend-only work, non-visual refactors, pure documentation, frontend logic, state management, API wiring, data loading, validation, event handling, routing behavior, simple styling tweaks, one-off CSS fixes, minor spacing/color changes, or visual tasks that require direct specialist execution instead of `agy`.

## Complexity Threshold

Prefer a regular general agent for simple styling work. Use Visual Engineering With agy only when the visual task is meaningfully complex.

Use a regular general agent for:

- Single-property CSS changes, minor spacing tweaks, small color adjustments, typo-level class changes, simple responsive fixes, and localized component polish.
- Styling changes where the desired outcome is already explicit and does not require design judgment.
- Any change that can be safely completed by editing one small area without broader visual reasoning.

Use Visual Engineering With agy for:

- Multi-component visual changes, page-level layout work, design-system or token alignment, complex responsive behavior, animation choreography, visual QA from screenshots, or broad aesthetic direction.
- Styling tasks where the agent must infer a cohesive visual direction, compare alternatives, or preserve a design language across multiple surfaces.
- Visual changes that need browser/manual QA beyond checking one localized element.

## Routing Boundary

Frontend work must be split by responsibility before delegation:

- Use a regular general agent for frontend logic: component behavior, state, hooks, stores, data flow, API calls, validation, event handling, routing, tests, and non-visual refactors.
- Use Visual Engineering With agy only for styling and visual presentation: CSS, design tokens, layout, spacing, typography, color, animation, responsive polish, accessibility presentation states, and screenshot-driven visual QA.
- Use a regular general agent for simple styling tweaks even when they are visual; reserve Visual Engineering With agy for complex or high-impact visual work.
- For mixed tasks, delegate the logic portion to a regular general agent first, then delegate only the styling portion to this skill after behavior is stable.
- Do not send a full frontend feature to this skill unless the requested change is exclusively visual.

## Operating Model

The lead agent owns orchestration. The low agent owns the `agy` session. Gemini 3.1 Pro owns design and implementation reasoning inside that `agy` session.

1. Lead agent gathers context, defines the visual objective, and identifies files or routes likely to change.
2. Lead agent delegates to a low-cost general agent with explicit instructions to launch `agy` using Gemini 3.1 Pro.
3. Low agent first runs `agy` in planning mode to split the complex visual task into small, ordered, atomic visual changes.
4. Low agent runs a separate `agy` execution for each atomic visual change, applying and verifying one small change at a time.
5. If `agy` is unavailable, blocked by authentication, or cannot access Gemini 3.1 Pro, the lead agent falls back to OMO's configured visual-engineering agent for styling work.
6. If the OMO visual-engineering path is unavailable too, use the best available general agent that can complete the styling task, while reporting the fallback reason.
7. Lead agent reads every touched file, runs diagnostics and tests, and performs browser/manual QA when UI behavior is visible.

## Review Mode

For complex visual review, QA, critique, screenshot audit, or static inspection tasks, this skill still requires an `agy` pass even if no file edits are requested.

- The wrapper agent must launch `agy --model gemini-3.1-pro --print` before producing the review.
- The prompt must explicitly say whether the task is review-only or edit-producing.
- For review-only tasks, ask `agy` for findings, visual risks, missing polish, responsive issues, accessibility presentation issues, and suggested atomic follow-up changes.
- Do not let the wrapper agent complete the task with only its own static review when `agy` is available and authenticated.
- If `agy` is unavailable, blocked by authentication, or times out, the wrapper agent must state the reason and then use the fallback chain.

## Delegation Template

Use the lowest-cost general-purpose implementation category available in the current harness. Do not use the OMO `visual-engineering` category as the first execution path for qualifying work, because this skill's purpose is to make `agy` the first visual pass. Keep the delegation atomic and include `load_skills` and `run_in_background` explicitly.

```text
TASK: Use agy with Gemini 3.1 Pro to implement the styling-focused visual engineering change.

EXPECTED OUTCOME: Working styling or presentation changes that match the requested visual objective, preserve existing app behavior and patterns, and include a concise report of the decomposition plan, each agy execution, touched files, and verification results.

REQUIRED TOOLS: shell for launching agy, file read/edit tools for the target project, browser or screenshot tooling only when available and necessary.

MUST DO:
- Inspect existing UI patterns before editing.
- Launch agy with Gemini 3.1 Pro before any final review or implementation output, including review-only tasks.
- State in the final response whether agy was launched. If agy was not launched, state the exact reason and fallback route used.
- Launch agy with Gemini 3.1 Pro first to decompose the complex visual task into atomic visual changes.
- Launch agy separately for each atomic visual change instead of asking one agy run to implement the full visual task.
- Give agy the exact user request, target files/routes, design constraints, and verification requirements.
- Use a low-cost generic agent as the wrapper; do not route directly to a premium visual specialist unless the user asks for that path.
- Confirm the task is styling-focused and complex enough before invoking agy; route frontend logic and simple styling tweaks to a regular general agent instead.
- Keep changes scoped to the requested visual task.
- Verify after each atomic visual change before starting the next one.
- Report every file changed, every command run, and which atomic visual change produced each edit.

MUST NOT DO:
- Do not bypass agy when it is available and authenticated.
- Do not complete a complex visual review, audit, or static inspection without launching agy first.
- Do not return a result without saying whether agy was launched.
- Do not ask one agy run to implement a large multi-part visual change; split first, then execute one atomic change per agy run.
- Do not use any model other than Gemini 3.1 Pro for the agy visual pass.
- Do not implement frontend logic, state management, API wiring, or behavior changes through this skill.
- Do not use this skill for simple styling tweaks that a regular general agent can handle directly.
- Do not commit, push, publish, or modify shared infrastructure.
- Do not add unrelated redesigns, dependency churn, or speculative fallbacks.

CONTEXT: <repo paths, target route/component, design system notes, user request, constraints>
```

## agy Launch Contract

Prefer a non-interactive invocation if the local `agy` install supports it. If the exact CLI flags differ, inspect `agy --help` first and adapt without changing the Gemini 3.1 Pro model requirement.

For complex visual work, always use two phases:

1. Decomposition phase: ask `agy` to return a numbered list of small, ordered, atomic visual changes, with files or surfaces likely affected and a verification note for each change. Do not ask it to edit files in this phase.
2. Execution phase: run `agy` once per atomic visual change. Each run should receive only that one change plus the shared design context and verification requirement.

This avoids one large `agy` run producing a broad diff or timing out. Keep each execution brief small enough to complete independently.

```sh
agy --model gemini-3.1-pro --print "<visual engineering brief>" --print-timeout 5m
```

The model requirement is strict for the `agy` path: `Gemini 3.1 Pro` is the only acceptable model for the `agy` visual pass. If the local `agy` installation uses a different model identifier, the low agent must map it to Gemini 3.1 Pro explicitly and report the exact identifier used. If `agy models` or `agy --print` asks for Google authentication, report that login is required and proceed to the fallback chain.

Verified CLI shape for `agy 1.0.7`:

- `agy --version` prints the installed version.
- `agy --help` exposes `--model`, `--print`, `--prompt` as an alias for `--print`, `--print-timeout`, `--continue`, `--conversation`, and `models`.
- `agy models` lists available models only after sign-in; verified signed-in output includes `Gemini 3.1 Pro (Low)` and `Gemini 3.1 Pro (High)`.
- `agy --model gemini-3.1-pro --print "Reply with exactly: agy-ok" --print-timeout 30s` successfully returns `agy-ok` when authenticated.

The brief passed to `agy` must include:

- The original user request in full.
- The target UI surface, routes, components, and files.
- Existing design constraints and patterns discovered during context gathering.
- The expected aesthetic direction or the instruction to infer one from product context.
- Whether the prompt is for decomposition or for one atomic execution step.
- Whether the task is review-only or edit-producing.
- For execution prompts, the single atomic visual change to implement and the previous completed changes, if any.
- Verification requirements: diagnostics, focused tests, build when applicable, and browser/manual QA for visible behavior.
- Scope limits: no unrelated refactors, no commits, no dependency changes unless explicitly required.

The prompt passed to the wrapper subagent must include:

- `You MUST report whether agy was launched.`
- `If agy was not launched, explain the exact reason and the fallback route used.`
- `Do not return a final answer that omits agy usage status.`

## Visual Engineering Standards

Gemini 3.1 Pro through `agy` should produce work that feels intentionally designed, not generic.

- Preserve existing design systems when present.
- Use a clear aesthetic direction when the app has no strong visual language.
- Avoid generic SaaS defaults, purple-on-white gradients, and arbitrary animations.
- Prefer CSS variables or existing tokens for color, spacing, and typography.
- Verify responsive behavior when layout changes are involved.
- Respect accessibility basics: contrast, focus states, semantic controls, and reduced-motion expectations.

## Lead Verification Gate

Never trust the delegated report alone. Before shipping the result, the lead agent must:

1. Read every file the low agent changed.
2. Run diagnostics on changed files when language tooling exists.
3. Run the most focused relevant tests or build command available.
4. Use browser/screenshot/manual QA for visible UI changes when the app can run.
5. Summarize any blockers, unavailable tools, or pre-existing unrelated failures.

## Failure Handling

If `agy` is missing, Google authentication is unavailable, Gemini 3.1 Pro is unavailable, or the low agent cannot run the workflow, use this fallback chain:

1. Try OMO's configured `visual-engineering` agent for styling-focused work.
2. If that path is unavailable, try the best available general agent that can complete the styling task.
3. Report which path was used, why `agy` was not used, and whether Gemini 3.1 Pro was unavailable or merely unauthenticated.

Do not silently fall back. The fallback is allowed only when the report names the blocker and the replacement execution path.

If `agy` produces broad or unrelated changes, reject the result, restore only the low agent's unrelated edits when safe, and re-run with a narrower brief.

If an `agy` execution times out, split that atomic change into smaller visual changes and retry them one at a time before using the fallback chain.

---
> Source: [186526/visual-eng-with-agy.skill](https://github.com/186526/visual-eng-with-agy.skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
