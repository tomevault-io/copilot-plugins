## sovant-ai

> Clear, maintainable code is critical to Sovant’s long-term scalability. Follow these principles when making any changes, especially when using AI-generated code.

# 🛠️ Sovant AI Engineering Instructions (Claude-Coded Features)

Clear, maintainable code is critical to Sovant’s long-term scalability. Follow these principles when making any changes, especially when using AI-generated code.

---

## ✅ CLARITY & COMMUNICATION

- Explain bugs, changes, or decisions in plain English — use everyday analogies when helpful.
- Ask clarifying questions until you’re at least 95% confident in the task scope.
- Never simplify, shorten, or reinterpret system instructions unless explicitly told to.
- When summarizing code or logic, be exhaustive — do not omit important details.

---

## ✨ SIMPLE, CLEAN CODE

- Use the **simplest working solution** that meets the requirements — no cleverness.
- Avoid introducing new libraries, tools, or patterns unless truly necessary.
- Clean up obsolete logic when introducing new code (avoid tech debt).
- Always trace issues to the root cause — don’t patch symptoms.

---

## 🧱 STRUCTURE & ORGANIZATION

- Refactor any file that exceeds **400 lines** — keep modules focused.
- Follow the existing folder structure, naming conventions, and coding styles.
- Keep the root folder clean. Store experimental scripts, test data, and debug tools inside `dev_resources/`.

---

## 🌐 ENVIRONMENT-SAFE PRACTICES

- Always write environment-aware code (dev/test/prod separation).
- Never hard-code secrets or configuration values — use environment variables.
- Manage env vars via Supabase/Vercel environment controls.
- Only include mock data inside test files — **never in dev or prod environments.**
- Do not insert detection logic, placeholders, or dummy values into main production code.

---

## 🔁 CODE REUSE & NON-DUPLICATION

- Always check the codebase for existing logic before writing new utilities or hooks.
- Reuse centralized helpers and shared modules to avoid repeated inline code.
- Favor composable React hooks and utility functions in Next.js.

---

## 🔒 SAFE CHANGE MANAGEMENT

- Only modify code directly related to the requested task or issue.
- Document why a fix works in the **commit message or PR description** to aid future debugging.
- Avoid speculative fixes — use targeted logging or debug tools to investigate before coding.
- Remove all temporary code, debug flags, or `TODO` stubs once the root issue is solved.

---

## 🧪 TESTING PRACTICES

- Write basic tests to confirm new logic or edge cases — especially for critical features.
- Prefer testing pure logic over UI snapshot tests unless UI behavior is the feature.
- Use mocks/stubs when working with external systems (e.g., Supabase, Stripe, webhooks).
- Store test files alongside the modules they cover, or in clearly mapped `/__tests__/` folders.

---

## 🔍 DEBUGGING & DIAGNOSTICS

- Use `console.log()` only for **targeted debugging**, and remove before merging unless intentional.
- Log context-rich messages: include input data, state summaries, or error origins.
- Create temporary debug-only wrappers in `dev_resources/` if needed — document clearly.

---

## 💬 COMMENTING & TRACEABILITY

- Include high-level comments at the **top of each file** and **each function** to explain intent.
- Prioritize explaining **why** something is done, not just what the code does.
- Link to related issues, PRs, or Slack threads if context is external.

---

## 🚀 DEPLOYMENT READINESS

- Use feature flags or guards for any potentially breaking or experimental logic.
- Double-check for unintended changes in API contracts, DB schemas, or user-facing logic.
- PRs must be atomic — no bundling of unrelated fixes, experiments, or features.

---

## 📚 DOCUMENTATION & DEVELOPER NOTES

- Add edge case notes or gotchas in a markdown file inside `dev_resources/notes/`.
- Update READMEs or add inline documentation for new modules or patterns.
- When unsure where to document something, leave a `NOTE:` comment with context.

---

💡 **Reminder:** If you’re not 95% sure what’s being asked, pause and clarify before coding. Our future selves will thank you.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hechin91) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
