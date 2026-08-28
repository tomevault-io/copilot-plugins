## dsh-ag-ui

> Community DeepSeek Harness (DSH) Host plugin exposing DSH Agents over the AG-UI

# dsh-ag-ui

Community DeepSeek Harness (DSH) Host plugin exposing DSH Agents over the AG-UI
protocol. See README.md for usage; see docs/CONTEXT.md for the local design
consensus (not committed).

## Agent skills

### Issue tracker

Issues are tracked in GitHub Issues via the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Default five-role vocabulary (`needs-triage` / `needs-info` / `ready-for-agent` / `ready-for-human` / `wontfix`). See `docs/agents/triage-labels.md`.

### Domain docs

Single-context. See `docs/agents/domain.md`.

## Git hygiene

Git information must not reveal AI collaboration:

- Commit messages must not mention AI tools, models, or agents (no "Claude",
  "Copilot", "generated with …", or equivalent in any language).
- No AI attribution trailers or footers of any kind (e.g. `Co-Authored-By:` an
  AI, "Generated with …" signatures).
- The same rule applies to PR titles, PR descriptions, and GitHub issue/PR
  comments.
- Author and committer identity stays the human owner's configured git
  identity.

---
> Source: [CaiZongyuan/dsh-ag-ui](https://github.com/CaiZongyuan/dsh-ag-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
