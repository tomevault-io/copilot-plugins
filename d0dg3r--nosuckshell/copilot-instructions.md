## nosuckshell

> Use GitHub CLI (gh) for GitHub data in this repo


# GitHub-Zugriff

In dieser Umgebung ist die **GitHub CLI (`gh`)** verfügbar und authentifiziert. Für Actions-Logs, Runs, PRs, Issues, Releases und Repo-Metadaten: **`gh` bevorzugen** statt nur die öffentliche Web-UI zu raten.

Beispiele:

- `gh run view <run-id> --log-failed` / `gh run view --job <job-id> --log`
- `gh api repos/:owner/:repo/actions/runs/...`
- `gh pr view`, `gh issue list`, etc.

Wenn GitHub-Informationen für die Aufgabe nötig sind, Befehle **selbst ausführen** und die Ausgabe auswerten.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/d0dg3r) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
