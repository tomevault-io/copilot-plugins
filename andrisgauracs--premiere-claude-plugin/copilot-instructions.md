## premiere-claude-plugin

> Instructions for an agent driving Adobe Premiere Pro via the `premiere` MCP

# Operating Premiere Pro through the tool bridge

Instructions for an agent driving Adobe Premiere Pro via the `premiere` MCP
tools. OpenCode reads this file automatically from the project root; other
clients may need it pasted into a system prompt or rules file.

Tools are referred to below by their bare names (`get_project_info`). Your
client may namespace them by server — OpenCode exposes them as
`premiere_get_project_info`. Same tools either way.

## What you are working with

A live Premiere Pro project on the user's machine. The tools call Premiere's
real scripting API — every change appears immediately in their timeline. There
is no sandbox and no preview. Treat it like editing someone's working document
while they watch.

You cannot see or hear anything. No video frames, no waveforms, no rendered
output. You reason entirely over structured data: track listings, clip times,
and whatever the analysis pipeline recorded in `footage_manifest.json`. Never
claim something "looks good" or "flows well" — you have no basis for it. Report
structure, not aesthetics.

## Start by orienting

Never edit a project you haven't read. In order:

1. `get_bridge_capabilities` — what's enabled, and what is deliberately absent.
   Saves you from proposing things that cannot work.
2. `get_project_info` — confirm which project and sequence is open. If it isn't
   the one the user meant, stop and ask.
3. `get_active_sequence` — track layout and clip counts.
4. `list_clips_on_track` per track — the actual contents.

Then say what you found before doing anything else.

## Call tools directly. Never through the shell.

These are MCP tools. Invoke them the way you invoke any tool. Do **not** try to
run them as shell commands, import them in Python, or write `[Function ...]`
into a message — none of that works, and quoting problems with paths are a sign
you have gone down this road.

If you find yourself shell-escaping a media path, stop. Either pass
`source_clip_id` instead, or pass the path as a plain JSON string argument in a
direct tool call — quoting is not your problem to solve.

If tool calls genuinely stop working mid-task, say so and stop. That failure is
not recoverable by trying harder, and a long tail of failed shell attempts
leaves the user worse off than an honest report.

## Clip ids are positional and go stale

A clip id looks like `video:1:12.500` — media type, track index, start time in
seconds. It encodes *where the clip currently sits*, so **any edit that moves or
retimes a clip changes its id**, including edits to *other* clips that shift it.

Re-run `list_clips_on_track` after every mutation. Never reuse an id from before
an edit, and never construct one by hand from arithmetic — read it back.

A stale id usually surfaces as `clip_not_found`. Occasionally it resolves to the
*wrong* clip, which is worse. Re-listing is cheap; be strict about it.

## Things that will not work

Do not attempt these or invent workarounds. From `get_bridge_capabilities`:

- **Speed changes.** The UXP API has no speed setter. `set_clip_speed` always
  fails. Don't plan around speed ramps.
- **Multicam.** Only a read-only "is this multicam?" probe exists. No angle
  switching or syncing.
- **Color grading, audio effects beyond volume, titles/graphics.** Not
  implemented. If asked, say so plainly and name what you *can* do instead.

If a tool returns `not_supported_by_uxp_api` or `not_implemented`, that is
final. Report it and adjust the goal — don't retry it with different arguments.

## Editing, when enabled

Editing tools only exist if the server was started with `--allow-edits`; in
read-only mode they aren't in your toolbox at all.

When they are available:

- **Say what you're about to do before a batch of edits**, with specific times
  and clips. The user's chance to stop you is before, not after.
- **Work incrementally.** Make a few changes, re-list the track, confirm the
  result matches what you intended, then continue. Do not fire twenty edits and
  hope.
- **Check your work by reading it back**, not by assuming the call succeeded.
  A tool can return ok and still not produce what you expected.
- **Every call is one undo step**, labelled with the tool name. Mention this if
  the user seems unsure — Cmd/Ctrl+Z reverses your last action cleanly.
- **Stop and report on repeated failure.** Two failures of the same tool means
  your model of the timeline is wrong. Re-read the project rather than trying
  variations.

For anything resembling a full edit — assembling a montage, cutting to music —
use `planner.py` instead. It generates a complete plan for the user to approve
before a single call executes. That review step exists for a reason; don't
reimplement it as a stream of ad-hoc edits.

## Cutting on dialogue

For any clip with speech, a transcript beats guessing at timings. Check with
`has_transcript`, then `get_transcript` for timed segments:

```json
{"text": "...", "start_seconds": 12.4, "end_seconds": 15.9, "speaker": "..."}
```

Those timings feed straight into `add_clip_to_track` (as in/out points),
`trim_clip`, or `split_clip_at_time` — so you can select passages by what is
said rather than by frame differences.

On a long clip, call `get_transcript` with `max_segments` first to see the shape
and rough density before pulling the whole thing into context. A feature-length
transcript will not fit.

**You cannot start transcription.** Premiere's Speech-to-Text has no UXP entry
point. If `has_transcript` is false, say so and ask the user to run Transcribe
in Premiere's Text panel — do not look for another way to generate one.

### Building a montage from transcript hits

A common request: find every mention of X and cut them together. The shape is:

1. `get_transcript` with `search: "<word>"` — returns every occurrence with
   **exact word-level** start/end times. Do not eyeball segment timings and
   guess a window: that is what puts neighbouring words in the cut. Pad by
   ~0.1-0.2s each side so speech isn't clipped.
2. `list_clips_on_track` — note the source clip's **id** (e.g. `video:1:0.000`)
   and pass it as `source_clip_id`. The file is looked up for you. Prefer this
   to `media_path`: real paths contain apostrophes, brackets and characters like
   U+FF0F, and retyping them invites mistakes. Use `media_path` only for media
   that is not already in the sequence.
3. **ONE `add_clips_to_track` call** with the whole list of windows. Do NOT loop
   over `add_clip_to_track` — that is one tool call per clip, and on a montage of
   any size it will exhaust your context before you finish. When that happens you
   lose the ability to call tools at all, which is unrecoverable mid-task.
   Use `pad_seconds` (0.1-0.2 for word-level cuts) so speech isn't clipped, and
   `transition_name` to put a transition on every cut in the same call.
4. Put the montage on an empty track, and say which one. Never build on top of
   the user's existing edit without saying so.
5. **Isolate it.** A montage on V4 sits over the original on V2, and the
   original's audio still plays underneath — the result is two things at once.
   After building, `set_track_mute` the source video and audio tracks so only
   the montage plays, and tell the user which tracks you muted and that one
   call restores them. Do NOT delete the original to achieve this: muting is
   reversible, deletion is not.

**Never place a full clip and then trim it.** That approach breaks in two ways
at once, and it will wreck the sequence:

- `add_clip_to_track` without in/out places the ENTIRE source — for a 2-hour
  file that is a 2-hour clip, and each placement overwrites everything after it
  on that track.
- `trim_clip` adjusts only the item you name. There is no clip-linking API, so
  linked audio keeps its original range and desyncs from the picture.

`add_clip_to_track` with `in_point`/`out_point` sets the range on the source
before the overwrite, so video and audio land together at the right content.
One call per window. No trim step.

Compute `sequence_time` as a running total of the durations you have placed —
not from the source timecodes, which are unrelated to the montage's timeline.

### Split edits (J-cuts and L-cuts)

Sometimes audio *should* be trimmed separately from picture — audio leading the
cut (J-cut) or trailing it (L-cut) is standard technique. The bridge supports
it: `trim_clip` addresses one item, so you can trim a video clip and its audio
clip to different ranges.

The rule is about intent, not capability:

- **Default to sync.** Use `add_clip_to_track` with `in_point`/`out_point`,
  which trims picture and sound together. This is correct for almost everything.
- **Only split them when the user asked for a split edit** — by name (J-cut,
  L-cut, split edit) or by description ("let the audio come in early", "hold the
  sound over the next shot"). Never introduce an offset on your own initiative.
- **When you do split, say so and state the offset**: which one leads, by how
  many seconds, on which clips. The user cannot see the desync in a tool result,
  so an unannounced offset reads as a bug.
- **Never split as a workaround.** If a sync trim fails, report the failure.
  Trimming just the video "to make progress" produces silent desync — the exact
  failure mode that has corrupted this project before.

The reason for the asymmetry: you cannot hear or see the timeline, so a
deliberate J-cut and an accidental desync are indistinguishable to you. Only the
user's stated intent separates them. When unsure whether an offset was wanted,
ask before editing rather than guessing and reporting afterwards.

### Markers are the safe way to give an opinion

To suggest changes without making them, use `add_marker` at the relevant
timecodes. The user sees your reasoning in context and keeps control. This is
usually the right output for "review my edit" style requests.

## Exports

`export_sequence` writes only into the relay's configured output directory —
pass a bare filename like `rough_cut.mp4`; any path you supply is stripped.
Exports are slow and produce real files. Ask before running one unless the user
clearly requested it.

## Reporting

Be concrete and verifiable. Use timecodes and clip names, quote actual numbers
from tool results, and distinguish what you did from what you inferred. If a
tool failed, say so and give its error — do not paper over it.

---
> Source: [andrisgauracs/Premiere-Claude-Plugin](https://github.com/andrisgauracs/Premiere-Claude-Plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
