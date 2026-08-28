## perch

> Written for agents — Claude Code, codex, Hermes — and for people who would

# Working on Perch

Written for agents — Claude Code, codex, Hermes — and for people who would
rather read one page than the whole tree.

Perch is a desk instrument: a Raspberry Pi with a 7-inch 800×480 touchscreen
showing a grid of widgets, plus a config app served to your laptop from the
same process.

## Layout

```
apps/kiosk        the screen. 800×480, touch, no chrome
apps/console      the config app. Drag layout, settings, AI
packages/ui       design tokens + primitives. Everything visual starts here
packages/board    the widget grid, shared by both apps
packages/shared   zod schemas: manifest, config, websocket protocol
server            hono + ws on port 7878. One process
widgets           one folder per widget. The part people extend
agent             the reporting script other machines run
install           systemd unit, kiosk launcher, udev rule, installer
```

## Commands

```bash
npm run dev         # server + both apps, hot reload
npm run registry    # re-scan widgets/ after adding a folder
npm run typecheck   # must pass
npm test            # node:test via tsx — guard, manifests, sanitize
npm run build
PERCH_TARGET=user@pi npm run deploy
```

## If you are adding a widget

Read **[docs/WIDGETS.md](docs/WIDGETS.md)** — it is the contract and it is
complete. The short version:

1. `widgets/<id>/widget.json` — manifest. Its `config` block generates the
   whole settings UI; you do not write a settings screen.
2. `widgets/<id>/Widget.tsx` — default-export a component. **Render a `Tile`
   from `@perch/ui` as your root**, compose the other primitives, and use
   tokens rather than literal colours or sizes.
3. `widgets/<id>/server.ts` — optional. Fetches data, holds secrets, and
   exports `action` if the widget does things as well as showing them.
4. `npm run registry && npm run typecheck`

Copy `widgets/weather/` for a fetching widget, `widgets/homeassistant/` for
one with controls, `widgets/clock/` for one with no server side.

## House style

- **Tokens, never literals.** A hard-coded colour becomes a torch at 11pm,
  because the palette inverts at dusk.
- **Numbers are mono and tabular.** A value whose digits change width as it
  updates is a visible bug.
- **`--signal` means "live right now"** and nothing else. It is the only
  accent, which is why one amber tick reads across a room.
- **One thing animates at a time**, and only while something is happening.
- **Errors are sentences.** They land on a 7-inch screen in front of someone
  who cannot see your stack trace.
- **Handle `data === null`.** The screen paints before any fetch finishes.
- **Secrets are `type: "secret"`.** They never reach a browser, and the server
  strips them out of any config it is handed — do not route around that.

## Things that will bite you

- The Pi runs **Wayland + labwc**, not X11. `xset` silently does nothing, and
  Chromium needs `--ozone-platform=wayland` or touch lands in the wrong place.
- Pi OS runs `labwc -m`, which *merges* autostart files — a user autostart
  does not replace the system one, so the desktop starts anyway.
- `pkill -f chromium` over SSH kills the SSH session, because `-f` matches the
  shell running it. Use `pkill -x chromium`.
- USB audio card indexes move between reboots. Find devices by name.
- Ollama streams reasoning into a separate field; on a small model it will eat
  the whole token budget before the answer starts. Perch sends `think: false`.
- 800×480 is genuinely small. Check a change on the device, not in a desktop
  browser window resized to look like one.

---
> Source: [mayukh4/perch](https://github.com/mayukh4/perch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
