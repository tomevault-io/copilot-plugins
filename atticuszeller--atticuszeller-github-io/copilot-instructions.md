## atticuszeller-github-io

> | Branch | Content | Purpose |

# CLAUDE.md — atticuszeller notes vault

## Layout

| Branch | Content | Purpose |
|:--|:--|:--|
| `main` | Obsidian vault at repo root | Writing notes |
| `page` | mkdocs config, docs.sh, deploy.yml | Build & deploy via GitHub Actions |
| `gh-pages` | Generated site (auto, do not touch) | Serves <https://docs.atticux.me/> |

Edit notes on `main` only. Never touch `page` or `gh-pages` manually.

## Auto-Deploy Pipeline

```
push to main → .github/workflows/notify.yml (main)
  → repository_dispatch "content-update"
  → .github/workflows/deploy.yml (page)
  → git fetch main → mkdocs build → mkdocs gh-deploy → gh-pages
```

Only pushes that touch `**/*.md` or `assets/**` trigger a deploy.

## Three-layer sync (cross-machine)

| Layer | Tool | Scope |
|:--|:--|:--|
| File sync | Obsidian Sync | Notes, assets, 99_system/, .obsidian/ |
| Git exclusions | .gitignore | 99_Private/, AI tool dirs, plugins/*/data.json |
| Git state | obsidian-git plugin | Auto-pull on startup, auto-commit/push every 5 min |

Obsidian Sync ignores hidden folders by default — `.git` at vault root is safe.

## 99_system/ convention

Mirrors the Scholar and Daily vaults. `.md` files under `99_system/agents/` are tool configs; vault-root `.claude/` etc. should be **relative** symlinks to `99_system/agents/...` to survive cross-machine paths.

## Note writing

Obsidian-flavored markdown: `[[wikilinks]]`, `![[embeds]]`, `> [!callout]`, `$...$`/`$$...$$` math, `%%comments%%`.

Vault root `assets/` is for note images/attachments. `99_system/assets/` is for system-level resources only.

---
> Source: [AtticusZeller/atticuszeller.github.io](https://github.com/AtticusZeller/atticuszeller.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
