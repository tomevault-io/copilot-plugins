## radio-gateway

> Claude Code's auto-memory path is derived from the working directory:

# Claude Instructions -- Radio Gateway

## Memory

Claude Code's auto-memory path is derived from the working directory:
`~/.claude/projects/<slugified-cwd>/memory/`. For this repo at the
default clone location that resolves to
`~/.claude/projects/-home-user-Downloads-radio-gateway/memory/`, but if
the repo is cloned anywhere else the path updates automatically.

At the end of every session, and whenever a significant bug or pattern
is found, update the memory files:
- `<auto-memory-path>/MEMORY.md` — concise project overview (keep under 200 lines)
- `<auto-memory-path>/bugs.md` — bug history

Also mirror the updated files into `.claude/memory/` inside this project
directory. IMPORTANT: this repo is PUBLIC, so `.claude/memory/` is gitignored
and is NOT committed here. The memory is version-controlled in the PRIVATE repo
`ukbodypilot/claude-notes` (folder `radio-gateway/`). Never put secrets
(passwords, keys, tokens) in memory files regardless.

### Moving to a new machine
Clone wherever you like (this repo is path-agnostic; the installer
substitutes paths at install time). Memory lives in the PRIVATE repo
`ukbodypilot/claude-notes` (folder `radio-gateway/`), NOT in this public repo.
Seed both the gitignored mirror and the auto-memory path from there:
```bash
git clone git@github.com:ukbodypilot/claude-notes.git ~/claude-notes
MEM_DIR="$HOME/.claude/projects/$(pwd | sed 's|/|-|g')/memory"
mkdir -p .claude/memory "$MEM_DIR"
cp ~/claude-notes/radio-gateway/* .claude/memory/
cp ~/claude-notes/radio-gateway/* "$MEM_DIR/"
```
To push memory changes back: copy `.claude/memory/*` into
`~/claude-notes/radio-gateway/` and commit/push that private repo.

#### Do NOT mix project and machine memory (`MEMORY.md` collision)

`claude-notes` holds two different indexes, both named `MEMORY.md`:

| Scope | claude-notes path | Live source |
|-------|-------------------|-------------|
| **Radio Gateway project** (`# Radio Gateway — Project Memory`) | `radio-gateway/MEMORY.md` | `.claude/memory/` + `<auto-memory-path>/` |
| **Machine-wide** (`# Machine: user-optiplex3020`) | `memory/MEMORY.md` | `~/.claude/projects/-home-user/memory/` |

The copy commands above are only safe while `radio-gateway/MEMORY.md` is the
*project* index. On 2026-07-31 the machine-wide index had been copied over it,
so `cp .claude/memory/* ~/claude-notes/radio-gateway/` showed up as an 84-line
deletion — it would have wiped the machine notes, and the seeding command in
the other direction would have pushed the wrong index into every new clone.

**Before running either copy, check `head -1` on both `MEMORY.md` files and
confirm each says what the table above expects.** Individual non-index memory
files may legitimately exist in both places (9 of the 10 currently shared
filenames are byte-identical); `MEMORY.md` is the only genuine collision.

### Syncing gateway_config.txt between machines
`gateway_config.txt` is NOT in the repo. If missing, ask the user for the source machine's
IP/hostname and username, then fetch it:
```bash
scp user@source-ip:$(pwd)/gateway_config.txt .
```

## Project Rules
- `gateway_config.txt` is in `.gitignore` -- NEVER commit it (repo is public; it contains stream keys and passwords)
- NEVER commit Broadcastify credentials (STREAM_PASSWORD, STREAM_MOUNT) or any other secrets
- To sync config between machines: copy the file manually (scp/rsync) -- do NOT commit it
- Never commit the `bak/` directory
- Only commit when the user explicitly asks
- Never auto-push

## Mixer v2.0 Architecture (COMPLETE)
**Reference docs:**
- `docs/mixer-v2-design.md` -- architecture reference (bus types, plugin model, routing, API)
- `docs/mixer-v2-progress.md` -- development history, decisions log, test results

The v2.0 architecture uses bus-based audio routing with all radios as plugins:
- **4 bus types:** Listen, Solo, Duplex Repeater, Simplex Repeater
- **4 radio plugins:** SDRPlugin, TH9800Plugin, D75Plugin, KV4PPlugin
- **Sources own their processing** (gate/HPF/LPF/notch/gain) -- busses route clean PCM
- **Ducking is per-bus, priority-based** -- no hardcoded source name rules
- **All sinks gated by routing connections** -- visual Drawflow node editor
- **BusManager** runs routing-configured busses alongside main loop

## Gateway Link (duplex audio + command protocol)
- See `docs/gateway_link.md` for architecture, protocol, plugin system, and roadmap
- See `CHANGELOG.md` for project-wide release history
- MVP: single endpoint, duplex audio, generic AudioPlugin
- Vision: all radios as plugins, gateway as mixer + protocol hub
- Config: `ENABLE_GATEWAY_LINK`, `LINK_PORT` (default 9700)

## AllStarLink (USRP bridge) — v4.0
- See `docs/allstar_bridge.md` for the bridge-node (ASL3 container) setup + gateway-side usage
- `plugins/usrp.py` — in-gateway USRP/DVSwitch plugin; `/usrp` panel for runtime node connect/disconnect via the bridge node's AMI
- Routing UI, `/routing/levels`, `_get_radio_plugin`, and `web_routes` dispatch are now generic over `_external_plugins` — discovered plugins become routable sources/sinks automatically
- Config: `ENABLE_USRP`, `USRP_REMOTE_HOST/PORT`, `USRP_LISTEN_PORT`, `USRP_NODE`, `USRP_AMI_*`

---
> Source: [ukbodypilot/radio-gateway](https://github.com/ukbodypilot/radio-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
