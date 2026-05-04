## gitlab-token-source

> Read GitLab token from local secrets.env


# GitLab Token Source

When any task requires GitLab API authentication, always load the token from `local/secrets.env`.

- Preferred source: `local/secrets.env` with `GITLAB_TOKEN="glpat-..."`
- If `GITLAB_TOKEN` is unset, source `local/secrets.env` before running GitLab API or release upload commands.
- Never hardcode, paste, or commit GitLab tokens in commands, code, docs, commit messages, or chat output.
- Keep `local/secrets.env` local-only (gitignored).

Example:

```sh
if [ -z "${GITLAB_TOKEN:-}" ] && [ -f "./local/secrets.env" ]; then
  . "./local/secrets.env"
fi
```

---
> Source: [OpenCSGs/csghub-lite](https://github.com/OpenCSGs/csghub-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
