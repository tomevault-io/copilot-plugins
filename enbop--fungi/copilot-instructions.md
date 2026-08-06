## fungi

> - Check the current branch and `git status`, and preserve unrelated changes.

# Agent guidance

## Working agreements

- Check the current branch and `git status`, and preserve unrelated changes.
- Keep changes focused on the requested scope.
- Do not commit, push, or open a pull request unless explicitly asked.

## Verification

Run checks relevant to the changed code:

```bash
cargo fmt --all -- --check
cargo build
cargo test --all
```

## Commits

Use Conventional Commits in English:

```text
<type>(<optional-scope>): <imperative summary>
```

## AI attribution

For non-trivial AI-assisted commits, add:

```text
Assisted-by: <agent>
```

Include the model only when it is explicitly known:

```text
Assisted-by: <agent>:<model>
```

## Pull requests

Before drafting or creating a pull request, read
`.github/PULL_REQUEST_TEMPLATE.md` and use it as a reference.

---
> Source: [enbop/fungi](https://github.com/enbop/fungi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
