## innate-os

> Project instructions for Claude. See [AGENTS.md](AGENTS.md) for the system overview, the

# CLAUDE.md

Project instructions for Claude. See [AGENTS.md](AGENTS.md) for the system overview, the
`innate` CLI, and the ROS package map.

## Comments: minimal, why-only

Write code clear enough to need no comment; a comment is a last resort, not a habit.

A comment earns its place only when it states something the code *cannot* say — and then it
is 1-3 lines, no more:

- a physical/hardware fact (`# preload beyond this overcurrent-trips the servo`)
- a crash or race being designed around (`# destroying a sub under the spinning executor → InvalidHandle`)
- a deliberate exception to a project rule (`# committed: teardown must not be cancellable`)
- a misleading external interface (`# /ik_delta is an ABSOLUTE pose despite the name`)

Never write comments that narrate the next line, restate the function name, label obvious
sections of a short function, justify the change to a reviewer, or repeat something already
said once elsewhere (say the invariant in one place and reference it). Docstrings follow the
same rule: one tight paragraph of contract — no Args/Returns lists that restate
self-explanatory parameter names, no prose repeating the signature. If a comment is needed to
make code understandable, first try renaming or restructuring so it isn't.

## Code style

Write code that reads like prose — structure carries the meaning; comments are a last resort
(the section above is the full comments rule). The best part is no part: prefer deleting or
reusing over adding.

### Shape

- **Early returns, minimal indentation.** Guard clauses over nested ifs; merge related
  conditions with `and`/`or`. A multi-clause condition worth explaining becomes a small named
  predicate method (`_abandon(turn)`), not a comment.
- **One function, one altitude.** Split do-everything functions into named steps so the caller
  reads as a sentence — e.g. the agent loop: `_turn` owns only the error boundary, `_think` is
  the straight-line look → generate → commit → act, `_generate` owns the one blocking call.
- **One concern per module.** When a file grows a second job, split it (gemini.py became
  transport.py / tools.py / context.py). Pure functions — no ROS, no I/O, no self-state — live
  in a `utils.py` beside their caller.

### Types

- Every signature fully typed. Collaborator classes are imported under `if TYPE_CHECKING:` —
  annotations must add zero runtime import edges.
- Pick the representation by where the data lives:
  - **Frozen `@dataclass`** for process-internal domain objects (`Event`, `RunningSkill`).
  - **`TypedDict`** only when the dict *is* the wire format — parsed from JSON, republished as
    JSON, or handed to workspace user code (`SkillMeta`). Foreign API bodies stay plain dicts.
  - **StrEnum** (`brain_client/common/enums.py` backport; py3.10 has no `enum.StrEnum`) for
    closed string vocabularies: event kinds, trace names, chat senders, backends. The values
    are wire-visible — never change them. Keys of external APIs stay string literals.
- Name the shapes you pass around (`Transport = Callable[[str, dict], Iterator[dict]]`,
  `Frame = tuple[FrameLabel, bytes]`); reuse an existing alias (`Pose`) instead of redefining it.
- `basedpyright` stays at **0 errors** on `brain_client`. Fix at the root — type the state
  field, thread a value through instead of re-reading a nullable attribute, make the base class
  generic, restructure so the narrow is visible — never `# type: ignore`. A `# noqa` needs its
  reason on the same line.

### try/except

- A `try` is an invariant at a boundary, not insurance. Legitimate boundaries: a long-lived
  loop's crash reporter, ROS executor callbacks, user code (skills, agents, input devices),
  import machinery, and `finally` for cleanup/flags.
- Catch narrow types for parses (`json.JSONDecodeError`, `KeyError`, `ValueError`). Never wrap
  code that cannot raise; guard only the statement that can.
- In a loop over independent items, catch per item — one bad device/skill/agent must not stop
  the rest.
- Broad `except Exception` carries its justification: `# noqa: BLE001 — one bad agent must not
  stop the roster`.

### Before / after

The rules above, compressed into one transformation (dict-shaped state, nested ifs,
narrating comments, blanket try — versus typed, flat, silent):

```python
# ❌ Before
def _report_result(self, result, running):
    # handle the result of the skill
    try:
        if running is not None:
            # check that the ids match
            if running.get("skill_id") == result.skill_type:
                if result.success:
                    output = result.message
                    # send the output to the chat
                    if output is not None and output.strip() != "":
                        self._chat.emit("skill_output", output, speak=False)
                    self.on_event("completed", running.get("primitive_name", "unknown"))
                else:
                    self.on_event("failed", running.get("primitive_name", "unknown"), result.message)
    except Exception as e:
        self._logger.error(f"Error reporting result: {e}")
```

```python
# ✅ After
def _report_result(self, result: ExecuteSkill.Result, running: RunningSkill | None) -> None:
    if running is None or running.skill_id != result.skill_type:
        return  # a stale result must not report against a newer run
    if not result.success:
        self.on_event("failed", running.primitive_name, result.message)
        return
    if result.message.strip():
        self._chat.emit(Sender.SKILL_OUTPUT, result.message, speak=False)
    self.on_event("completed", running.primitive_name)
```

What changed, rule by rule: the signature is fully typed, so `running.get("skill_id")`
becomes `running.skill_id` and the `"unknown"` fallback vanishes — the type guarantees the
field. Four nesting levels become guard clauses with merged conditions. The magic string
`"skill_output"` becomes `Sender.SKILL_OUTPUT`. Every narrating comment is deleted; the one
that survives states an invariant the code can't show. The blanket `try` is gone — nothing
here can raise, and the ROS callback boundary that *calls* this already owns crash reporting.

### Tests are a development tool, not a deliverable

Write tests freely *while* developing — they are the fastest way to prove a change does what
you think. Then **delete the ones you wrote before pushing the PR**. A new test file ships
only when the user asked for tests.

The existing suite is different: keep it passing, and update a test the change legitimately
invalidates. Never delete someone else's test to make a change look clean.

### Done means verified

Before reporting work complete, all three must be clean:

```bash
ruff check ros2_ws/src/brain/brain_client/ && ruff format --check ros2_ws/src/brain/brain_client/
```

```bash
cd ros2_ws/src/brain/brain_client && PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python3 -m pytest test/ -q --ignore=test/test_node_boot.launch.py
```

```bash
cd ros2_ws/src/brain/brain_client && basedpyright brain_client/
```

## Nodes: the fleet is the budget

**Before adding a ROS node, look for somewhere it could live instead.** We are past 20 and
every one of them costs RAM whether or not it is doing anything — a process, an executor, a
DDS participant, its own discovery traffic. On a Jetson that budget is real and shared with
the models.

The pull toward more nodes is that it *looks* like clean design: one concern, one process,
crisp boundaries. That instinct is right on a server and wrong on an embedded system, where
composability sometimes has to be sacrificed for performance. (The Quest went from hundreds
of microservices to about four while optimizing.)

So: prefer a new function, class, or timer in a node that already owns the data. Reach for a
new node when it genuinely needs its own lifecycle — separate crash domain, different rate,
hardware it must own exclusively — and when it does, put it in an existing composable
container so it shares a process rather than starting another one.

## Writing skills

### Never `time.sleep` — always `self.sleep`

**In skill code, use `self.sleep(seconds)`. Never `time.sleep(seconds)`.**

`self.sleep` wakes and raises `SkillCancelled` the moment a Stop lands; `time.sleep` blocks
to completion, so a skill that uses it keeps running (and keeps the robot moving) after the
user pressed Stop. Sleeping is the only cancel point a loop needs — write the loop as if
cancel didn't exist and let the framework halt the base and report `CANCELLED`.

```python
while traveled < target:
    self.mobility.send_cmd_vel(linear_x=velocity, duration=0.5)
    self.sleep(0.1)          # ✅ cancellable
    # time.sleep(0.1)        # ❌ Stop is ignored until the sleep finishes
```

`time` itself is fine for *measuring* — `time.time()` / `time.monotonic()` for deadlines and
elapsed checks. The rule is only about blocking.

`self.wait_for(read, timeout)` and `self.check_cancelled()` are cancel-aware too; cleanup
belongs in `try/finally`.

**The one exception:** teardown and already-committed physical actions must *not* be
cancellable, so they use `time.sleep` deliberately — e.g. once `pick_any_object` closes the
gripper, a cancel must not unwind mid-grip and drop the object. If you write such a section,
comment it, or the next reader will "fix" it back to `self.sleep` and reintroduce the bug.

See [AGENTS.md](AGENTS.md#writing-skills) for the full cancellation contract.
@AGENTS.md

---
> Source: [innate-inc/innate-os](https://github.com/innate-inc/innate-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
