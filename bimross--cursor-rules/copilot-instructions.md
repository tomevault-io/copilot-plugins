## cursor-rules

> BimRoss Git flow: default branch is dev integration; push finished work to GitHub; deploy = commit + push + v* tag + release notes, no rollout polling.


# BimRoss: default branch, GitHub, and releases

## Default branch = dev

Treat **`main`** / **`master`** (per repo) as the **integration line**—not a sacred “only via release” branch. Work lands here so you can **passively** verify via Git history, Actions, and diffs on GitHub.

## After a medium or large implementation

When implementation is **done** (builds/checks pass, self-review complete):

1. **Commit** with a clear message.
2. **Push to GitHub** on the default branch (or the branch you use for integration—match the repo’s norm).

Do **not** leave large finished changes only on disk; **push** so your human loop can see it without asking.

## What “deploy” means at BimRoss

When Grant says **deploy** (in the sense of shipping the work just finished—not casual hypotheticals), treat it as the **standard release path** for the repo(s) in scope:

1. **Commit** and **push** so GitHub has the merged integration line.
2. Cut a **`v*`** **semver tag** on the repo(s) that ship versioned images or GitOps bumps (match each repo’s CI: tag-driven Docker push, **`bimross/rancher-admin`** updates, Fleet apply).
3. Add an **informative GitHub Release** (or release notes on the tag) describing what changed—enough for you and Fleet to know what rolled.
4. Stop active deploy tracking after tag/release creation. CI/CD and GitOps continue asynchronously (image builds/push, manifest bumps in **`bimross/rancher-admin`** where configured, then Fleet rollout), and runtime confirmation comes from the downstream Slack deploy signal rather than in-session polling.

Agents do not invent releases silently; **“deploy” from Grant is explicit authorization** to run that full path for the feature context.

## Releases and tags — human gate

- **Do not** create **`v*`** tags, GitHub **Releases**, or production promotion steps **unless Grant explicitly asks** in the conversation—including **“deploy”** (see above), “tag it,” “cut a release,” “publish `v0.0.7`”, **“okay deploy”** / **“deploy everything”**.
- Pushing to the default branch **without** a deploy ask is **not** a release; it is **dev visibility + CI**.
- If unsure whether to tag: **push only** and state what’s left for a human release.

### Exception: multi-repo “okay deploy”

When Grant runs the full multi-repo deploy pass (**`bimross-okay-deploy.mdc`**): follow that checklist (e.g. **`cursor-rules`** semver tag, **`employee-factory`** when applicable), but end the agent task after ship artifacts are pushed/tagged. That phrase is explicit full-workspace release authorization—not a requirement to poll rollout inside the same session.

## Why

Releases are where **timing and comms** meet **automation**: tag + notes trigger **CI/CD and GitOps**; agents ship **code** and stop active tracking after release artifacts are cut, while downstream async deploy confirmation arrives in Slack.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BimRoss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
