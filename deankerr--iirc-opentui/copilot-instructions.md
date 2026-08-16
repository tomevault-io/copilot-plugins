## iirc-opentui

> Explore building a TUI IRC Client using bun, iirc-lib, and OpenTUI with Solid.js.

# iirc-opentui

Explore building a TUI IRC Client using bun, iirc-lib, and OpenTUI with Solid.js.

@CONTEXT.md

- See `docs/` for architecture and feature information.
- Activate the `opentui` skill
- Activate the `solid-js-best-practices` skill (adapt guidance for the OpenTUI medium)
- Solid JS state management docs/solid-js/

## Status

Prototype mode. No production, no backwards compatibility, no data migrations.

## Local `ngircd` IRC server

- hostname: `localhost`
- port: `6667`
- channel: `#dev`
- join channel `#auto` to experience a series of IRC events from a scripted user client.

## Commands

- Use `bun run fix` type check, lint, and format.
  - Do not use `tsc`.
  - Inline disables may be used if the reasoning is justified.
- `bun run test`, not `bun test`

## IRC Notes

- `NICK` and `QUIT` are not scoped to a channel by IRC. It is up to the client to fan out event messages if desired.
- IRCv3 capabilities are optional enhancements. Both iirc-lib and the client must work without them and with any partially supported capability set.
- Maintain docs/iirc-lib-improvements.md when an iirc-lib limitation requires a client workaround, degrades reasonably expected IRC client behavior, or blocks a feature. Record the client impact and the library behavior or API needed; update or remove the entry when the limitation changes.

## TypeScript 6

- `@types/*` packages are manually specified `"types": ["bun"]`, only if required
- Subpath Imports support, e.g. `"#/*": "./dist/*"`, replace deep relative paths `../../utils.js` with `#root/utils.js`

---
> Source: [deankerr/iirc-opentui](https://github.com/deankerr/iirc-opentui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
