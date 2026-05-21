## agent-workflow-kit

> This is a public documentation and agent-workflow repository. Keep changes focused, readable, and tool-neutral.

# AI Project Workflow

## Project Rules

This is a public documentation and agent-workflow repository. Keep changes focused, readable, and tool-neutral.

Before editing, inspect the existing structure and follow local patterns. Do not introduce unrelated formatting churn, broad rewrites, or new tooling unless the change needs it.

## Synchronization Requirements

When changing public guide content, keep these pairs synchronized:

- `README.md` and `README.zh-CN.md`
- `docs/guide.md` and `docs/guide.zh-CN.md`
- `skills/agent-workflow-kit/` and `skills/agent-workflow-kit-zh-cn/`

When changing reusable workflow rules or references, check whether the same idea appears in:

- `docs/guide*.md`
- `skills/*/SKILL.md`
- `skills/*/references/agents-templates*.md`
- `skills/*/references/engineering-references*.md`

Do not leave one language or package with stale references that were removed elsewhere.

## Verification

Before completion, run:

```bash
ruby scripts/check-docs.rb
git diff --check
```

For faster local structural checks, or when network link checks are flaky, run:

```bash
SKIP_LINK_CHECK=1 ruby scripts/check-docs.rb
```

## External Action Safety

Ask before changing GitHub repository metadata, topics, releases, branch settings, Pages settings, or other remote configuration.

Ask before pushing commits unless the user explicitly requests a push.

## Completion Criteria

A task is complete only when:

- the requested documentation or workflow change is implemented
- related English and Chinese files are synchronized
- related guide and skill references are synchronized
- verification commands pass, or failures are clearly explained
- the final response reports what changed and what was verified

---
> Source: [crisxuan/agent-workflow-kit](https://github.com/crisxuan/agent-workflow-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
