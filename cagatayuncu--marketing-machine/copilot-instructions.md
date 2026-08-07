## marketing-machine

> Claim-provenance and brand-compliance harness for marketing content. Scans a host codebase for

# marketing-machine — contributor notes

Claim-provenance and brand-compliance harness for marketing content. Scans a host codebase for
product truth, enforces rules mechanically, renders cards (PNG) and video (MP4) from one HTML
contract. Full design rationale: README.md.

## Commands

```bash
npm test                        # node:test suite, no dependencies, no browser needed
node scripts/init.mjs --root <repo>   # install into a host repo (use a throwaway fixture)
node scripts/doctor.mjs         # environment check (run inside a host repo)
node scripts/lint.mjs --json    # machine-readable findings
```

## Architecture in seven lines

- `lib/detect.mjs` — everything derived from a host codebase; shared by `init` and `scan` so they
  cannot disagree. **Detection that fails returns `null`, never a default.**
- `lib/update.mjs` — toolchain freshness: installed-skill-copy sync and upstream version.
  Unknown (offline, no repo URL) is reported as unknown, never asserted as stale or current.
- `lib/config.mjs` — config discovery (walks up from cwd), validation, shared CLI arg parsing.
- `lib/browser.mjs` — the ONE Chromium locator; one-shot screenshots and the CDP frame client.
- `lib/fonts.mjs` — three-tier font resolution; missing fonts are hard failures.
- `lib/html.mjs` — the single HTML contract: `<!--FONTS-->`/`<!--TOKENS-->` markers, six brand
  roles (`--mm-*`), `{{placeholder}}` resolution. Unresolved role or placeholder refuses to render.
- `scripts/*.mjs` — thin CLIs over lib; `rules/*.json` — data, not code.

## Invariants (violating these is a bug, not a style choice)

1. No product-specific value (color, font, name, claim) ships in the package.
2. The linter holds only mechanical checks; judgment stays in `skills/marketing-machine/SKILL.md`.
3. Templates contain no hex value and no font family name — roles only.
4. Never use `\w` in rule patterns; it misses non-ASCII letters (ı ş ğ ç ö ü). Use `[^\s,.;:]+`.
5. A scene's `window.__seek(t)` must stay a pure function of `t`.
6. Rendering with a wrong or missing brand value must fail, not fall back.

---
> Source: [cagatayuncu/marketing-machine](https://github.com/cagatayuncu/marketing-machine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
