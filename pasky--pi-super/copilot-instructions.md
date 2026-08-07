## pi-super

> pi-super is a **long-running server** (usually `systemctl --user` — see

# Working on pi-super

## A change is not live until the server is restarted

pi-super is a **long-running server** (usually `systemctl --user` — see
`contrib/pi-super.service`). Half of a change deploys itself and half does not, which makes
for a genuinely nasty debugging session:

- **client** (`src/client/**`): served from `dist/client` on disk per request, and the
  build hash in `hello`/`pong` makes open phones reload themselves. Lands after `npm run
  build`, no restart.
- **server** (`src/server/**`, `src/meta/**`): lives in the running process. A stale server
  keeps happily serving the *new* client, then **silently ignores** every new control
  message (`onControl`'s `switch` has no `default`) and every new query parameter. The
  symptom is "the new feature does nothing at all", with no error in any log.

So, after touching server code:

```sh
npm run build && systemctl --user restart pi-super   # or restart however it was started
```

`contrib/hooks/post-commit` does this automatically (`git config core.hooksPath
contrib/hooks` to enable). The server also reports `stale: true` in `hello` when
`dist/server/server.js` is newer than the process, and the client nags about it — but do
not rely on that, it only tells you afterwards.

**Green tests do not mean the running instance works.** Every harness
(`tests/mobile-shots.mjs`, `tests/smoke.mjs`) spawns its *own* server from fresh `dist`, so
it cannot see a stale live process. When a change is meant to be tried on the phone, say
explicitly that the server was restarted.

## Testing

- `npm test` — unit tests, no tmux needed. **Always run it under `timeout`**: a bug that
  leaks a child process hangs `node --test` forever rather than failing.
- `node tests/mobile-shots.mjs` — real headless Chrome with Pixel metrics against a real
  throwaway tmux; drives touch gestures, writes screenshots to `/tmp/pi-super-shots/` and
  runs layout/behaviour checks. This is the only place phone layout is verified — look at
  the screenshots, not just the PASS lines.
- `node tests/smoke.mjs <url> <token>` — e2e against a live tmux; needs a throwaway session
  (it types into panes and kills windows). Extract the token with
  `grep -oP 'token=\K\S+'`, or start the server with `--auth token`.

## Conventions

- No formatter is configured (prettier's defaults fight the house style: tabs, wide lines).
  Match the surrounding code; do not reformat files you touch.
- Comments explain *why*, especially the non-obvious constraint that forced the shape of
  the code. Do not narrate what the next line does.
- Commit with scoped `git add <paths>`. `git add -A` here has already swept up an editor
  swap file from a live editing session.

---
> Source: [pasky/pi-super](https://github.com/pasky/pi-super) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
