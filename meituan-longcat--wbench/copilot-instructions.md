## wbench

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Critical: Do NOT delete or switch branches destructively

- `data/` and `weights/` are **symlinks** to the parent project, NOT tracked by git (in `.gitignore`).
- **NEVER** run `rm -rf *`, `git checkout --orphan`, or any command that deletes untracked files in the working tree.
- **NEVER** switch to gh-pages or any orphan branch in this working directory. Use a separate temp directory for gh-pages deployment instead.
- If you need to deploy to gh-pages, do it in `/tmp/` without touching the working tree:
  ```bash
  # Correct way to deploy gh-pages:
  rm -rf /tmp/gh-pages-deploy
  mkdir /tmp/gh-pages-deploy
  cp -r homepage/* /tmp/gh-pages-deploy/
  cp -r homepage/data/thumbs homepage/data/masks homepage/data/gallery.json /tmp/gh-pages-deploy/data/
  cd /tmp/gh-pages-deploy && git init && git checkout -b gh-pages && git add -A && git commit -m "Deploy"
  git remote add origin <url> && git push --force origin gh-pages
  ```

## Symlinks

| Path | Target | Purpose |
|:---|:---|:---|
| `data` | `../../data/wbench_preview` | Preview dataset (289 cases, images, masks) |
| `weights` | `../../weights` | Model weights (aesthetic, RAFT, DA3, SAM2, etc.) |

These will be deleted if you run destructive git operations. If lost, recreate with:
```bash
ln -sf /mnt/dolphinfs/hdd_pool/docker/user/hadoop-aipnlp/EVA/yingkaining/WBench/data/wbench_preview data
ln -sf /mnt/dolphinfs/hdd_pool/docker/user/hadoop-aipnlp/EVA/yingkaining/WBench/weights weights
```

## Homepage Deployment

The homepage lives in `homepage/` and is deployed to the `gh-pages` branch.
- Total size ~26MB (thumbnails + compressed videos)
- Push via proxy: `export https_proxy=http://10.229.18.30:8412`
- GitHub Pages serves from root of gh-pages branch

---
> Source: [meituan-longcat/WBench](https://github.com/meituan-longcat/WBench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
