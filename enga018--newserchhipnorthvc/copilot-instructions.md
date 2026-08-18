## newserchhipnorthvc

> The app version lives in `app/index.html` as `const APP_VERSION = "vMAJOR.MINOR.PATCH"` and in `sw.js` as `SW_VERSION: vMAJOR.MINOR.PATCH` (also used as the service worker cache-busting query string). Bump it on every commit to `main`:

# Versioning policy

The app version lives in `app/index.html` as `const APP_VERSION = "vMAJOR.MINOR.PATCH"` and in `sw.js` as `SW_VERSION: vMAJOR.MINOR.PATCH` (also used as the service worker cache-busting query string). Bump it on every commit to `main`:

- **Small change** (bug fix, tweak, small addition — roughly a diff of a few dozen lines in one area): PATCH += 1.
  Example: 1.1.1 → 1.1.2
- **Large change** (new feature, significant behavior change, broad/multi-file diff): MINOR += 1, PATCH resets to 0.
  Example: 1.1.1 → 1.2.0
- Judge small vs. large from the actual diff size/scope for that push (`git diff --stat` against the previous version), not a fixed line-count rule.

**Caps**: MINOR and PATCH each cap at 20. On overflow, the overflowing number and everything to its right reset to **1** (not 0), and the number to the left increments (cascades if that also overflows):
- PATCH would exceed 20 (small change): PATCH → 1, MINOR += 1 (check MINOR overflow too).
- MINOR would exceed 20 (large change, or cascading from a PATCH overflow): MINOR → 1, PATCH → 1, MAJOR += 1.

Examples: 1.20.20 + small change → 2.1.1. 1.20.0 + large change → 2.1.1 (MINOR was already at the cap).

# Post-commit hook for automatic version bumping

A git post-commit hook automates version bumping. To install it:

```bash
./hooks/install-hooks.sh
```

The hook:
- Runs on commits to `main`, `master`, or branches starting with `claude/`
- Analyzes the diff to determine if the change is "small" or "large":
  - **Small**: single file or < 100 lines changed → PATCH += 1
  - **Large**: multiple files or ≥ 100 lines changed → MINOR += 1, PATCH = 0
- Automatically creates a follow-up version bump commit
- Skips on commits with "version bump" in the message (to avoid double-bumping)
- Updates `app/index.html`, `sw.js`, and `manifest.json`

---
> Source: [enga018/newserchhipnorthvc](https://github.com/enga018/newserchhipnorthvc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
