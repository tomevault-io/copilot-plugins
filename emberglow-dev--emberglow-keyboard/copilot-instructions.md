## emberglow-keyboard

> Guidance for Claude (and Claude Code) working in this codebase. Read this first.

# CLAUDE.md — working in the Emberglow repo

Guidance for Claude (and Claude Code) working in this codebase. Read this first.

## What this is

Emberglow lights a **Keychron Q10** based on Claude activity, over the **VIA
protocol (QMK raw HID)**. Two front ends share one lighting engine:

- **CLI** (`emberglow set <state>`) — called by **Claude Code hooks**.
- **Flask server** (`emberglow serve`) — receives **Anthropic Managed Agents
  webhooks**, verifies their HMAC signature, and drives the same engine.

## Hard-won knowledge (read before touching `keyboard.py`)

**The VIA lighting protocol has two incompatible dialects, and the Q10 speaks the
older one. Always handshake the version first — never assume the dialect.**

- **v3** (protocol version ≥ 11): `set = [0x07, channel, value_id, data…]`, value
  IDs `1–4`, RGB-matrix channel `3`.
- **v2** (protocol version ≤ 10 — the Q10 reports **10**):
  `set = [0x07, value_id, data…]`, value IDs `0x80–0x83`, **no channel byte**.

Both use command byte `0x07`, and the firmware **echoes every packet back even
when it ignores it**. So sending v3 packets to a v2 board looks perfect — clean
echo, no error — while doing nothing, and reads return `0` on a visibly-lit
board. The telltales that cracked this: reads were `[0]` on a glowing keyboard
(brightness can't be 0 on a lit board), and the version line said `10` when v3
would be `11+`.

`keyboard.detect_dialect()` reads version command `0x01` (the one packet
identical across all VIA versions), picks v2/v3 framing, and `_verify()` does one
read to confirm the reply echoes the request header. This runs before every
command via `Keyboard.apply()`. Do not bypass it with hardcoded framing — that
is the exact bug the next contributor would reintroduce. The original `relay.py`
POC is v3-only and does **not** work on this board; it's kept for reference only.

A second trap: a **probe that only changes the effect index** is nearly
invisible against whatever the board was already doing. `emberglow probe` forces
full-brightness red on every step so each effect change is unmistakable.

## Layout

```
emberglow/
  states.py     Pure data: LightingState, STATES, EVENT_STATE, state_for_event().
                Add/retune states and event mappings HERE. No I/O.
  keyboard.py   The ONLY module that touches hardware. VIA framing + Keyboard class.
                Keyboard(open_device=..., state_file=..., sleep=...) — all injectable.
  server.py     create_app(keyboard, signing_key, allow_unverified) Flask factory.
  cli.py        argparse entry point (`emberglow`), subcommands set/serve/status/probe/...
tests/          Unit (fake HID), server (test client), e2e (real socket). conftest.py
                has FakeHidDevice, RecordingKeyboard, and sign_webhook().
examples/       Claude Code hook settings + a venv wrapper script.
docs/           Architecture, VIA protocol, webhook, contributing docs.
relay.py, enumeration.py, test-keyboard.py   Original standalone POC scripts,
                superseded by the `emberglow` package. Kept for reference.
```

## The states and where they come from

`STATES` (in `states.py`) defines four looks. The two *active* states pulse
(breathing effect): `working` is **pulsing blue**, `needsyou` is **pulsing
orange** (`HUE_ORANGE = 12`). The two *terminal* states are solid: `done` green,
`failed` red. `EVENT_STATE` maps Anthropic webhook `data.type` values to state
names:

| Event (`data.type`) | State |
|---|---|
| `session.status_run_started`, `deployment_run.started` | `working` |
| `session.status_idled`, `session.thread_idled` | `needsyou` |
| `session.status_terminated`, `deployment_run.failed`, `vault_credential.refresh_failed` | `failed` |
| `session.outcome_evaluation_ended`, `deployment_run.succeeded` | `done` |

Claude Code hooks map differently (there are no webhook events locally):
`UserPromptSubmit`→`working`, `Notification`→`needsyou`, `Stop`→`done`. See
`examples/claude-code-settings.json`.

`done` is special-cased in `Keyboard._apply`: it flashes green, then restores the
lighting snapshot saved on the first takeover (`~/.emberglow_state.json`).

## How the keyboard is detected / versioned

The Q10 exposes several HID interfaces; we use the QMK raw-HID one
(`usage_page=0xFF60`, `usage=0x61`) at VID `0x3434` / PID `0x01A1`
(`emberglow enumerate` lists them). `keyboard.protocol_version(dev)` sends the
VIA `CMD_GET_PROTOCOL_VERSION` (`0x01`) and reads the reply's bytes 1-2 as a
big-endian 16-bit **VIA protocol version**. `detect_dialect()` uses that version
to choose v2 vs v3 lighting framing (see Hard-won knowledge above — the Q10 is
v2) and `emberglow status` prints both the version and the chosen dialect. Effect
*indices* (`KB_BREATHING_EFFECT`, `KB_SOLID_EFFECT`) are firmware-specific; find
them with `emberglow probe` (confirmed on this Q10: `KB_SOLID_EFFECT=1`).

## Common tasks

- **Add a lighting state:** add an entry to `STATES` in `states.py`; if a webhook
  should trigger it, add to `EVENT_STATE`. Add a case to `test_states.py`. No
  hardware code changes needed unless it needs special behavior like `done`.
- **Change what a webhook does:** edit `EVENT_STATE` only.
- **Change VIA framing / add a command:** `keyboard.py`. Keep functions taking an
  injectable `dev` so tests can use `FakeHidDevice`.
- **Run it against Claude Code:** `emberglow set <state>` — safe to call anytime;
  it opens the board, applies, closes.

## Testing (do this before committing)

```bash
pytest                 # 24 tests, no hardware needed
pytest -m hardware     # opt-in: flashes each state on a real board
```

Tests never require a physical keyboard by default — `FakeHidDevice` records the
VIA writes and `sign_webhook()` (conftest) produces genuinely-signed payloads.
When adding behavior, prefer asserting on `fake_device.set_values` (decoded
`(value_id, data)` tuples) over raw byte lists.

## Gotchas

- Close VIA before running anything that opens the device.
- The webhook body must be verified from the **raw request bytes** — don't
  re-serialize JSON before `webhooks.unwrap()`.
- Signature verification needs the `anthropic[webhooks]` extra (already a
  dependency). It reads `ANTHROPIC_WEBHOOK_SIGNING_KEY` from the environment.
- A missing keyboard must never fail a hook or cause webhook retries — the server
  returns 2xx and the CLI wrapper exits 0.

---
> Source: [emberglow-dev/emberglow-keyboard](https://github.com/emberglow-dev/emberglow-keyboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
