## conventional-commits

> Require scoped conventional commits with detailed bullet-point bodies via heredocs.


# Conventional commits

Use [Conventional Commits](https://www.conventionalcommits.org/) for every commit:

```
<type>(<scope>): <short imperative summary>
```

- Prefer explicit scopes tied to changed areas in this repo, such as `workflow`, `auth`, `reviewer`, `applicant`, `docs`, `deployment`, `tests`, or `migrations`.
- If staged files span unrelated concerns, split them into separate scoped commits.
- Keep the subject line imperative, concise, and without a trailing period.

## Commit body

Every commit must include a body with detailed bullet points that explain:

- what problem is being solved or why the change is needed
- what was implemented or changed
- impact, risk, migration notes, or test coverage when relevant

Body bullets must start with `- ` and be specific. Do not use placeholder bullets.

## Writing commit messages

Always pass commit messages through a heredoc so newlines are preserved and stray `\n` literals are not introduced:

```bash
git commit -m "$(cat <<'EOF'
feat(workflow): add review start transition

- add the explicit review start transition so submitted applications move into under-review intentionally
- assign the reviewer during the transition and persist the action in the audit trail
- cover legal and illegal review-start paths in tests so reviewer ownership is enforced by the backend

EOF
)"
```

Do not use `-m "line1\nline2"` or other escaped-newline strings for multi-line commit messages.

---
> Source: [stctheproducer/open-submission-approval-workflow](https://github.com/stctheproducer/open-submission-approval-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
