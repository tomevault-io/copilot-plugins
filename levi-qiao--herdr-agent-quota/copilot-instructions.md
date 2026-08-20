## herdr-agent-quota

> Notes for agents working on `herdr-agent-quota`. Read this before touching

# AGENTS.md

Notes for agents working on `herdr-agent-quota`. Read this before touching
anything that talks to Herdr.

## The rule that matters most: reading or writing a pane is not free

This plugin's whole job is to put quota numbers in Herdr's sidebar. Every
Herdr call it makes to do that lands on a pane that a human is actively
watching. Two calls are visible to the user:

- `herdr pane read <id>` — makes Herdr repaint that pane. The agent's TUI
  redraws its whole frame, which the user sees as the terminal scrolling up
  and then snapping back to the bottom.
- `herdr pane report-metadata <id>` — same repaint risk when the tokens
  actually change.

Neither is detectable from `pane get`: `offset_from_bottom` and
`max_offset_from_bottom` stay `0` throughout, because full-screen agent TUIs
have no Herdr scrollback. The viewport never moves. What moves is the agent's
own repaint. **Do not conclude "no scroll happened" from the scroll offsets.**

Concretely, this means:

1. **Never read every pane of a provider.** An event names one pane; read only
   that one. Fanning out across panes multiplies the repaints by the number of
   panes the user has open for that agent.
2. **Publish once per invocation.** Two `publish` passes in a row means each
   pane can take two metadata writes for one user action.
3. **Keep `metadata_matches` honest** (`src/herdr.rs`). It is the only thing
   stopping a no-op refresh from repainting every pane. If you add a token,
   add it to `METADATA_TOKEN_NAMES` too, or the comparison silently stops
   covering it and every refresh becomes a write.
4. **Preserve, don't clear.** When a topic read fails or finds nothing, keep
   the previously published topic. Clearing it churns the token and triggers
   a write on the next refresh, which triggers a repaint.

## Event paths, and what each is allowed to do

| Entry point | Fired by | Allowed to read panes? |
|---|---|---|
| `refresh` | startup, manual action, Grok hooks | No |
| `event` | `pane.agent_detected`, `pane.agent_status_changed` | Only the pane named in `HERDR_PLUGIN_EVENT_JSON` |
| `focus` | `pane.focused` | No |

`pane.agent_status_changed` fires **twice per turn** (idle→working on submit,
working→idle on completion). Anything `event` does, the user pays for twice
every time they press Enter. Budget accordingly.

## Event payload shapes

`HERDR_PLUGIN_EVENT_JSON` is nested and not uniform across events. `pane.focused`
carries no `agent` at all, which is why `focus` has to call `herdr pane current`:

```json
{"event":"pane_focused","data":{"type":"pane_focused","pane_id":"w1:p9","workspace_id":"w1"}}
```

`find_agent` and `find_pane_id` in `src/refresh.rs` walk the tree rather than
assuming a fixed path. Keep them tolerant — the shapes differ per event and are
not part of a stable contract.

## Debugging a "the panes are scrolling" report

Bisect from the outside in. Each step needs the user to reproduce once, so do
them in this order and don't skip ahead:

1. `herdr plugin disable herdr-agent-quota` **and** remove the `statusLine`
   entry from `~/.claude/settings.json`, then **restart the agent pane**.
   `herdr plugin disable` alone is not enough — Claude Code runs the statusLine
   command itself, independent of Herdr, and reads the setting at startup.
2. Restore the statusLine only. Scrolls → the statusLine hook is at fault.
3. Re-enable the plugin, then remove event hooks from `herdr-plugin.toml` one
   at a time, reloading with `herdr plugin disable && herdr plugin enable`
   (needed to re-read the manifest; `herdr server reload-config` does not).

To capture a real event payload, temporarily point a hook at
`sh -c "printf '%s' \"$HERDR_PLUGIN_EVENT_JSON\" > /tmp/ev.json; exec <real command>"`
so the plugin keeps working while you collect the shape.

Beware of instrumenting with a polling probe: polling `herdr pane read` at
several Hz is itself a repaint source and will contaminate whatever the user
reports while it runs.

## Verifying

```
cargo fmt
cargo test
cargo clippy --release
```

Reloading the plugin after a rebuild:

```
herdr plugin disable herdr-agent-quota && herdr plugin enable herdr-agent-quota
```

---
> Source: [levi-qiao/herdr-agent-quota](https://github.com/levi-qiao/herdr-agent-quota) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
