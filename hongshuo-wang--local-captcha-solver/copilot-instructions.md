## local-captcha-solver

> - Keep the core product limited to static, single-image text CAPTCHA recognition for digits, English letters, alphanumeric strings, and one-step integer arithmetic.

# Project Instructions

## CAPTCHA scope

- Keep the core product limited to static, single-image text CAPTCHA recognition for digits, English letters, alphanumeric strings, and one-step integer arithmetic.
- Prefer common CAPTCHA formats. Do not expand the core model for niche scripts, multi-step mathematics, image selection, sliders, puzzles, animation, or behavioral challenges without an explicit product decision.
- Arithmetic supports `+`, `-`, `*`, `/`, `x`, `X`, `×`, and `÷`. Training data should use nonnegative subtraction and exact division. Common suffix forms such as `=?`, `=`, `?`, and no suffix must be represented.

## Model and data changes

- Treat the repository benchmark, not an individual screenshot or same-generator random split, as the source of truth for model quality.
- Never add benchmark fixtures to training or validation data. Split synthetic templates, public dataset generators, and real website families by `group`; one group must not cross splits.
- Every training image must have a stable label, SHA-256, source, license id, and scenario/template group in the dataset manifest.
- Do not train on or redistribute a public dataset until its license and provenance have been recorded. A repository license alone does not automatically license bundled images.
- Keep downloads, extracted third-party data, checkpoints, and training output out of git. Commit only manifests, generation code, pinned metadata, documentation, and intentionally redistributable fixtures.
- A production model replacement requires reproducible training inputs, a completed model card, Paddle-to-ONNX consistency checks, the frozen benchmark, and Chrome/Edge offline verification. Do not replace `public/models` from a single issue sample.

## Open-source scenario contributions

- Secondary TODO: maintain a concise contributor retraining guide as the model and dataset recipe stabilizes; this documentation must not block current model-quality and browser-verification work.
- Convert requests for a new CAPTCHA style into a reproducible scenario contribution: authorized samples, exact labels, license/provenance, a unique group, a failing held-out benchmark, and a description of the visual mechanism that current coverage misses.
- Prefer extending deterministic generators or augmentation families over adding site-specific image hacks. Keep site-specific behavior behind an explicit per-site mode only when a general model/decoder change is not justified.
- Update `docs/model-training.md`, the dataset source catalog, tests, and the model card whenever the training recipe, supported scope, charset, data policy, or release gate changes.
- Preserve the single-model-first architecture. Add a second production model only after the primary model fails the documented selective-accuracy and coverage gates on isolated groups.

## Release gates

- Optimize for selective precision: automatic-fill precision must be at least 99.5%, coverage at least 80%, cold start at most 3 seconds, and warm single-image P95 at most 500 ms on the documented reference machine.
- A missing or weak arithmetic operator is not evidence for a digits fallback. Abstain instead of automatically filling a structurally ambiguous value.
- Report results by category, source, scenario group, and arithmetic symbol. Aggregate accuracy alone is insufficient.

## Release experience

- Every version update opens its standalone “本次更新” page once. Keep a “新功能” entry in the popup header so users can reopen it at any time.
- First-install setup and every release introduction start with a concise welcome scene, followed by explanations or settings reached through native scrolling. Use shared glass surfaces and elastic scene transitions, with reduced-motion support. Release introductions describe shipped benefits and end with a “开始使用” action.
- Every published version, including patch releases, must have its own dated introduction and bilingual user-facing notes in the local release archive. Keep historical entries accurate to what that version shipped. Read the full notes from `CHANGELOG.md`; maintain the shared brand rules in `DESIGN.md`. Historical browsing must preserve the installed-version reminder and user settings.
- Apply the same editorial standard to release headlines, summaries, introductions, full local notes, and GitHub Release notes: explain a shipped feature, a useful interaction improvement, an affected browser or website, a concrete fix, or a permission/privacy change the user needs to understand. Lead with the user's task and the resulting behavior; include technical terms only when needed to act or troubleshoot.
- Keep developer-support promotions, donation-page additions, code restructuring, dependency/build/test changes, training implementation details, and maintainer workflows in commits or developer documentation. When such work has a verified user-facing effect, describe that effect in the release notes. Scale the introduction to the actual change; a small fix can have one highlight. A maintenance-only version still gets a brief, factual introduction about the continuity of use, without invented feature, reliability, or performance claims.
- When a documented change has a confirmed issue association, append a clickable link to that specific item in both languages using `[#N](https://github.com/hongshuo-wang/local-captcha-solver/issues/N)`. Verify associations against the issue or commit record, preserve links in the local pages and generated release notes, and keep historical links attached to their original version.
- The popup is one current-page workspace with one primary action selected from the page context. Manual binding is a secondary guided action; saved bindings expose their status and management actions.
- First-install setup covers welcome, static CAPTCHA site access, and filling preferences in three scrollable sections. Finish setup saves the choices and requests the necessary permissions from that user action. Local practice and Chrome/Edge slider setup are optional actions after completion. The optional slider branch stays in the same tab, provides Back, and returns to a slider completion state after saving its own settings.

## User communication

- When answering project usage, maintenance, or workflow questions for ordinary users, write like a developer speaking directly with a user: lead with the practical answer, use conversational language, and prefer familiar words over technical terms.
- Introduce a technical term only when it helps the user act or make a decision, and explain it briefly in plain language.
- Answer the user's current question directly. Do not repeat information already established unless it is needed to prevent ambiguity or a mistake.

## Browser testing

- Run all browser tests in headless mode with isolated test profiles so the user can continue using the computer without interruption.
- Keep browser windows, focus changes, and test input off the user's desktop. If a test requires a visible browser, adapt it for headless execution or report the limitation.

## No Negative Echo



生成最终产物及其包装时，包括标题、文件名、正文、注释、标签、commit、

PR 和交付说明，只描述最终采用的状态，假设读者没看过本次会话。



- 会话里的否决、中间尝试和措辞纠正，只当作控制信息，不要让它们成为最终产物的命名或叙述中心。

- 对每个交付面分别判断：不知道本次会话的读者需要这条信息吗？省略会不会导致不准确、不安全、误导或兼容性信息缺失？它是不是任务开始时已提交或用户确认状态中的真实变化，而且当前交付面需要解释它？

- 「不要提 X」不是让你写「无 X」。标题、文件名、开篇和标签应从正向目标重新生成，不要逐词修改被否文案。

- 保留真实的基线变化、已经执行的外部操作，以及必要的技术名称、诊断、测试和快照。任务开始前已有的用户改动不算被否内容。

- 不要把与本任务无关的改动写进本次 commit、PR 或交付说明。对比、引用、审计和迁移说明，只在用户要求或当前交付面确实需要时保留。

- 写完后通读全部用户可见内容及其包装，包括文件名、元数据和 hook 改写。内容发生变化后重新检查，不要另加「已清理」或「无残留」类声明。

---
> Source: [hongshuo-wang/local-captcha-solver](https://github.com/hongshuo-wang/local-captcha-solver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
