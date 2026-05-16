## assemblyclaw

> Instructions for AI agents working on this repository.

# AGENTS.md

Instructions for AI agents working on this repository.

## Package Manager

**Use Bun exclusively.** Do not use npm, yarn, or pnpm.

- `bun install` — not `npm install`
- `bun run <script>` — not `npm run <script>`
- Lock file is `bun.lock` — never create `package-lock.json`

This applies to the `site/` directory and any future JS/TS tooling in this repo.

## Assembly Core

See `CLAUDE.md` for full architecture rules, code conventions, and constraints.

Key points:

- All source is ARM64 assembly (`.s` files in `src/`)
- AAPCS64 calling convention, 128-byte cache-line alignment
- Arena allocator only (mmap-backed, no malloc/free)
- Test every change with `ninja test`
- Binary must stay under 32 KB stripped

## Site

The landing page lives in `site/` and uses Vite + Bun.

```bash
cd site
bun install
bun run dev       # Dev server
bun run build     # Production build
```

CI deploys to GitHub Pages via `.github/workflows/deploy.yml` using `oven-sh/setup-bun@v2`.

## Build

```bash
ninja              # Release build (optimized, stripped)
ninja debug        # Debug build (with symbols)
ninja test         # Run tests
ninja bench        # Full benchmark suite
ninja -t clean     # Remove build outputs
```

Requires macOS on Apple Silicon + Ninja + Xcode Command Line Tools. No other dependencies.

## Scripting Rules

All `.ts` scripts use **Bun-native APIs only**. Do not import from `node:fs`, `node:os`, or `node:http`.

| Use this | Not this |
|----------|----------|
| `Bun.file(p).text()` | `readFileSync(p, "utf8")` |
| `Bun.file(p).exists()` | `existsSync(p)` |
| `Bun.file(p).size` | `statSync(p).size` |
| `Bun.write(p, data)` | `writeFileSync(p, data)` |
| `Bun.serve({ fetch })` | `createServer(...)` |
| `Bun.env.FOO` | `process.env.FOO` |
| `$\`mkdir -p ${dir}\`` | `mkdirSync(dir, { recursive: true })` |
| `$\`mktemp -d ...\`` | `mkdtempSync(...)` |

`node:path` (`join`, `dirname`) is acceptable — Bun supports it natively.

## Release & Changelog

**Every meaningful change** → add a bullet to `CHANGELOG.md` under `## Unreleased`.

Do this for: new features, bug fixes, refactors, build changes, dependency updates.
Skip: whitespace-only or comment-only changes.

**Version bump:**

```bash
bun version.ts patch|minor|major   # bumps constants.inc + version.s + CHANGELOG
git push && git push --tags         # triggers GitHub Release + Homebrew formula update
```

**Version is stored in 3 places** (the script keeps them in sync):
- `include/constants.inc` — `.set VERSION_MAJOR/MINOR/PATCH`
- `src/version.s` — version string literals

The CI release workflow auto-updates `Formula/assemblyclaw.rb` with the correct SHA256 on each tagged release.

---
> Source: [gunta/AssemblyClaw](https://github.com/gunta/AssemblyClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
