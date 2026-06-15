## herdr-web

> This is a lightweight internal onboarding note for agents working in this repo.

# Agent Onboarding (herdr-web)

This is a lightweight internal onboarding note for agents working in this repo.

## Start Here

- Read `README.md` for the product shape, bridge runtime model, and local run commands.
- Read `web/README.md` before changing the React/Vite app.
- Read `docs/vendoring.md` before touching `vendor/herdr/`.
- Web source lives in `web/src/`.
- The Herdr bridge overlay lives in `vendor/herdr/src/web_bridge.rs`.
- Other vendored overlay files are `vendor/herdr/src/main.rs`, `vendor/herdr/Cargo.toml`, and `vendor/herdr/Cargo.lock`.
- Do not treat `vendor/herdr/AGENTS.md` as repo-level guidance; it belongs to the vendored upstream tree.

## Conventions

- Keep product changes scoped to the web app and bridge overlay unless the user explicitly asks for core Herdr changes.
- Prefer small pure helpers in `web/src/` for state, launch, and protocol formatting logic that can be unit tested.
- Keep bridge command exposure narrow. Browser commands should stay allow-listed and parameter-validated in `web_bridge.rs`.
- Keep generated outputs out of commits: `web/dist/`, `vendor/herdr/target/`, `.zig-cache/`, and `zig-out/`.
- The bridge is local-first and currently has no full browser authentication. Treat LAN binding and upload behavior as security-sensitive.

## Testing

- Run `npm install --prefix web` if dependencies are missing.
- Run `npm run vendor:check` to verify the vendored layout.
- Run `npm run lint:web` for ESLint.
- Run `npm run test:web` for Vitest.
- Run `npm run build:web` for the frontend production build.
- Run `npm run bridge:test` for bridge unit tests when Zig is available.
- Run `npm run check` before committing or releasing.
- If Zig is missing, call out that bridge build/test verification could not run.

## Changelog

- Add user-facing changes to `CHANGELOG.md` under `## [Unreleased]`.
- Use these subsections when applicable: Breaking Changes, Added, Changed, Fixed, Removed.
- Add the needed subsection under `## [Unreleased]` if it is missing; do not create duplicate subsection headings.
- Include bridge compatibility and vendored Herdr refresh notes when protocol or overlay compatibility changes.
- When cutting a release, remove empty unused subsections from the released version section; keep the fresh `## [Unreleased]` template headings intact.

## Release

- Release from a clean `main` branch.
- Ensure `CHANGELOG.md` has the release notes under `## [Unreleased]`.
- Run `npm run check`.
- Run the browser smoke checklist in `docs/release.md`.
- Run `node scripts/release.mjs vX.Y.Z`.
- The release script promotes the changelog, commits, tags, pushes, creates a GitHub release from changelog notes, and opens the next `## [Unreleased]` section.
- Do not bump npm package versions or upload release artifacts until the packaging process is defined.

---
> Source: [kcosr/herdr-web](https://github.com/kcosr/herdr-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
