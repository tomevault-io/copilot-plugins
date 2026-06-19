## vapix-pp-cli

> Every VAPIX call you'd reach for, plus a local SQLite history, fleet sweep, and an MCP server no other Axis wrapper has. Trigger phrases: `fingerprint axis camera`, `discover axis cameras on subnet`, `drive PTZ camera to preset`, `list ACAP applications`, `what changed on this camera`, `use vapix`, `run vapix`.


# VAPIX — Printing Press CLI

## Prerequisites: Install the CLI

This skill drives the `vapix-pp-cli` binary. **You must verify the CLI is installed before invoking any command from this skill.** If it is missing, install it first:

1. Install via the Printing Press installer:
   ```bash
   npx -y @mvanhorn/printing-press install vapix --cli-only
   ```
2. Verify: `vapix-pp-cli --version`
3. Ensure `$GOPATH/bin` (or `$HOME/go/bin`) is on `$PATH`.

If the `npx` install fails before this CLI has a public-library category, install Node or use the category-specific Go fallback after publish.

If `--version` reports "command not found" after install, the install step did not put the binary on `$PATH`. Do not proceed with skill commands until verification succeeds.

A single Go binary for fleet operations against Axis IP cameras. It absorbs every operation in the open VAPIX wrapper landscape (param.cgi, PTZ, applications, analytics, events, MQTT bridge), keeps a per-camera SQLite history of parameters and presets, and exposes an MCP server so an LLM agent can sweep a subnet, drive a PTZ camera to a preset, or diff parameter trees from a conversation. Built around the quirks that bite real Axis fleets: the Q6325-LE basicdeviceinfo digest loop, the AXIS OS 12 ONVIF/VAPIX user-database split, and the AOA + analytics-MQTT surfaces that landed in OS 12.2.

## When to Use This CLI

Reach for vapix-pp-cli when you have a fleet of Axis cameras and a question whose answer requires hitting more than one of them, or when an LLM agent needs to script camera operations. Don't reach for it for live RTSP video pull (use VLC/ffmpeg) or for ONVIF-only Profile S/T flows (use swift-onvif or the ONVIF Device Manager).

## Unique Capabilities

These capabilities aren't available in any other tool for this API.

### Fleet operations
- **`discover`** — Sweep a CIDR range and fingerprint every Axis camera it finds in one parallel shot.

  _When an installer drops you on a network with 'somewhere between 4 and 40 Axis cameras', this turns 'go figure out the inventory' into one command._

  ```bash
  vapix-pp-cli discover 192.168.1.0/24 --json
  ```
- **`sweep param`** — Run the same param.cgi query against every camera in the local store and return one table.

  _'What hostname is each camera reporting?' takes one command instead of an ad-hoc loop you re-write every audit._

  ```bash
  vapix-pp-cli sweep param --group root.Network.HostName --json
  ```

### Local state that compounds
- **`param diff`** — Diff the current param.cgi tree of a camera against the previous snapshot stored in the local SQLite cache.

  _Answers 'did anything change on this camera in the last week?' without trusting whoever last touched the on-camera config._

  ```bash
  vapix-pp-cli param diff --host 192.168.1.33 --since 7d --json
  ```
- **`preset copy`** — Read presets from one camera and write them to another using their stored coordinates.

  _Replacing or duplicating a PTZ rig stops being a 30-minute click-through and becomes one line._

  ```bash
  vapix-pp-cli preset copy --from Q6358 --to Q6325
  ```
- **`applications drift`** — Diff the installed ACAP application set per camera against last sync; flag installs, uninstalls, version bumps.

  _Catches 'who installed an unsigned ACAP on the front-door camera' without manually walking each device._

  ```bash
  vapix-pp-cli applications drift --json
  ```

### Compound operator workflows
- **`ptz tour`** — Walk a sequence of named presets with a configurable dwell at each.

  _Manual rounds for an integrator becomes scriptable; useful for demos and for late-night perimeter sweeps._

  ```bash
  vapix-pp-cli ptz tour --host 192.168.1.33 --presets Gate,Lobby,Lot --dwell 8
  ```
- **`events watch`** — Tail the MQTT bridge for a topic and shell out to a snapshot command when it fires.

  _Captures evidence frames the second a camera detects motion — without standing up a separate event broker._

  ```bash
  vapix-pp-cli events watch --host 192.168.1.33 --topic tns1:VideoSource/MotionAlarm --on-fire 'snapshot.sh {topic} {ts}'
  ```
- **`doctor`** — Composite health check — param reachability, PTZ whoami, ACAP app list, system clock skew, current firmware.

  _Pre-flight before a demo or an installation handoff in one command._

  ```bash
  vapix-pp-cli doctor --host 192.168.1.33 --json
  ```

### Reachability mitigation
- **`device info`** — Probes param.cgi first (works on Q6325-LE where basicdeviceinfo.cgi loops on digest 401) then falls back to richer endpoints for newer OS.

  _Avoids the silent 401-loop failure mode that has bitten every team that wrote 'a quick Python script' against a Q6325-LE._

  ```bash
  vapix-pp-cli device info --host 192.168.1.32
  ```

## Command Reference

**aoa** — AXIS Object Analytics (AOA) and analytics-metadata configuration

- `vapix-pp-cli aoa aoa-config` — Get AXIS Object Analytics configuration (scenarios, triggers, classes)
- `vapix-pp-cli aoa aoa-supported` — List supported scenario types and capabilities
- `vapix-pp-cli aoa metadata-producers` — List RTSP analytics metadata producers and their per-channel state
- `vapix-pp-cli aoa metadata-sample` — Request a one-shot sample of metadata from a named producer
- `vapix-pp-cli aoa mqtt-data-sources` — AXIS OS 12.2+ — list available analytics-MQTT data sources

**applications** — AXIS Camera Application Platform (ACAP) — manage installed apps via /axis-cgi/applications/*

- `vapix-pp-cli applications list` — List installed ACAP applications: name, version, status, license
- `vapix-pp-cli applications remove` — Uninstall an ACAP application. Destructive — use --dry-run first.
- `vapix-pp-cli applications restart` — Restart an ACAP application
- `vapix-pp-cli applications start` — Start an ACAP application by package name
- `vapix-pp-cli applications stop` — Stop a running ACAP application

**device** — Device identity, capabilities, and lifecycle

- `vapix-pp-cli device info` — Fingerprint device — model, firmware, serial, MAC, AXIS OS via param.cgi (works on Q6325-LE where...
- `vapix-pp-cli device properties` — List root.Properties tree — capability flags, AXIS OS major/minor, supported features
- `vapix-pp-cli device restart` — Reboot the device. Use --dry-run first.
- `vapix-pp-cli device systemtime` — Get device system time

**events** — Event subscription and topic discovery

- `vapix-pp-cli events action-configurations` — List action configurations (SOAP GetActionConfigurations)
- `vapix-pp-cli events action-rules` — List configured action rules (SOAP GetActionRules)
- `vapix-pp-cli events action-templates` — List supported action types (SOAP GetActionTemplates)
- `vapix-pp-cli events change-virtual-input` — Trigger a virtual input event (SOAP ChangeVirtualInputState) — useful for testing rules
- `vapix-pp-cli events list-topics` — List all event topics on this device (SOAP GetEventInstances). Returns ONVIF-shaped topic tree.
- `vapix-pp-cli events recipient-templates` — List supported recipient templates
- `vapix-pp-cli events recipients` — List configured recipients

**media** — Snapshots and live image capture

- `vapix-pp-cli media snapshot` — Capture a JPEG snapshot. Use --output to write to a file; default is base64.
- `vapix-pp-cli media streamprofiles` — List configured stream profiles

**mqtt** — MQTT client + Event Bridge configuration via /axis-cgi/mqtt/*

- `vapix-pp-cli mqtt activate-client` — Activate (start) the MQTT client
- `vapix-pp-cli mqtt client-config` — Get MQTT client configuration (broker, credentials, TLS, last will)
- `vapix-pp-cli mqtt client-status` — Get MQTT client status (running, broker URL, last error)
- `vapix-pp-cli mqtt deactivate-client` — Deactivate (stop) the MQTT client
- `vapix-pp-cli mqtt event-bridge-publish` — Get current event publication map (which event topics → MQTT topics)
- `vapix-pp-cli mqtt event-bridge-subscribe` — Get current MQTT subscription map (incoming MQTT → device events)

**param** — Universal parameter tree (param.cgi). The most reliable VAPIX surface — works across every Axis device.

- `vapix-pp-cli param add` — Add a new dynamic parameter (motion windows, stream profiles, etc.)
- `vapix-pp-cli param list` — List all parameters or a specific group. Returns key=value pairs.
- `vapix-pp-cli param listdefinitions` — List parameter definitions (type, range, default) — useful for discovery
- `vapix-pp-cli param remove` — Remove a dynamic parameter
- `vapix-pp-cli param update` — Update an existing parameter. Use param.update KEY VALUE.

**preset** — PTZ preset save / rename / delete via /axis-cgi/com/ptzconfig.cgi

- `vapix-pp-cli preset list` — List all server presets on this camera
- `vapix-pp-cli preset remove` — Remove a named preset
- `vapix-pp-cli preset removeall` — Remove every server preset on this camera. Destructive — use --dry-run first.
- `vapix-pp-cli preset rename` — Rename a preset by number
- `vapix-pp-cli preset save` — Save current PTZ position as a named preset

**ptz** — Pan / tilt / zoom control via /axis-cgi/com/ptz.cgi

- `vapix-pp-cli ptz areazoom` — Area zoom — center on pixel coords + zoom factor
- `vapix-pp-cli ptz autofocus` — Enable or disable autofocus
- `vapix-pp-cli ptz autoiris` — Enable or disable autoiris
- `vapix-pp-cli ptz auxiliary` — Trigger an auxiliary function (e.g. wiper, washer, defogger). Device-specific.
- `vapix-pp-cli ptz center` — Center on pixel coordinates within the live image
- `vapix-pp-cli ptz continuouspantiltmove` — Continuous pan/tilt move at given speed (-100..100, comma-separated). Set 0,0 to stop.
- `vapix-pp-cli ptz continuouszoommove` — Continuous zoom at given speed (-100..100). 0 to stop.
- `vapix-pp-cli ptz gotoserverpresetname` — Move to a server-side preset by name
- `vapix-pp-cli ptz gotoserverpresetno` — Move to a server-side preset by number
- `vapix-pp-cli ptz info` — List supported PTZ commands on this device
- `vapix-pp-cli ptz ircutfilter` — Control IR cut filter (auto | on | off)
- `vapix-pp-cli ptz move` — Move in a named direction (up/down/left/right/upleft/upright/downleft/downright/home/stop)
- `vapix-pp-cli ptz pan` — Absolute pan position (-180..180 degrees)
- `vapix-pp-cli ptz query` — Query PTZ status: position, presets, limits
- `vapix-pp-cli ptz tilt` — Absolute tilt position
- `vapix-pp-cli ptz whoami` — Identify PTZ driver + capabilities. First call to confirm a camera is PTZ-capable.
- `vapix-pp-cli ptz zoom` — Absolute zoom position (1..9999, device-dependent)


### Finding the right command

When you know what you want to do but not which command does it, ask the CLI directly:

```bash
vapix-pp-cli which "<capability in your own words>"
```

`which` resolves a natural-language capability query to the best matching command from this CLI's curated feature index. Exit code `0` means at least one match; exit code `2` means no confident match — fall back to `--help` or use a narrower query.

## Recipes


### Subnet discovery

```bash
vapix-pp-cli discover 192.168.1.0/24 --json --select host,model,firmware,axis_os_major
```

Sweep a /24 and pull only the four fields agents actually need to plan further calls.

### What changed on this camera since last week

```bash
vapix-pp-cli param diff --host 192.168.1.33 --since 7d --json
```

Compares the current parameter tree to the snapshot you took 7 days ago — answers the audit question without manual scrolling.

### Replicate Q6358 presets onto Q6325

```bash
vapix-pp-cli preset copy --from 192.168.1.33 --to 192.168.1.32 --dry-run
```

Reads cached presets from the source and previews the saves on the destination. Drop --dry-run to commit.

### Late-night perimeter tour

```bash
vapix-pp-cli ptz tour --host 192.168.1.33 --presets Gate,Lobby,Lot --dwell 8
```

Walk three presets with an 8-second dwell each. Loop with --repeat 5 for a sustained tour.

### Snap a frame the second motion fires

```bash
vapix-pp-cli events watch --host 192.168.1.33 --topic tns1:VideoSource/MotionAlarm --on-fire snapshot.sh
```

Stages an MQTT bridge subscription and prints the suggested mosquitto_sub invocation plus the on-fire handler. Substitution tokens: {topic}, {ts}, {host}.

## Auth Setup

No authentication required.

Run `vapix-pp-cli doctor` to verify setup.

## Agent Mode

Add `--agent` to any command. Expands to: `--json --compact --no-input --no-color --yes`.

- **Pipeable** — JSON on stdout, errors on stderr
- **Filterable** — `--select` keeps a subset of fields. Dotted paths descend into nested structures; arrays traverse element-wise. Critical for keeping context small on verbose APIs:

  ```bash
  vapix-pp-cli applications list --agent --select id,name,status
  ```
- **Previewable** — `--dry-run` shows the request without sending
- **Offline-friendly** — sync/search commands can use the local SQLite store when available
- **Non-interactive** — never prompts, every input is a flag
- **Explicit retries** — use `--idempotent` only when an already-existing create should count as success

### Response envelope

Commands that read from the local store or the API wrap output in a provenance envelope:

```json
{
  "meta": {"source": "live" | "local", "synced_at": "...", "reason": "..."},
  "results": <data>
}
```

Parse `.results` for data and `.meta.source` to know whether it's live or local. A human-readable `N results (live)` summary is printed to stderr only when stdout is a terminal — piped/agent consumers get pure JSON on stdout.

## Agent Feedback

When you (or the agent) notice something off about this CLI, record it:

```
vapix-pp-cli feedback "the --since flag is inclusive but docs say exclusive"
vapix-pp-cli feedback --stdin < notes.txt
vapix-pp-cli feedback list --json --limit 10
```

Entries are stored locally at `~/.vapix-pp-cli/feedback.jsonl`. They are never POSTed unless `VAPIX_FEEDBACK_ENDPOINT` is set AND either `--send` is passed or `VAPIX_FEEDBACK_AUTO_SEND=true`. Default behavior is local-only.

Write what *surprised* you, not a bug report. Short, specific, one line: that is the part that compounds.

## Output Delivery

Every command accepts `--deliver <sink>`. The output goes to the named sink in addition to (or instead of) stdout, so agents can route command results without hand-piping. Three sinks are supported:

| Sink | Effect |
|------|--------|
| `stdout` | Default; write to stdout only |
| `file:<path>` | Atomically write output to `<path>` (tmp + rename) |
| `webhook:<url>` | POST the output body to the URL (`application/json` or `application/x-ndjson` when `--compact`) |

Unknown schemes are refused with a structured error naming the supported set. Webhook failures return non-zero and log the URL + HTTP status on stderr.

## Named Profiles

A profile is a saved set of flag values, reused across invocations. Use it when a scheduled agent calls the same command every run with the same configuration - HeyGen's "Beacon" pattern.

```
vapix-pp-cli profile save briefing --json
vapix-pp-cli --profile briefing applications list
vapix-pp-cli profile list --json
vapix-pp-cli profile show briefing
vapix-pp-cli profile delete briefing --yes
```

Explicit flags always win over profile values; profile values win over defaults. `agent-context` lists all available profiles under `available_profiles` so introspecting agents discover them at runtime.

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 2 | Usage error (wrong arguments) |
| 3 | Resource not found |
| 5 | API error (upstream issue) |
| 7 | Rate limited (wait and retry) |
| 10 | Config error |

## Argument Parsing

Parse `$ARGUMENTS`:

1. **Empty, `help`, or `--help`** → show `vapix-pp-cli --help` output
2. **Starts with `install`** → ends with `mcp` → MCP installation; otherwise → see Prerequisites above
3. **Anything else** → Direct Use (execute as CLI command with `--agent`)

## MCP Server Installation

Install the MCP binary from this CLI's published public-library entry or pre-built release, then register it:

```bash
claude mcp add vapix-pp-mcp -- vapix-pp-mcp
```

Verify: `claude mcp list`

## Direct Use

1. Check if installed: `which vapix-pp-cli`
   If not found, offer to install (see Prerequisites at the top of this skill).
2. Match the user query to the best command from the Unique Capabilities and Command Reference above.
3. Execute with the `--agent` flag:
   ```bash
   vapix-pp-cli <command> [subcommand] [args] --agent
   ```
4. If ambiguous, drill into subcommand help: `vapix-pp-cli <command> --help`.

---
> Source: [oneshot2001/vapix-pp-cli](https://github.com/oneshot2001/vapix-pp-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
