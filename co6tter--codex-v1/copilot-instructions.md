## codex-v1

> > **Guidelines for contributing to the *Agents* code‑base**

# AGENTS.md

> **Guidelines for contributing to the *Agents* code‑base**

---

## Table of Contents

- [AGENTS.md](#agentsmd)
  - [Table of Contents](#table-of-contents)
  - [Purpose](#purpose)
  - [Branch Strategy – Gitflow](#branch-strategygitflow)
    - [Typical Workflow](#typical-workflow)
  - [Commit Messages – Conventional Commits](#commit-messagesconventionalcommits)
    - [Commit Format](#commit-format)
    - [Allowed Types](#allowed-types)
    - [Example](#example)
  - [Pull Request Process](#pull-request-process)
  - [Versioning \& Releases](#versioning--releases)
  - [CI / CD Pipeline](#cicd-pipeline)
  - [Code Review Checklist](#code-review-checklist)
  - [Git rules](#git-rules)
  - [References](#references)

---

## Purpose

This document establishes the workflow and coding standards for the **Agents** project. It ensures every contributor follows a predictable branching model and a uniform commit message convention, enabling easier code review, efficient release management, and clearer project history.

---

## Branch Strategy – Gitflow

We follow the classic **Gitflow** branching model:

| Branch      | Role                                      | Typical Naming         | Merge Target        |
| ----------- | ----------------------------------------- | ---------------------- | ------------------- |
| `main`      | Stable, production‑ready code             | n/a                    | n/a                 |
| `develop`   | Integration branch for completed features | n/a                    | `main` (on release) |
| `feature/*` | New features or experiments               | `feature/awesome‑idea` | `develop`           |
| `release/*` | Preparation for a versioned release       | `release/v2.1.0`       | `main` & `develop`  |
| `hotfix/*`  | Critical fixes in production              | `hotfix/urgent‑patch`  | `main` & `develop`  |

### Typical Workflow

```bash
# 1. Start a feature
git checkout develop
git pull origin develop
git checkout -b feature/your‑feature

# 2. Commit your work (see Conventional Commits below)
# ...code & commit...

git push -u origin feature/your‑feature

# 3. Open a PR → develop
# 4. After approval, merge via **Squash & Merge** (preserves clean history).
```

> **Tip :** Keep feature branches short‑lived (< 1 week) and rebased on `develop` regularly to avoid merge headaches.

---

## Commit Messages – Conventional Commits

We enforce the **Conventional Commits** specification ([https://www.conventionalcommits.org/](https://www.conventionalcommits.org/)) to automate changelog generation, semantic versioning, and release notes.

### Commit Format

```
<type>[optional scope]: <subject>

[optional body]

[optional footers]
```

* **type** – one of the approved types (see list below)
* **scope** – section of the code affected (`agent`, `docs`, `ci`, etc.)
* **subject** – concise description (< 50 chars), **imperative mood**

### Allowed Types

| Type       | Use for                                                 |
| ---------- | ------------------------------------------------------- |
| `feat`     | Introducing a new feature                               |
| `fix`      | Bug fix                                                 |
| `docs`     | Documentation only                                      |
| `style`    | Code style (formatting, white‑space)                    |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf`     | Performance improvements                                |
| `test`     | Adding or correcting tests                              |
| `build`    | Build system or external dependencies                   |
| `ci`       | CI / CD configuration                                   |
| `chore`    | Maintenance tasks                                       |
| `revert`   | Reverting a previous commit                             |

### Example

```
feat(agent): add retry logic to conversation handler

* Adds exponential back‑off with jitter
* Updates unit tests

BREAKING CHANGE: The handler now rejects on max retries instead of returning null.
```

---

## Pull Request Process

1. **Draft First :** Open a *Draft PR* early to signal upcoming work.
2. **Link Issues :** Prefix the PR description with `Closes #<issue‑number>` (or `Refs #<id>` for partial work).
3. **One Logical Change :** Keep the PR focused – avoid mixing unrelated fixes or features.
4. **Checks Must Pass :** PRs are blocked until CI succeeds and all review comments are addressed.
5. **Squash & Merge :** Use GitHub’s *Squash & Merge* button to keep a tidy history under one Conventional Commit‑styled message (edit it before finalizing).

---

## Versioning & Releases

* We follow **Semantic Versioning 2.0.0** (`MAJOR.MINOR.PATCH`).
* A release branch triggers the **Release CI** that:

  1. Runs full test suite.
  2. Generates and commits `CHANGELOG.md` from Conventional Commits.
  3. Tags the commit (`vX.Y.Z`).
  4. Publishes artifacts (containers, packages, docs).

Hotfix branches mirror the release flow but start from `main`.

---

## CI / CD Pipeline

| Stage                | Purpose                                                              |
| -------------------- | -------------------------------------------------------------------- |
| **Lint**             | Enforce code style / commit message patterns                         |
| **Test**             | Run unit, integration, and e2e tests                                 |
| **Build**            | Compile TypeScript, bundle assets                                    |
| **Security Scan**    | Check vulnerabilities (e.g., `npm audit`)                            |
| **Deploy / Publish** | Automatically deploy to staging or production upon merging to `main` |

Configuration resides in `.github/workflows/*`.

---

## Code Review Checklist

* [ ] Code compiles and tests pass
* [ ] Follows design guidelines and architecture
* [ ] No obvious security risks
* [ ] Documentation updated
* [ ] All TODOs resolved or tracked

---

## Git rules
- ALWAYS create branches that respect **git-flow**:
  - `feature/<slug>` for new features
  - `bugfix/<slug>` for normal fixes
  - `hotfix/<slug>` for urgent patches
- Branch names MUST be ASCII-only, kebab-case.
  - Transliterate non-ASCII (日本語→romaji) when needed.
- When opening a PR, set an **English title**; the Japanese description can follow in the body.

---

## References

* **Gitflow** – [https://nvie.com/posts/a-successful-git-branching-model/](https://nvie.com/posts/a-successful-git-branching-model/)
* **Conventional Commits** – [https://www.conventionalcommits.org/](https://www.conventionalcommits.org/)
* **Semantic Versioning** – [https://semver.org/](https://semver.org/)

---

*Questions or improvements? Create an issue on GitHub!*

---
> Source: [co6tter/codex-v1](https://github.com/co6tter/codex-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
