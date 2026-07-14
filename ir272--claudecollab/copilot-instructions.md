## claudecollab

> Make a Claude Code session multiplayer: host CLI wraps Claude in a PTY, a relay

# claude-share — project context

Make a Claude Code session multiplayer: host CLI wraps Claude in a PTY, a relay
forwards bytes, guests join from a browser. See README.md for usage/architecture.

## Deployed state (2026-07-11)

- Relay live at **https://claudecollab.org** (Fly app `claudeshare`, region dfw;
  claudeshare.fly.dev still answers). Certs: apex + www, Let's Encrypt via fly
- Host command: `node packages/cli/bin/claude-share.js --relay ssh://claudecollab.org:2222`
  (ssh door rides the same A record; needs CLAUDE_SHARE_SECRET in the environment)
- Relay ssh fingerprint (pin/verify): `SHA256:K2JkcwxqpWo5F+CP5q5Ke7RGTZYgjJ7tE/ajEajxdxY`
- **Exactly ONE machine, always** (`fly scale count 1`) — rooms live in the relay
  process's memory; Fly auto-adds a second machine on fresh deploys, scale it back
- A redeploy/restart ends all live rooms (hosts drop to solo) — deploy between sessions
- Deploy with `fly deploy` from the repo root; the dashboard's GitHub deployer fails opaquely
- Dedicated IPv4 168.220.81.240 ($2/mo, required for the raw-TCP ssh door) + IPv6
- `HOST_KEY` secret set (regen: `node packages/relay/bin/serve.js --make-key`)
- ROOM_SECRET set on Fly (2026-07-11) — room creation is gated; Ian's
  CLAUDE_SHARE_SECRET lives in his ~/.zshrc
- The CLI pins the relay's ssh fingerprint on first connect
  (`~/.claude-share/known_relays.json`, TOFU; `--fingerprint` pins explicitly;
  loopback exempt). Rotating HOST_KEY makes every host refuse until they clear
  the pin — rotate deliberately

## Dev & testing

- `npm test` — full suite, keep it green
- node-pty spawn-helper exec-bit gotcha — FIXED 0.1.3 (2026-07-13). Installers
  that skip build scripts (npm allow-scripts / ignore-scripts, pnpm default) ship
  spawn-helper non-executable → "posix_spawnp failed" on first spawn. startPty()
  now self-heals at RUNTIME via `ensureSpawnHelperExecutable()` in pty.js (chmods
  every prebuilds/*/spawn-helper before spawning; a postinstall can't — same
  policies block it). The error hint stays as a last resort if chmod can't (perms).
- `scripts/rig.sh` — local relay + host wrapping a FAKE claude (test/fixtures/fake-claude.cjs),
  ports **2226 (ssh) / 8788 (web)**; `scripts/rig-real.sh` — same with the real claude
- Never use ports 2222/8787/8080 for test rigs — those are Ian's live/dev ports
- Verify UI changes by driving a real browser tab against the rig and screenshotting;
  prefer scripted ws clients / synthetic events over typing into real windows

## Conventions

- Fast direct iteration: edit → verify live → screenshot → commit. No heavy orchestration.
- Concise conventional commits. No AI attribution, no Co-Authored-By trailers.
- If something looks broken while working, say so instead of silently working around it.

## Backlog (parked, in priority order)

1. Direction DECIDED 2026-07-11: **free open-source project, not a business.**
   MIT license (LICENSE committed). Domain **claudecollab.org** = landing page
   with a GitHub link; the deliverable is "install one command, run it" (makes
   backlog #4, the Rust single binary, the headline item). No accounts, no
   payments, no hosted platform. Trademark risk accepted knowingly by Ian —
   OSS/free does NOT immunize the name (Clawdbot was free OSS and was still
   forced to rename); if traction draws enforcement, rename then. Contingency:
   keep the brand SKIN-DEEP (domain + landing copy only — never in binary names,
   protocol strings, or config paths). Pre-cleared fallback: **barge.sh**
   (available + collision-free 2026-07-11; coterm.co, ptyparty.co also clear).
   Framing rule regardless: "collaborate on a live session," never "share your
   Claude subscription."
2. DONE 2026-07-11: claudecollab.org bought (Namecheap), certs issued (apex+www),
   A/AAAA → dedicated IPs, PUBLIC_URL switched, ROOM_SECRET activated — one deploy
2a. Community relay (decided direction 2026-07-11): claudecollab.org doubles as the
   free public relay so OSS users get "install → run → share a link" with zero
   infra (the tmate/sshx/Syncthing model; terminal bytes are cheap, ~$7/mo until
   real traction). A community relay runs WITHOUT ROOM_SECRET — protection is the
   maxRooms lid + per-IP knock limits; the Rust CLI will default its --relay here.
   Prereq feature — DONE 2026-07-11: **optional per-room join password**
   (`--room-password`; HELLO `pass`; ssh masked prompt + web reveal-on-demand
   field; wrong guesses burn tryKnock slots; host tab exempt; knock/admit stays
   the final gate). Keep Ian's private secret-gated mode working from the same
   code path.
3. DONE 2026-07-11 (code + ops): relay hardening — HELLO room secret + relay-key
   pinning shipped; ROOM_SECRET live on Fly
4. Rust port — PARKED 2026-07-11 (Ian: "isn't necessary at all"). The audience
   runs Claude Code, an npm package — they have Node by definition, so npm IS the
   native install channel and the single-binary thesis dissolved. The `rust/`
   workspace + `collab-protocol` crate (spec-port of protocol.js, 7 tests green)
   stays in-repo as the restart point. Revisit only on a concrete trigger:
   real non-Node demand, relay scale pain, or a brew-distribution moment.
4a. LAUNCH PATH: ✅ npm published 2026-07-11 (**@claudecollab/cli@0.1.0**, org
   scope under Ian's npm acct `iroyballer`; unscoped `claudecollab` permanently
   blocked — npm similarity rule vs `claude-collab`, a real 2025 project by
   Peter Yuqin. Bins `collab` + `collab-relay`, default relay claudecollab.org,
   publishConfig public). Remaining, in order: landing page at / →
   README-for-strangers → open the community relay (drop ROOM_SECRET; maxRooms +
   lockouts + room passwords are the abuse lids) → repo public.
5. Agent door: expose the guest protocol as a machine interface (MCP server —
   join_room / read_screen / wait_for_idle / send_prompt / answer_ask) so an
   EXTERNAL agent can supervise a session as an outer loop: independent judge
   holding the goal, re-prompting on idle until machine-checkable done-criteria
   pass (tests green, spec ticked), able to /clear or re-anchor a rotted session
   from outside. Agents join like any guest (knock/admit, prompter at most, never
   host; pause/kick apply) and browsers stay the human-watching surface — agents
   speak the protocol, not screenshots. MUST be explicit host opt-in and framed/
   defaulted as supervision, not unattended automation: continuous agent-driven
   prompting on a Pro/Max-billed session is the OpenClaw pattern Anthropic banned
   (Apr 2026) — steer this mode toward API-key billing. Free spin-off already true
   today, just needs saying on the landing page: solo remote access (open your own
   room from your phone, answer asks from anywhere).

## Hardening batch (2026-07-11, after live-test + codex adversarial review)

DONE this pass:
- **Pause actually pauses** — routeSend now blocks every Claude-bound guest send
  while paused (the {ui,command} path bypassed it; onKey already did). Regression
  test in overlay-state.test.js.
- **Host-seat binding** — a leaked host LINK no longer grants host. Browser mints
  a per-browser `seat` secret (localStorage, never in the URL); CLI binds the seat
  to the first browser (webhost:<token>:<seat>) and refuses token+wrong/absent
  seat. New-device handoff: a refused attempt arms a 60s window where Ctrl-G in the
  HOST TERMINAL releases the seat. e2e proves a stolen-link seat is refused.
- **Honest docs** — README no longer says "kicked out for good" / "stores nothing".

DEFERRED BY IAN'S CHOICE ("we don't need all that security — password + admit/deny
is the model") — revisit only if the threat model changes:
- Durable bans: kicked web guests rejoin via new token/incognito/new network;
  keyless-ssh kicked guests reconnect. No account system to bind to.
- Flood caps: protocol decoder only caps newline-free buffers, so a huge COMPLETE
  json line (giant key/ui/state payload) is parsed unbounded; pending-knock spam
  can flood the host with cards without hitting the room cap.
- Fly proxy client IP: knock lockout uses req.socket.remoteAddress = Fly's proxy,
  not the real client — per-IP limits misbehave in prod (need X-Forwarded-For).
- Room password rides the WS query string + localStorage (history/proxy leak).

KNOWN BUGS NOT YET FIXED (not security; surfaced by codex — decide next):
- `/end` always writes session.md even on the browser's "Just end" (overwrites an
  existing session.md when the host chose NOT to save). claude-share.js ~695.
- Bracketed-paste text isn't escape-sanitized before landing in join cards /
  session.md — a pasted prompt could inject terminal escapes. drafts.js/card.js.

## Machine-local cleanup owed (Ian's old Mac)

`/etc/hosts` has a pin `168.220.81.240 claudeshare.fly.dev` (ISP DNS had cached an
NXDOMAIN) — remove once DNS resolves naturally. New machines don't need it unless
they hit the same stale resolver.

---
> Source: [ir272/claudecollab](https://github.com/ir272/claudecollab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
