## acherus-llm-chat

> **Status: shelved, shut down on purpose, not abandoned.** This note is

# Acherus LLM bridge

**Status: shelved, shut down on purpose, not abandoned.** This note is
written for someone — you, a future instance, doesn't matter — with zero
memory of any of this. Git history has the detailed story; this is just
enough to get oriented and decide what to do next.

## What this is

An AzerothCore (WotLK 3.3.5a) private server, `Acherus`, has four AI
"personas" — locked game accounts that are never actually logged in —
holding a running conversation in Trade chat. A Python process on the
host generates their lines with a local LLM (or Claude); a Lua script
inside the VM's `worldserver` delivers them as real chat packets.

## What's solid

- **Architecture.** Host (bridge, Python, generates lines) and VM
  (emitter, Lua, delivers them) talk only through one MySQL queue table,
  `chat_outbound`. Neither calls the other directly — the bridge blocking
  on an LLM call is fine because nothing on the world thread ever waits
  on it. Both sides report health into a `control` table (heartbeats,
  last error, last load time), readable from the host without needing
  the VM's own logs.
- **Everything is data, not code.** Adding a fifth persona means a JSON
  file (`personas/CREATE_A_PERSONA.md` walks through it) and a script
  run — nothing in `bridge/`, `ale/`, `sql/`, or `deploy/` names a
  specific persona. Confirmed by actually adding one (Kelgrom) that way.
- **The packet approach works.** Hand-built `SMSG_MESSAGECHAT`, sender
  GUIDs that resolve to real (never-logged-in) characters via
  `sCharacterCache` — no game-side hacks, no logged-in bot accounts.
- **Four personas, distinct and working**: Aldrethis (over-extrapolates
  from real lore), Nixxel (cross-franchise crackpot theories, contained —
  see bleed guard below), Wick (new player, believes whoever spoke last),
  Kelgrom (his own war story grows a little every telling).
- **Pacing and timing.** A topic lifecycle (pick → turn budget → retire →
  real quiet → next), reply chains that actually sustain a back-and-forth
  instead of one reply and silence, a guarantee that direct questions get
  answered, and delivery timing (`chat_outbound.send_after`) fully
  decoupled from LLM generation latency — the conversation's rhythm is no
  longer just "how fast did the model respond." All of it goes through
  one `TIMING_SCALE` knob in `config/.env` for fast local testing.
- **Resilience.** The bridge pauses generating (rather than building an
  unbounded backlog) if the emitter looks dead or the queue backs up;
  `deploy/acherus.sh` gives you start/stop/restart/status/test/clear
  instead of raw `systemctl`/`mysql` calls.

## What's explicitly unfinished — and why

**Line quality.** Personas still tend to open a reply with the topic
keyword ("wrathgate...", "the tournament...") rather than genuinely
engaging with what the previous line actually said, and the model still
occasionally invents a proper noun. This is a known gap, not a bug that
slipped through — the last work phase was deliberately scoped to fixing
*structure* (does an exchange sustain itself, does timing feel natural,
is delivery reliable), not to making individual lines smarter. That's
real, separate work for whoever picks this up next.

## Two clear next steps, not started

1. **Move off Trade** to an actual joinable world/global channel, instead
   of the current shortcut (broadcasting to every online Horde player via
   `GetPlayersInWorld`, since ALE exposes no channel-membership API).
2. **A conversational-depth pass** — the line-quality gap above. Probably
   means richer context in the prompt about what's actually being
   responded to, not just more lore.

Persistent memory, inbound (personas reacting to real players), and NPC
gossip were all discussed early on as later phases; none of them started,
and numbering for them was deliberately left unsettled — don't let
whichever of the two steps above happens first collide with that.

## It is currently shut down. Here's what "off" means and how to undo it

Everything below was verified directly (not assumed) before being
touched. Reverse it to bring the project back:

| What | State | To resume |
|---|---|---|
| VM (`acherus`, libvirt) | shut off, autostart disabled | `virsh start acherus`, or however you'd normally boot it |
| `worldserver`/`authserver` | not running (VM is off) | start in tmux — see README's cold-start steps, cwd matters |
| `acherus-bridge` (host, systemd --user) | stopped, **disabled** | `systemctl --user enable --now acherus-bridge`, or just `deploy/acherus.sh start` after the VM is up |
| `ollama` (host, systemd --user) | stopped, **disabled** | `systemctl --user enable --now ollama` |
| `systemd --user` linger | off (was always off, never enabled) | not required for any of the above — only matters if you want these surviving logout |
| GPU VRAM | released (was ~5.3GB resident, now ~0) | comes back automatically once Ollama starts and the bridge generates its first line |

The bridge and Ollama were both still running when this shelving pass
started, despite Acherus/the VM already being shut down — worth knowing
that "stopped" and "disabled" don't imply each other, and that checking
`systemctl --user status` directly beats assuming from memory.

## Where to actually look

- `README.md` — cold-start setup instructions (still current, this is
  just the "why" note).
- `personas/CREATE_A_PERSONA.md` — how to add a character.
- `config/.env.example` — every tunable, with what each one does.
- `deploy/acherus.sh` (no args) — the operational command reference.
- Git log — every phase's actual reasoning, verification, and numbers.

---
> Source: [VX-Lo/acherus-llm-chat](https://github.com/VX-Lo/acherus-llm-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
