## autoprober

> > Read this before using an AI agent or automation script with this project.

# AGENTS.md — Operational Rules for Autoprober Agents

> Read this before using an AI agent or automation script with this project.
> The rules here are safety rules for physical machine control.

## Absolute Safety Rules

### 1. `Pn:P` in GRBL status is not the safety endstop

The CNC controller probe input is not trusted for the optical endstop.
Autoprober uses the real GRBL limit flags and an independent oscilloscope
measurement instead.

- `<Idle|MPos:...|Pn:P|...>` means the GRBL probe flag is active. Ignore it
  for safety decisions.
- `<Idle|MPos:...|Pn:X|...>` means the X limit switch is active.
- `<Idle|MPos:...|Pn:Y|...>` means the Y limit switch is active.
- `<Idle|MPos:...|Pn:Z|...>` means the Z limit switch is active.
- `Pn:PX`, `Pn:PY`, and `Pn:PZ` include a probe flag plus a real limit flag;
  use only the `X`, `Y`, or `Z` part as the real limit condition.

Any code that treats `Pn:P` as the optical endstop is wrong.

### 2. The independent endstop signal is oscilloscope Channel 4

The optical endstop is read as a voltage on Channel 4 through the oscilloscope
wrapper. Configure the oscilloscope host and port with environment variables,
not hardcoded lab addresses.

Required wiring:

- Endstop OUT to Channel 4 probe tip.
- Endstop GND to Channel 4 ground clip.
- Endstop VCC to an external 5V supply.

Voltage interpretation:

| Voltage | Meaning | Action |
|---|---|---|
| 4.5V to 5.2V | Healthy, clear slot | OK |
| 1.0V to 4.5V | Ambiguous wiring or signal fault | STOP |
| Less than 1.0V | Optical beam broken / carriage compressed | STOP immediately |
| Unreadable / disabled channel | Scope configuration fault | STOP |

Use `Scope.measure_mean(4)` or `Scope.read_endstop(channel=4)`.

### 3. Monitor Channel 4 continuously during motion

Checking Channel 4 once before a move is not enough. Motion must run with an
`EndstopMonitor` polling in the background. A trigger during a move must issue
feed hold immediately.

Example:

```python
from autoprober.cnc import CNC
from autoprober.scope import Scope
from autoprober.safety import EndstopMonitor

scope = Scope(quiet=True)
cnc = CNC()

scope.connect()
cnc.connect()

monitor = EndstopMonitor(
    scope,
    threshold=1.0,
    poll_interval=0.1,
    hold_callback=cnc.feed_hold,
)
monitor.start()
try:
    cnc.move_relative(dx=5, feed=800)
    cnc.wait_for_idle()
finally:
    monitor.stop()

monitor.require_clear()
```

### 4. On trigger: stop, report, do not recover automatically

When Channel 4 triggers or faults:

1. Stop motion with feed hold.
2. Log what was running, the CNC position if available, and the Channel 4
   voltage or error.
3. Report the stop condition to the operator.
4. Do not move again until a human operator explicitly clears the condition.

Do not automatically back off Z, jog away, retry, or continue scanning after a
trigger. Those are still motion and require operator approval.

## Startup Sequence

For a hardware session:

1. Verify USB enumeration for the CNC and microscope.
2. Run `apps/preflight.py`.
3. Verify Channel 4 is clear with no motion.
4. Home the CNC with `apps/home.py` or a monitored `$H` workflow.
5. Run `apps/calibrate.py` over a feature-rich area before coordinate-based
   scan or probe work.

If USB devices disappear after a reboot, hub change, or controller reset,
repeat the startup sequence. Do not trust prior runtime state.

## Motion Rules

- Use `autoprober.cnc.CNC`; do not open raw serial ports in app scripts.
- Use `autoprober.scope.Scope`; do not open raw SCPI sockets in app scripts.
- Use `autoprober.microscope.Microscope`; do not shell out for snapshots in
  app scripts.
- Any script that moves the CNC must use `EndstopMonitor`.
- A real `X`, `Y`, or `Z` limit pin is a stop condition.
- A hard limit alarm is not a safety strategy. Stay inside the known travel
  envelope.
- Homing must complete normally before absolute coordinate work.
- If homing appears to complete too quickly or the machine position does not
  change, stop and investigate.

Recommended feed-rate ranges:

- Clear travel: 1000 to 1500 mm/min.
- Scan-step XY moves: 500 to 800 mm/min.
- Z descent near contact: 100 to 200 mm/min.
- Calibration moves: about 500 mm/min.

Use a Z hop for XY travel when the pogo pin or carriage could drag on a target:

```python
cnc.move_relative(dz=+z_hop)
cnc.move_absolute(x=target_x, y=target_y)
cnc.move_relative(dz=-z_hop)
```

## Calibration Rules

- Calibration is runtime state, not source-controlled project data.
- Read calibration from `AUTOPROBER_RUNTIME_ROOT` through the project helpers.
- Run calibration after microscope remounting, refocus, meaningful Z changes,
  or any mechanical change that can affect field of view.
- Do not save calibration from a blank or low-feature image.

## Logging Rules

Use `autoprober.logging.log()` for meaningful actions. The log path is
configured by `AUTOPROBER_LOG_PATH`.

Recommended log sources:

- `operator` for manual/operator-driven script actions.
- `cnc` for CNC wrapper commands and responses.
- `scope` for oscilloscope wrapper activity.
- `microscope` for microscope capture/stream activity.
- `calibrate`, `preflight`, and `dashboard` for app-level events.
- `endstop-mon` for safety monitor stop conditions.

Do not commit runtime logs.

## Public Release Rules

- Do not commit lab-specific IP addresses, hostnames, usernames, SSH aliases,
  credentials, or local filesystem paths.
- Do not commit calibration files, flat-field images, microscope captures,
  stitched target maps, probe-review artifacts, or runtime logs unless they are
  explicitly prepared as public examples.
- Keep hardware addresses configurable through environment variables.
- Keep deployment instructions generic to a hardware host.
- Keep CAD files limited to the intended public printable parts.

## Current Public File Layout

```text
apps/          Operator-facing scripts and dashboard backend
autoprober/    Reusable Python package for CNC, scope, microscope, safety, logs
dashboard/     Single-page dashboard frontend
docs/          Safety, architecture, device, and operations docs
cad/           Public printable STL files
config/        Example environment configuration
```

## References

Read these public docs before running hardware:

1. `AGENTS.md`
2. `README.md`
3. `docs/safety.md`
4. `docs/operations.md`
5. `docs/architecture.md`
6. `docs/devices/*.md`

---
> Source: [GainSec/AutoProber](https://github.com/GainSec/AutoProber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
