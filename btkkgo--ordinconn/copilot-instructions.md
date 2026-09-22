## ordinconn

> 1. Read `docs/PRODUCT_BASELINE.md` before making product changes. It is the only product baseline. Read-only tasks and instruction/documentation maintenance do not require product-baseline reading unless a product decision depends on it. Reuse already-read, unchanged context.

# OrdinConn Agent Rules

1. Read `docs/PRODUCT_BASELINE.md` before making product changes. It is the only product baseline. Read-only tasks and instruction/documentation maintenance do not require product-baseline reading unless a product decision depends on it. Reuse already-read, unchanged context.
2. Never restore features or product definitions from the previous OrdinConn repository.
3. Make small, testable changes and inspect the active implementation before editing it.
4. Keep Evidence distinct from model inference and keep every Published Signal traceable to Evidence.
5. User-funds operations always require explicit approval. V0.1 contains no real-money execution.
6. Never collect private keys, seed phrases, or password-field contents.
7. Route every model through Model Gateway and every data source through Connector Registry.
8. A Trade Proposal must pass through Approval before any execution adapter.
9. Keep core crates transport-agnostic and keep React behind typed IPC contracts.
10. Use the approved user-provided purple, black, and yellow brand system and locale keys for all formal UI copy.
11. For product code changes, run Rust tests, TypeScript tests, typecheck, and Rust/frontend builds before declaring completion. Run desktop packaging when packaging, native integration, or release delivery is affected. For read-only or instruction/documentation-only tasks, validate the relevant instructions, links, and diff instead. Reuse successful checks for identical code, dependencies, commands, and relevant environment; rerun affected checks after changes and report failures or unverified criteria explicitly.
12. Push only when the user explicitly asks or when the issue-first public development workflow below already authorizes the same scoped task. Never force-push.

## Task scope and Skills

- Use a Skill when explicitly requested or when its capability directly matches the task and actual technology. A keyword, file extension, dev-server start, or progress question alone is insufficient. Reading a Skill for audit does not activate its workflow.
- Load only task-relevant references. Reuse unchanged instructions already in context; reread when content changes or needed context is missing. Freetower, Figma, Vercel, and analytics artifact workflows apply only when that project, tool, or deliverable is actually in scope. OrdinConn desktop validation follows the real Tauri application and typed IPC path.
- For clear, authorized, reversible work, inspect the implementation, make the smallest useful change, and continue without design, per-section, execution-method, or continuation approval prompts. Preserve an explicit plan-only, review-only, or approval-first boundary. Ask about material missing information or scope changes that cannot be resolved from the request and existing context.
- Existing authorization applies to the same scope and action; it does not authorize funds operations, destructive data changes, sensitive access, new external communications, or publication. Obtain any missing explicit approval for those actions before execution. Never infer consent from silence. The funds, privacy, Approval, and no-push rules above remain binding.
- Choose one workflow to coordinate planning, debugging, and verification. Share its evidence with supporting Skills; do not repeat intake, full test suites, or review of an unchanged diff. Diagnose routine dependency/test failures within scope; escalate when progress requires missing authority or a material user decision.
- Use existing components, dependencies, Model Gateway, and Connector Registry. Create worktrees only when isolation is needed or requested. Verify file ownership and destination before writes; preserve unrelated changes. Generate persistent plans only when useful to the requested deliverable. Commit only when requested or established project policy authorizes it; do not present merge/push menus for ordinary task completion.

## PUBLIC DEVELOPMENT LOG POLICY

- After an engineering-significant task, append a sanitized `Public Interaction Summary` to `docs/devlog/YYYY-MM-DD.md` with: Timestamp, Stage, User Goal, inspected areas, changes, files changed, problems, solution, commands/tests, result, remaining risks, next step, and a Codex engineering note.
- Never copy a full prompt, private conversation, credential, token, cookie, session, password, account identity, customer data, private contact detail, exact personal address, restricted asset, or raw machine home path into the public log.
- Use current code, current documentation, Git history, and fresh verification as truth sources. Keep `Implemented`, `Designed`, `Verified`, and `Blocked` distinct; fixture coverage never substitutes for real-environment evidence.
- Scheduled public-log sync may stage only `docs/`, `social/x/drafts/`, `.github/ISSUE_TEMPLATE/`, and `.github/PULL_REQUEST_TEMPLATE.md`. It must never stage Actions workflows. Product code and sync/security scripts must use the verified task-completion commit path, never the two-hour documentation job.
- Run the public-log sanitizer before any public commit, GitHub push, or X publication. Any suspected secret fails closed; do not print the suspected value or continue publication.
- Generate X material only for a meaningful stage, reusable engineering failure, or weekly summary. X publication is always manual and user-owned. Never authenticate to X, read an X browser session, call an X API, or publish automatically. Move a draft to `published/` only after the owner records its timestamp, URL, source commit, and content hash.

## BUILD IN PUBLIC LANGUAGE POLICY

- Codex communicates with the user in Chinese by default for requirements, execution updates, analysis, test results, acceptance reports, and next-step recommendations. Use English only when the user explicitly requests it. Engineering terms and report headings may remain in English when that improves precision.
- Code, schemas, identifiers, and commit messages use English. Do not use bilingual or mixed-language commit subjects.
- GitHub public documentation is bilingual: English is the primary/default version and Simplified Chinese is the secondary version. Important documents use `<NAME>.md` and `<NAME>.zh-CN.md`, with reciprocal `English | 简体中文` links at the top.
- Keep `README`, `CURRENT_STATUS`, architecture, roadmap, changelog, security, contributing, Problems and Solutions, Codex Field Notes, Build in Public policy, and other core public records synchronized in both languages. When either language changes, update or create its counterpart in the same task.
- English and Chinese status documents must report identical stage, status, tests, `PASS` / `FAIL` / `BLOCKED` outcomes, Issues, commits, risks, and next steps. If they disagree, resolve the truth from code, real tests, GitHub Issues, Current Status, DevLog, then X, and update both documents.
- Core architecture documentation is bilingual. Short ADRs may contain `## English` and `## 中文` sections in one file; long ADRs use paired `.md` and `.zh-CN.md` files. English remains the primary version.
- DevLogs use `docs/devlog/YYYY-MM-DD.md` for English and `docs/devlog/YYYY-MM-DD.zh-CN.md` for Chinese. Both files describe the same sanitized facts and must be updated together for meaningful engineering work.
- GitHub Issue titles use English. Issue bodies and important milestone comments put `# English` first and `# 中文` second, with Goal, Scope, Current State, Acceptance Criteria, Result, and Validation represented consistently in both languages. Very short mechanical status comments may be English-only.
- Pull Request titles use English. Pull Request bodies put English first and Chinese second.
- X drafts remain manual and owner-published. Each draft contains `## English — Publication Version` for the intended English post and `## 中文 — 参考版本` for the complete corresponding Chinese thread. Derive both naturally from the same sanitized engineering facts; do not mechanically translate private conversation.
- Final Codex execution reports to the user are in Chinese by default. Never let GitHub's English-first policy change the actual collaboration language.

## ORDINCONN PUBLIC DEVELOPMENT RULES

- Official GitHub repository: `https://github.com/Btkkgo/OrdinConn`. Never create another OrdinConn repository, an `OrdinConn-v2`, or a substitute repository. Continue all public development in this repository.
- GitHub is the public engineering source of truth. If code, tests, Current Status, Issues, DevLogs, and X differ, trust them in that order; X can never claim a more advanced state.
- Meaningful engineering work is issue-first:
  1. Read the relevant Issue.
  2. Create a scoped Issue if none exists.
  3. Implement within the Issue boundary.
  4. Run proportional tests and real-environment gates.
  5. Update the daily DevLog.
  6. Update the Issue with inspected areas, changes, files, commands, tests, problems, result, remaining work, and next step.
  7. Run `scripts/security/check-public-repo.sh` and `git diff --check`.
  8. Commit with `Refs #N`; use `Fixes #N` only when every acceptance criterion is complete.
  9. Push the active issue branch or `main` without force.
  10. Create an X draft only when stage-worthy.
  11. Never publish X automatically.
- Use `status:in-progress` when work starts, `status:blocked` for a named unmet prerequisite, `status:needs-validation` when code exists without adequate real acceptance, and `status:verified` only after the Issue acceptance criteria pass. Close an Issue only when its complete acceptance criteria pass.
- Use `main` as the default branch. Significant runtime work should normally use `issue/<number>-<short-name>`; small documentation changes may use `main`. Avoid complex Git Flow.
- At the end of a meaningful task, update the Issue and DevLog, run the security/diff/test gates, commit, and push promptly. A failed or blocked test may still be recorded with an honest `wip:`, `docs:`, `test:`, or `investigation:` commit; never describe it as a completed feature.

---
> Source: [Btkkgo/OrdinConn](https://github.com/Btkkgo/OrdinConn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
