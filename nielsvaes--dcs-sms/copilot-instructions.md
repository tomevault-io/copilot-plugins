## dcs-sms

> This is the orientation document for AI agents (and humans) working in this repo. It exists so an agent dropped in cold can write idiomatic dcs-sms code without grepping every module first.

# AGENTS.md — dcs-sms framework reference for AI agents

This is the orientation document for AI agents (and humans) working in this repo. It exists so an agent dropped in cold can write idiomatic dcs-sms code without grepping every module first.

It is deliberately a **rules-and-conventions** document, not an API reference. Every public `sms.*` symbol is documented per-module with worked examples in [`docs/api/`](docs/api/) — that is the canonical reference. Read the relevant `docs/api/<module>.md` page before you write code that touches that module.

> **Companion documents:**
> - [`docs/api/`](docs/api/) — per-module reference: signatures, options tables, runnable examples, see-also.
> - [`docs/superpowers/specs/`](docs/superpowers/specs/) — per-module design docs (canonical "why is it shaped this way").
> - This file is a *summary*. When the spec disagrees with this file, the spec wins.

---

## 1. The prime directive

**Always prefer `sms.*` over vanilla DCS API when the framework provides it.**

The framework exists precisely because vanilla DCS scripting is awkward, undocumented, and full of footguns. If a vanilla call would replace a single line of `sms.*` code, use the `sms.*` version every time.

**When the framework does not yet cover what you need:**

1. Use the vanilla DCS API (`Group.getByName`, `coalition.addGroup`, `world.addEventHandler`, `trigger.action.*`, etc.) to get the job done.
2. **Surface this to the user.** Say something like:
   > "I needed to call vanilla DCS `<api>` here because `sms.<module>` doesn't expose this yet. Want me to file a GitHub issue so we can fold this into the framework later?"
3. If the user agrees, invoke the `make-issue` skill (or use `gh issue create`) with a description of the gap, the use case, the vanilla API used, and a sketch of what the `sms.*` shape might look like.

This is non-negotiable. The framework only grows by noticing where it falls short. **If you silently fall back to vanilla without flagging the gap, you have failed the assignment.**

### What counts as "the framework doesn't cover it"

- The function does not exist on any `sms.*` module.
- The function exists but its current contract doesn't expose the data you need (e.g. `sms.unit.get_position` returns vec3 but you need the full Position3 with orientation axes).
- The function exists but a known issue (`docs/superpowers/specs/`, GitHub issues) limits it.

### What does NOT count

- "There's a vanilla one-liner that's slightly shorter." Use `sms.*` anyway — the framework's value is the *consistency* and the failure model, not the syntactic sugar.
- "The framework version returns nil on bad input but I want it to throw." This is the framework's [failure model](#3-failure-model-log--nil-never-throw) by design. Never bypass it.

---

## 2. Repo layout at a glance

- **`framework/`** — in-DCS Lua framework. Runs inside the mission environment. One file per `sms.*` module, plus `load_all.lua` (one-shot loader) and `test/` (bash smoke tests driven by `tools/dcs-sms.exe`).
- **`tools/`** — host-side Go. Produces the `dcs-sms` CLI and the embedded `Scripts/Hooks` Lua hook. Not loaded into DCS missions; talks to them via a filesystem mailbox.
- **`docs/api/`** — per-module reference pages with worked examples. Canonical source for "how do I call this".
- **`docs/superpowers/specs/`** — design docs, one per sub-project / module. Canonical source for "why is it shaped this way".

**Two distinct Lua environments:**

- **Mission environment** (`framework/*.lua`): sandboxed by `Scripts/MissionScripting.lua`. `os`, `io`, `lfs` are nilled. Lua 5.1. `print` is silent — use `env.info` / `env.error` (or `sms.log.*`).
- **Hook environment** (`tools/lua/dcs-sms-hook.lua`): NOT sandboxed. Has LuaSocket, `lfs`, full file I/O. Used by the host-side bridge.

The framework runs in the mission environment. The bridge runs in the hook environment. They communicate through the filesystem mailbox the bridge installs.

---

## 3. Failure model: log + nil, never throw

**Every public framework call follows this contract:**

- On bad input or missing entity → log via the module's tagged logger and return `nil` or `false`.
- **Never `error()` out of an `sms.*` call.** Throwing aborts the entire mission script — that is the failure mode dcs-sms exists to avoid.
- Methods accept either a handle (`{name=...}` table) or a raw name string interchangeably. They also tolerate garbage (nil, numbers, booleans) — those normalize to "not alive" and produce a logged nil.

```lua
-- All four of these behave the same way: log + return nil if the unit isn't there.
sms.unit("ghost"):get_position()
sms.unit.get_position("ghost")
sms.unit.get_position({name = "ghost"})
sms.unit.get_position(nil)
```

### Log levels: warn for caller misuse, error for real failures

`sms.log` exposes four levels — `debug`, `info`, `warn`, `error` — each landing on the corresponding `env.*` sink (`env.info`, `env.info`, `env.warning`, `env.error`). Pick the right one when authoring framework code:

- **`log.warn`** — caller misuse. The API user passed garbage, named a non-existent entity, called an air-only verb on a ground group, requested an unknown enum, called against a destroyed handle, etc. The caller can fix it by changing what they pass or what state they call against. Most framework rejection paths are warns.
- **`log.error`** — real failure that the caller couldn't have prevented. DCS rejected something the framework expected to succeed (`coalition.addGroup` raised, `addStaticObject` accepted but not findable post-call), DCS returned a value outside our known enum, an internal invariant was violated, or `pcall` caught a user callback raising during dispatch.

When in doubt: if the message could plausibly cause the user to think *"oh, I called this wrong"*, it's a `warn`. If it points at framework or DCS misbehavior, it's an `error`.

The runtime threshold defaults to `info` (everything visible). Production missions can mute caller-misuse warns with `sms.log.set_level("error")`.

Smoke tests deliberately exercise warn paths to verify rejection. Seeing `[sms.*]` WARNING lines in `dcs.log` during a smoke run is *evidence the test passed*, not a failure.

When you write new framework code, mimic this contract. Do not `error()`. Do not `assert()` on user input. Use `pcall` around any vanilla DCS call that could throw.

### Category enforcement: air-only / ground-only / naval-only / ROE

Four private flags on a payload (task / command / option) mark category restrictions:

- `_sms_air_only = true` — only `airplane` / `helicopter` groups accept this payload.
- `_sms_ground_only = true` — only `ground` groups accept (ships and trains are excluded).
- `_sms_naval_only = true` — only `ship` groups accept. (No v1 builder sets this; reserved for forward-compat.)
- `_sms_roe = true` — special marker on ROE options. Apply layer reads the group's category and dispatches to `AI.Option.{Air,Ground,Naval}.id.ROE` with category-specific value validation. Rejects values not allowed for the resolved category (e.g. `"weapon_free"` against ground groups).

`set_task` / `push_task` / `set_command` / `set_option` reject mismatches at apply time with `log.warn + return false`. `combo` (sms.task only) aggregates: a combo containing any air-only or ground-only sub-task inherits the corresponding flag.

---

## 4. Conventions and units

| Concept | Convention |
|---|---|
| **Coordinates** | `vec3 = {x = north, y = altitude, z = east}` (DCS-native). DCS-2D uses `{x = north, y = east}` (no altitude). Conversion: 2D `y` ↔ 3D `z` (both are east). Verified by spawning a unit and observing F10 movement. |
| **Headings** | **Public API: degrees**, 0=north, 90=east, clockwise. Internal: radians (DCS native). Use `sms.utils.deg_to_rad` / `rad_to_deg` to cross the boundary. |
| **Altitudes** | **Public API: meters** (DCS native). Pilot-facing helpers: `sms.utils.feet_to_meters` / `meters_to_feet`. |
| **Coalition strings** | DCS wire format is lowercase `"red"` / `"blue"` / `"neutral"` — exposed as `sms.K.coalition.RED` / `BLUE` / `NEUTRAL` for authoring. (DCS internally uses `0/1/2` — never expose these.) |
| **Categories** | DCS wire format is lowercase `"airplane"` / `"helicopter"` / `"ground"` / `"ship"` / `"train"` — exposed as `sms.K.category.AIRPLANE` / `HELICOPTER` / `GROUND` / `SHIP` / `TRAIN`. |
| **Naming** | snake_case for everything public (`get_position`, `is_alive`, `from_drawing`). Internal helpers: `_leading_underscore`. |
| **Auto-suffix on collision** | Spawning with a name already taken yields `name-1`, `name-2`, ... Always trust the returned handle's `:get_name()` over the input string. |
| **Log tags** | Each module logs as `[sms.<module>]` via `sms.log.module("sms.<module>")`. Top-level untagged calls log as `[sms]`. |

### Style: split multi-line builder calls

When a `sms.task.*` / `sms.commands.*` / `sms.options.*` builder spans multiple lines, bind it to a local first and apply on a separate line. Do not nest the builder directly inside `set_task` / `push_task` / `set_command` / `set_option`:

```lua
-- Preferred — easy to scan, the trailing `})` is one place not two.
local move_task = sms.task.move_to({x = 12000, y = 4500, z = -3500}, {
  speed = 180,
})
cas:set_task(move_task)

-- Avoid for multi-line builders — the closing `}))` is hard to track.
cas:set_task(sms.task.move_to({x = 12000, y = 4500, z = -3500}, {
  speed = 180,
}))
```

Single-line builders are fine inline (`cap:set_option(sms.options.rtb_on_bingo(true))`). The split is for readability of the multi-line case; don't add a named indirection for a trivial value. Conventional names: `local <verb>_task` for tasks, `local plan` for `sms.task.combo`, `local opt` / `local cmd` for options / commands.

This style applies to docs, examples, and smoke tests.

### Style: descriptive variable names over short ones

In docs, examples, and smoke tests, prefer role- or content-based names over single-letter abbreviations. Mission scripts read more like natural language when the variables describe what they hold:

```lua
-- Preferred
local convoy   = sms.group("Red Armor 1")
local bandit   = sms.unit("Bandit-1")
local target   = convoy:get_position()
local bearing  = sms.utils.bearing(bandit:get_position(), target)

-- Avoid in docs/examples
local g = sms.group("Red Armor 1")
local u = sms.unit("Bandit-1")
local p = g:get_position()
local brg = sms.utils.bearing(u:get_position(), p)
```

Exceptions where short names stay idiomatic:
- Single-letter loop indices (`i`, `k`, `v`).
- Conventional aviation / military shorthand used as a role: `cas`, `cap`, `awacs`, `lz`, `fob`, `sa6`, `mlrs`.
- Short event-payload binders inside handlers: `evt`, `id`.

---

## 5. Entity handles — the universal pattern

Every entity wrapper (`sms.group`, `sms.unit`, `sms.static`, `sms.area`) follows the same shape:

```lua
local handle = sms.unit("Bandit-1")    -- callable lookup; returns handle | nil + log

-- Method-style and module-style both work:
handle:get_position()
sms.unit.get_position(handle)
sms.unit.get_position("Bandit-1")      -- bare name string also works
```

A handle is a small `{name = "..."}` table with a metatable whose `__index` points at the module. They are cheap; build and discard them freely. They do not cache — every method call re-resolves through DCS, so handles stay correct after the underlying entity dies.

`sms.area` handles also carry a `kind` field (`"circle"` or `"polygon"`) and an internal `_data` table. `sms.timer` handles and `sms.events` connection handles use a different pattern (private metatables, identity-checked) but expose the same `handle:method()` ergonomics.

---

## 6. Loading order

The bridge currently loads framework files via `net.dostring_in` in this order:

```
sms.lua → log.lua → utils.lua → constants.lua (which dofiles every framework/constants/*.lua) → group.lua → unit.lua → area.lua → timer.lua → rule.lua → group_spawn.lua → static.lua → events.lua → weapon.lua → task.lua → commands.lua → options.lua
```

Each module asserts the dependencies it actually uses. When adding a new module, decide where it slots in based on what it needs and append the assert.

**One symbol, one home.** Every `sms.<module>.<symbol>` is defined in `<module>.lua` (or, for `sms.group`, also in `group_spawn.lua`, which is a clearly-named continuation file for the spawn factories). No module writes into another module's namespace. When you add behavior that semantically extends another module (e.g. an `sms.group` method that needs `sms.events` internals), the public symbol goes in the owning module's file; cross-module data needed at call time is exposed as a private `sms.X._name` field.

Cross-module call-time references resolve fine even when the dependent module loads later: `sms.group.connect` references `sms.events.*`, and `sms.group.set_task` references `sms.timer.*`. Both files load before events / timer in the chain — that's safe because the references are inside function bodies and only resolve at call time. Users must finish loading the whole framework (e.g. via `load_all.lua`) before invoking those methods.

For one-shot (re)loading of the whole framework in a mission, use [`framework/load_all.lua`](framework/load_all.lua), which `dofile`s every module in the order above.

---

## 7. Module index

**Per-symbol reference with worked examples lives in [`docs/api/`](docs/api/) — one page per module.** This section is a one-line index; click through to the module page for signatures, options tables, examples, and see-also links.

| Module | File(s) | Reference | Purpose |
|---|---|---|---|
| `sms` (root) | `sms.lua` | — | Single global namespace; idempotent on reload. Internal handle factories. |
| `sms.log` | `log.lua` | [`docs/api/log.md`](docs/api/log.md) | Tagged logger; four levels with runtime threshold. |
| `sms.utils` | `utils.lua` | [`docs/api/utils.md`](docs/api/utils.md) | Cross-cutting helpers: unit conversions, vec3 maths, coalition/country lookup. |
| `sms.constants` (alias `sms.K`) | `constants.lua` + `constants/*.lua` | [`docs/api/constants.md`](docs/api/constants.md) | Single namespace for every wire-format constant: `sms.K.coalition`, `sms.K.category`, `sms.K.countries`, `sms.K.skill`, `sms.K.alt_type`, `sms.K.waypoint.type` / `.action`, `sms.K.targets`, `sms.K.designations`, `sms.K.roe` / `alarm_state` / `formation` / `reaction_on_threat` / `radar_using` / `flare_using`, plus the auto-generated `sms.K.units` and `sms.K.statics` catalogs (`origin_of` helpers preserved). |
| `sms.group` | `group.lua` (+ `group_spawn.lua`, `events.lua`) | [`docs/api/group.md`](docs/api/group.md) | Group entity wrapper; `create` / `clone` factories; `:connect` event sugar; apply API for tasks / commands / options. |
| `sms.unit` | `unit.lua` (+ `events.lua`) | [`docs/api/unit.md`](docs/api/unit.md) | Unit entity wrapper; `:connect` event sugar. |
| `sms.area` | `area.lua` | [`docs/api/area.md`](docs/api/area.md) | Unified circle/polygon abstraction; ME zones, drawings, runtime construction. |
| `sms.timer` | `timer.lua` | [`docs/api/timer.md`](docs/api/timer.md) | Sim-time scheduling: `after` / `every` / `now`. |
| `sms.rule` | `rule.lua` | [`docs/api/rule.md`](docs/api/rule.md) | Declarative trigger rules; `once` / `continuous` / `toggle` lifecycle with orthogonal cooldown + sustain; per-rule timer; dev_condition bypass for instant testing. |
| `sms.static` | `static.lua` | [`docs/api/static.md`](docs/api/static.md) | Static-object wrapper; `create` / `clone` factories. |
| `sms.events` | `events.lua` | [`docs/api/events.md`](docs/api/events.md) | Pub/sub bus over DCS world events; entity-scoped sugar; user-emittable signals. |
| `sms.weapon` | `weapon.lua` | [`docs/api/weapon.md`](docs/api/weapon.md) | Weapon-object wrapper from SHOT/HIT events; tracking lifecycle; impact reports. |
| `sms.task` | `task.lua` | [`docs/api/task.md`](docs/api/task.md) | Task-table builders (`move_to`, `attack`, `orbit`, `combo`, ...) + `group:set_task` / `:push_task` apply. |
| `sms.commands` | `commands.lua` | [`docs/api/commands.md`](docs/api/commands.md) | One-shot controller commands + `group:set_command` apply. |
| `sms.options` | `options.lua` | [`docs/api/options.md`](docs/api/options.md) | Persistent controller options (ROE, formation, alarm state, ...) + `group:set_option` apply. |
| `sms.prefab` | `prefab.lua` (+ `prefab_distill.lua`) | [`docs/api/prefab.md`](docs/api/prefab.md) | Portable bundles of groups + statics + zones + drawings. Distill ME selection dumps; load/save prefab files; spawn at any anchor + rotation + (optional) country override; per-instance lifecycle. |

For end-to-end recipes that combine multiple modules, see [`docs/api/examples.md`](docs/api/examples.md).

---

## 8. Flagging gaps in the framework

When you need behavior the framework doesn't cover, this is the workflow:

1. **Get the user's task done first.** Use vanilla DCS API. The user wants their feature; they don't want to wait for framework work.

2. **In your reply, name the gap explicitly.** Use this phrasing pattern:

   > "I used vanilla DCS `<api>` for `<purpose>` because `sms.<module>` doesn't currently expose this. The shape it would need is something like `sms.<module>.<proposed_name>(<args>) → <return>`. Want me to file a GitHub issue so we can fold this into the framework?"

3. **If the user agrees, file the issue.** Prefer the `make-issue` skill — it'll rewrite the description for clarity and link to the relevant code. If unavailable, use `gh issue create` directly.

   The issue should include:
   - **Use case:** what the user was trying to do (one or two sentences).
   - **Vanilla API used:** the actual DCS calls you fell back to.
   - **Proposed `sms.*` shape:** function signature, return value, failure mode.
   - **Notes:** anything tricky about the underlying DCS behavior (silent drops, namespace quirks, mid-frame deconstruction risk, etc.).

4. **Do not silently work around the framework.** A vanilla call without a flagged gap is a bug-shaped omission — it deprives the project of the signal that the framework is missing something.

### What does NOT need a gap-flag

- Reading `env.mission.*` for one-off mission-descriptor introspection (used in spawn.clone / static.clone). The mission descriptor is read-only metadata; wrapping it module-by-module would balloon scope.
- Calling `world.event.S_EVENT_*` numeric constants directly. The framework already mirrors them as `sms.events.*` — use the mirrored form, but the underlying `world.event` table is fine to inspect for completeness.
- Using `pcall` defensively around any vanilla call inside framework code itself. That is the framework code; it's expected to touch vanilla.

---

## 9. When you write new framework code

- Read the relevant spec in [`docs/superpowers/specs/`](docs/superpowers/specs/) before changing a module. That document explains *why* the shape is what it is.
- Mirror the existing patterns: callable handles, `_name_of` normalization, `is_alive` gates on every method that touches DCS state, log + nil + never throw.
- Add a tagged logger at the top of the file: `local log = sms.log.module("sms.<name>")`.
- If you depend on another `sms.*` module, `assert(type(sms.<dep>) == "table", ...)` at the top of the file. State the load order in the file's top comment.
- Update [`AGENTS.md`](AGENTS.md) (this file) as part of the same PR. Adding or removing public surface without updating the §7 module index is a regression — agents and humans both lose visibility.
- **Update the per-module reference page at [`docs/api/<module>.md`](docs/api/) in the same PR.** Add or revise the entry for every public symbol you touch — signature, options table, return value, runnable example. A PR that ships new public surface without a corresponding `docs/api/` entry is incomplete. This rule is parallel to the `AGENTS.md` sync rule and applies at every stage (spec → plan → implementation → review).
- If your change introduces a cross-module pattern worth showcasing, add or update a recipe in [`docs/api/examples.md`](docs/api/) too.
- Add or extend a smoke test under `framework/test/` (`smoke_<module>.ps1` driven by `tools/dcs-sms.exe`; shared helpers live in `framework/test/_smoke.psm1`).

---

## 10. Out-of-DCS tooling (`tools/`)

The `tools/` directory is host-side Go. It produces `dcs-sms` / `dcs-sms.exe`, a CLI that:

- `install-hook` — drops `dcs-sms-hook.lua` into `<Saved Games>/DCS*/Scripts/Hooks/`.
- `install-me-mod` — installs the Mission Editor mod into `<DCS install>/MissionEditor/`. Patches `MissionEditor.lua` with sentinel-marker delimited `require('dcs_sms_me.init')`, copies the mod files to `MissionEditor/modules/dcs_sms_me/`. Backs up `MissionEditor.lua` first. Idempotent. Cache `--dcs-path` in `%AppData%\dcs-sms\config.toml` (or env `DCS_SMS_DCS_INSTALL`).
- `uninstall-me-mod` — reverses `install-me-mod`. Removes the patch block from `MissionEditor.lua` (surgically, by markers; falls back to backup-restore), deletes the modules dir, deletes the backup file.
- `status` — confirms the hook is alive and reports current mission name.
- `exec --code "<lua>"` — runs Lua in the running mission and returns structured JSON `{ok, return_value, output, error}`.
- `tail-log -n <N>` — last N lines of `dcs.log`.

Agents writing or testing framework code typically use `dcs-sms exec` to run snippets against a running mission. See [`tools/cmd/dcs-sms/README.md`](tools/cmd/dcs-sms/README.md) for installation and the required `Scripts/MissionScripting.lua` edit, and [`docs/release-gate/bridge-smoke.md`](docs/release-gate/bridge-smoke.md) for the manual smoke checklist run before each release.

This is separate from in-DCS framework work. Don't conflate the two environments.

---

## 11. Versioning and releases

Two parallel tracks, both semver `0.x.y`. Major (1.0) is reserved for the moment the public surface stops moving and the project commits to deprecation cycles for breaking changes — explicitly not on the v1 roadmap.

- **Framework** (`framework/`): tags `framework-v0.X.Y`, canonical version at `framework/sms.lua` → `sms.version`.
- **ME-mod** (`tools/me-mod/`): tags `me-mod-v0.X.Y`, canonical version at `tools/me-mod/lua/dcs_sms_me/version.lua`.
- **Prefab data format** (`PREFAB_VERSION` in `prefab_distill.lua`): independent of release tags; bumps when the on-disk prefab schema changes.

Bump rules under 0.x:

- **Patch** (`0.x.y` → `0.x.y+1`) — pure bug fix. No new public symbols, no behaviour change for callers that were working before.
- **Minor** (`0.x.y` → `0.x+1.0`) — anything else: new public function / module / UI feature, OR a breaking change to an existing one (allowed under 0.x).

Workflow:

1. **In-source first.** Bump the version string in the same commit that lands the change. The git tag is the announcement; the in-source string is the truth.
2. **Annotated tags only** (`git tag -a`), prefixed by component (`framework-vX.Y.Z`, `me-mod-vX.Y.Z`). The tag's annotation message is the release summary.
3. **Update `CHANGELOG.md`** in the same commit — section per component, newest entry on top.
4. **Push with `--follow-tags`** (or push the tag separately) so the tag reaches `origin`.

Specs and plans that touch public surface should call out the version bump as part of their scope, the same way they call out `AGENTS.md` §7 and `docs/api/` updates.

---
> Source: [nielsvaes/dcs-sms](https://github.com/nielsvaes/dcs-sms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
