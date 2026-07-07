## link-extension-for-xfn

> - Slug: link-extension-for-xfn

# CLAUDE.md — Link Extension for XFN

## Project

- Slug: link-extension-for-xfn
- Text Domain: link-extension-for-xfn
- Prefix: lexfn\_
- Min WP: 6.4 | Min PHP: 8.2 (upgrading from 7.4)
- Repo: https://github.com/courtneyr-dev/link-extension-for-xfn

## What It Does

Extends Gutenberg link interface with XFN 1.1 relationship options across all link-supporting blocks. Triple interface (toolbar, inspector, link popover), collapsible sections, visual pills, mutual exclusivity validation. Stores data in HTML rel attribute. 53.3% JavaScript.

## Standards

Same as Post Kinds: WPCS, Security Trinity, i18n, apiVersion 3.

## WP 7.0 Upgrade — Branch: feature/wp70-api-integration

### Priority Order

1. Abilities API — add/remove/get/validate/suggest relationships
2. Block Bindings — xfn/relationship-data source
3. Interactivity API — frontend tooltips, relationship directory block, visual indicators
4. PHP-only blocks — blogroll, relationship-badge
5. WP AI Client — relationship suggestions from linked page content

### Version Gate Pattern

```php
if ( version_compare( get_bloginfo( 'version' ), '7.0', '>=' ) ) { /* 7.0 features */ }
if ( function_exists( 'wp_register_ability' ) ) { /* abilities */ }
if ( function_exists( 'wp_ai_client_prompt' ) && get_option( 'lexfn_enable_ai' ) ) { /* AI */ }
```

## Key Notes

- Currently no frontend interactivity — editor-only. The Interactivity API additions are net-new frontend features.
- XFN data lives in HTML rel attributes on `<a>` tags, not in post meta. Block Bindings source needs to parse post_content to extract relationships.
- The plugin is JS-heavy (53.3%). Interactivity API features should be for NEW frontend blocks only — don't rewrite the existing editor JS.

## Testing to capacity

- Every feature or bugfix lands with tests — write the failing test first, then make it pass.
- Cover edge and failure paths, not just the happy path.
- No OR-assertions (`assert($x === 1 || $x === 2)`) — assert the one outcome that should happen.
- No self-grading tests — a test must not be written by the same pass that decided what "correct" looks like without independent verification.
- CI green is the source of truth over local runs — a local pass with a red CI is still red.

## Security by default

- Sanitize input, validate data, escape output — the Security Trinity already named above applies to every new code path, not just existing ones.
- Secrets live in env only, never in code or commits.
- Pre-ship security review for any public-facing feature (REST routes, AJAX handlers, anything that touches user-supplied URLs or HTML).

## Accessibility floor: WCAG 2.2 AA

- Semantic HTML first; reach for ARIA only where semantics can't do the job.
- Keyboard-only pass and screen reader pass before shipping any UI change.
- This plugin already claims WCAG 2.2 AA compliance (see readme.txt) — new UI must not regress that.

## Release gate: prepare != ship

- Never cut a release, tag, or deploy without Courtney's explicit go-ahead.
- This repo has **no auto-deploy configured in `.github/workflows/ci.yml`** — CI only runs lint, PHPStan, tests, and a dependency audit. `deploy-to-wordpress-org.sh` is a standalone manual script that checks out the wp.org SVN repo and requires an interactive credential prompt (`read -p "Press Enter to continue..."`); it is not wired into any GitHub Actions workflow and does not run automatically on tag or release.
- Because deployment is manual, a merged PR or bumped version number does NOT mean the plugin shipped to wp.org — always check `deploy-to-wordpress-org.sh`'s last actual run / wp.org SVN state before assuming a release is live.

## Commit convention: Emoji-Log

Going forward, commits use exactly these seven prefixes (emoji + CAPS + imperative mood):

| Prefix | Meaning |
| --- | --- |
| `📦 NEW:` | New feature |
| `👌 IMPROVE:` | Enhancement or refactor |
| `🐛 FIX:` | Bug fix |
| `📖 DOC:` | Documentation |
| `🚀 RELEASE:` | New version |
| `🤖 TEST:` | Testing |
| `‼️ BREAKING:` | Breaking change |

## CI lint ruleset is settled canon

The `phpcs.xml.dist` ruleset and PHPStan gate (see PRs #2/#3, merged 2026-07-03) are a deliberate, already-negotiated decision matching the sibling repos (post-kinds-for-indieweb, post-formats-for-block-themes). Future sessions should **not** relitigate or "modernize" the linter config — treat the current ruleset as final unless Courtney explicitly asks to change it.

## Gotchas

- **PR merge sequencing**: always check actual PR state via `gh pr view`/`gh pr list` before merging or retargeting — stale local branch state has caused sequencing mistakes here before. Concretely: PR #2 was stacked on PR #1; #1 squash-merged first, which made #2's branch diverge, so the same four commits had to be re-opened as PR #3 rebased onto main. Verify against `gh`, not local git state.
- **Built JS artifacts must actually be committed, not just built locally**: PR #4 fixed a case where `build/interactivity/tooltip.js` was never built or committed (and `@wordpress/interactivity` was missing from `package-lock.json`, so `npm ci` couldn't even install it). Every git install shipped with dead tooltips as a result. Before shipping a change touching the Interactivity API or any `build/` output, confirm the built file is actually tracked in git, not only present in a local `npm run build` output.

---
> Source: [courtneyr-dev/link-extension-for-xfn](https://github.com/courtneyr-dev/link-extension-for-xfn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
