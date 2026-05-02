## image2-design-director

> - `README.md`: public-facing project overview and first-run navigation.

# AGENTS.md

## Project Map

- `README.md`: public-facing project overview and first-run navigation.
- `SKILL.md`: skill entry point, trigger contract, default workflow, and core quality gates.
- `references/`: reusable protocols, decision gates, scorecards, prompt rules, runtime-memory rules, and delivery policies.
- `patterns/`: promoted reusable design patterns; only add patterns after a traceable capture exists.
- `scripts/`: Python helper scripts for runtime memory, bundle management, overlay, export, publication review, and governance.
- `docs/README.md`: documentation router for public docs, historical validation, and maintainer-only context.
- `docs/benchmarks/` and `docs/experiments/`: historical validation evidence and regression scenarios.
- `assets/readme-story/`: README homepage visual assets.
- `agents/openai.yaml`: minimal host/interface example.

## Read First

- First-time project context: read `README.md`, then `docs/README.md`.
- Skill behavior changes: read `SKILL.md` and the relevant files under `references/`.
- Runtime-memory changes: read `references/runtime-memory.md` and the related `scripts/*runtime*` files.
- Delivery/overlay/export changes: read `references/delivery-ops.md`, `references/delivery-viability-gate.md`, `references/post-processing-policy.md`, and `references/text-overlay-policy.md`.
- Publication-asset changes: read `references/scorecard.md`, `references/quality-bar.md`, and any relevant `docs/articles/` or `docs/benchmarks/` evidence.
- Pattern changes: read `patterns/README.md` before adding or promoting a pattern.

## Commands

- Syntax check scripts: `python3 -m compileall scripts`
- Runtime-memory smoke test: `python3 scripts/init_runtime_memory.py --host generic --root tmp/runtime-smoke`
- Runtime read smoke test: `python3 scripts/read_runtime_context.py --host generic --root tmp/runtime-smoke --raw-only`
- Script help check: `python3 scripts/<script>.py --help`

There is currently no repo-wide package manager file, lint command, typecheck command, or test suite. If a change introduces one, update this section in the same change.

Image-processing scripts use Pillow. If those scripts fail with `ModuleNotFoundError: No module named 'PIL'`, install or use an environment that provides Pillow before validating image operations.

## Repository Rules

- Do not treat `runtime/`, `state/`, `tmp/`, `artifacts/`, `preview/`, generated images, local captures, or test outputs as public repo source unless the task explicitly asks to promote them.
- Do not change generated or historical evidence files just to make the tree look tidy.
- Do not promote a benchmark candidate, overlay demo, or delivery-bundle artifact to a publication asset unless the relevant review criteria pass.
- Do not add a new pattern without evidence that can trace `brief -> image_prompt -> image output -> evaluation`.
- Keep root docs as navigation and stable rules; move long workflows, runbooks, and historical notes into `docs/` or `references/`.
- Prefer updating existing protocol files over creating overlapping rules in a new location.
- If a behavior change affects `SKILL.md`, keep the corresponding `references/` document and validation evidence aligned.

## Working Style

- You may use technical methods to finish the task, but report back in simple, direct language.
- Before starting, internally decide what would count as finished for the current task.
- Work autonomously unless a blocking condition below applies.
- Do not hand back a half-finished change for manual inspection when it can be completed and checked locally.
- Preserve unrelated user changes in the working tree.

## Completion Rules

- For script changes, run `python3 -m compileall scripts` and at least one representative script smoke test.
- For runtime-memory changes, run the runtime init/read smoke tests with a `tmp/` root.
- For delivery-bundle, overlay, publication-review, or export changes, run the smallest representative command that exercises the changed path, using `tmp/` output when possible.
- For docs-only changes, re-read the edited sections and check that links, paths, and document routing still match the repository structure.
- For `SKILL.md` or protocol changes, verify the change against the relevant reference docs and update benchmark or experiment notes when behavior meaningfully changes.
- For README or visual-asset changes, check the referenced asset paths and, when possible, open or render the affected preview.
- If validation fails, fix the issue and rerun the relevant check before reporting completion.

## Escalate Before Changing

Ask the user before proceeding only when:

- The request has a key ambiguity that would likely produce the wrong asset, protocol, or public behavior.
- The task requires an account, secret, missing file, external permission, or business judgment.
- The operation may delete data, affect production, rewrite history, or be hard to reverse.
- The current environment cannot perform the necessary validation.
- A change would alter the public skill contract, promotion policy, or publication-readiness criteria in a way that is not clearly requested.

## Final Report Format

Use this format for final replies:

```text
完成情况：成功 / 部分完成 / 未完成
做了什么：用 2-4 句话说明
验证结果：说明跑了什么、结果怎样
需要注意：只写重要风险或限制；没有就写“暂无”
```

Do not fill the final reply with implementation details, terminal logs, or code jargon unless the user explicitly asks for them.

---
> Source: [boyzcl/image2-design-director](https://github.com/boyzcl/image2-design-director) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
