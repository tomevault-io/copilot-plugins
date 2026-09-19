## jev-recruiter

> This adaptation adds a LinkedIn recruiting workspace. Its explicit user requested

# Jev Ultrafast

This adaptation adds a LinkedIn recruiting workspace. Its explicit user requested
scope permits a site specific, navigation and scrolling only discovery loop in
recruiter.py. Preserve the generic agent source as a reference. The recruiting server exposes
only Jev model paths, never the generic text helper. Browser decisions must call
the upstream operation and target chooser. Assessments use Jev choices over
explicit criteria and indexed observed excerpts, not generated text. Assessments must
cite observed professional evidence, retain unknowns, and leave candidate decisions
to the user. Recruiting results belong in ignored artifacts/recruiting/.

Read README.md before editing. Keep the loop small: page -> indexed elements -> operation + target -> execution.

- The input is one natural-language goal. Do not add site-specific plans or hardcoded field values.
- TypeSafe chooses an operation and operation-specific target heads in one request. Consume only the selected operation's target.
- Targets must map to observed elements and supported operations. Never let the model emit selectors or executable code.
- TYPE_TEXT invokes the text LLM. Cache a stale retry's value only while its entire helper input is identical.
- Never retry a browser mutation. Log execution before observing its result.
- Screenshots are optional; the model does not consume them. Keep demonstration footage at its original speed.
- Keep credentials server-side and .env ignored. Tests must not call paid APIs.
- Verify actual final outcomes independently. A DONE choice is not proof of success.
- Keep examples, README claims, raw evidence, and model-call counts consistent.
- Do not commit or push unless the user requests it.

Checks: uv run ruff check ., uv run pytest, node --check jev_ultrafast/static/app.js, uv build.

---
> Source: [skeptrunedev/jev-recruiter](https://github.com/skeptrunedev/jev-recruiter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
