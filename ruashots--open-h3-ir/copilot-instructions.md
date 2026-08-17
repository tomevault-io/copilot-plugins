## open-h3-ir

> **This document is for changing the compiler.** Read it before changing anything. It is the set of

# Working on OpenH3-IR

**This document is for changing the compiler.** Read it before changing anything. It is the set of
rules that are not preferences, a map of which file owns what, and an honest list of what is missing.
There is no install path here, on purpose.

Two neighbours, if one of them is the job instead:

- **Installing it and making it run:** [HANDOFF.md](HANDOFF.md).
- **Calling the service from an application:** [docs/calling-the-api.md](docs/calling-the-api.md).

The project is `open-h3-ir`; the import package and the command are both `h3ir`.

The checks to hold your work against, both reproducible with no model and no GPU: `h3ir controls` is
21/21 in under a tenth of a second, and `pytest -q` is green. The control count is a gate and should
only move when you deliberately add or remove a control. The test count is not pinned here, because it
moves every time anyone adds a test and a stale number reads as a regression.

## What this is and where it runs

A local rebuild of MiniMax H3's closed Context-IR stage. Brief in, validated H3 prompt plus the
asset wiring it is true for, out.

- **Assume the compiler and the GPU are on different machines.** No path or URL is hardcoded
  outside `config.py`, and ComfyUI is always reached over HTTP, never through the filesystem. Keep
  it that way. A filesystem shortcut works on a single box and fails silently everywhere else.
- Reasoning and vision run on whatever `H3IR_LLM_URL` points at. Nothing calls MiniMax.
- `h3ir doctor` tells you what is actually reachable before you debug anything else.

## The rules that are not preferences

1. **Never let a model decide structure.** Labels, label order, speaker IDs, cut times, retention
   markers, task-type prefixes and section order are computed in `plan.py` and emitted by
   `render.py`. If you find yourself adding a structural instruction to a prompt file, the fix
   belongs in the planner instead.
2. **The user's words never pass through a model.** Dialogue reaches the output through
   `{{D1}}` placeholder substitution in `render.py`. If you change that path, `D4` will catch you.
3. **Never trust the endpoint's structured output.** It is documented in `backend.py` with the
   measurements: `json_schema` is silently not applied while reasoning is on, and even with it off
   the grammar constrains shape but not completion or sense. Parse and re-check everything.
4. **Never degrade silently.** If the model is unreachable the service raises. A caller cannot
   tell a good IR from a bad one, so quietly producing a worse one is the failure nobody notices.
5. **Nothing ships on judgement.** See below.
6. **The deterministic draft is the product floor, not a degraded mode.** `draft.py` builds a
   complete valid IR with no prose model. The LLM pass is additive. Any validator error, leaked
   reasoning, or model outage falls back to the draft, so the caller always gets something valid.
   The draft failing its own validator is the one thing that raises, because it is deterministic
   and there would be nothing to fall back to. Do not turn this back into retry-until-valid.
7. **Thinking is per call, not global.** ON for the beat sheet (planning: measured +5.3pp), OFF
   for extraction, classification and prose (precision: measured −8.5pp). This is contingent on
   code owning every machine-checkable field. If you ever let the model emit a timecode, turn
   thinking off for that call.
8. **Never enable guided decoding without re-reading why it is off.** vLLM #39130 can skip grammar
   enforcement silently with a reasoning parser active; llama.cpp #20345 reports the converse on
   this model family. `H3IR_GUIDED_DECODING=1` exists for comparison, not for production.
9. **The model is deaf. Never ask it about audio.** `analyse_audio` makes no model call. Audio
   facts are typed metadata plus a real transcript. An invented timbre is worse than none because
   `<Audio N>` carries no content into the encoder, so the IR text is its only channel.
10. **Proportionality is part of the bar, and it is an explicit input.** "If I ask for simple, I want
    simple, if I say go crazy, I want crazy." `brief.creativity` is `restrained | balanced | bold`,
    default balanced, and it governs exactly one thing: whether the writer may add **content the
    request never supplied**: a spoken line, a score, on-screen text. It does NOT mean "more shots"
    or "more camera moves"; putting effort on this dial would be shot-count-as-a-rule one layer above
    the validator, where nothing catches it. Never infer the setting from the request. That was
    considered and rejected, because it would be wrong often and the maintainer could not overrule it. An
    explicit prohibition in the request outranks every position: `bold` on "No dialogue" licenses no
    dialogue. See `creativity.py` and design doc section 19.
11. **A rule asserts a decidable fact, never a preference.** The test: could a competent director
    disagree with it? If they could, it is not a check, at any severity. Shot count is not a
    defect. Prose quality is not a defect. Whether an edit is good is the maintainer's call and the
    validator has no access to it. Where a spec sentence constrains something discretionary, narrow
    the rule to its decidable residue and hold it at **WARN**, because **ERROR is what the fix loop
    sends back to the model** and a rule that can be argued with must never instruct a rewrite.
    Section 18 of the design doc records the audit that established this and every rule it changed.
    Before adding a rule, name its source: a spec line, a measured fact about the model, or a
    decidable property of the text. "It looked wrong to me" is not one of the three.

## Changing a prompt or a template

Prompt text lives in `h3ir/prompts/*.txt` as versioned files precisely so a change is an artifact you
can score. The loop is:

```bash
h3ir controls                                  # must be 21/21 before anything else
h3ir eval --label my-change --prose prose_shot.v3.txt
# read the gate; if SHIP-ABLE:
h3ir baseline --label my-change
```

This is not ceremony. It has already caught a change of mine that improved the metric I was
aiming at while introducing validator errors, and the root cause turned out to be a latent bug
rather than the prompt. Assume your next confident improvement is the same.

**Do not add a control exception to make a rule pass.** MiniMax's own published example is the
control; if a rule fires on it, the rule is wrong. That is how the 350-word floor and the closed
camera vocabulary became guidance rather than law, and later the shot-citation check, which
fired on their example because a persistent setting legitimately is not re-cited every shot.

**A metric reads the artifact that SHIPS, never an intermediate.** In the write-first path `doc.plan`
is the *deterministic draft's* plan (the model's prose never goes back into it), so anything reading
`plan.shots` is scoring an object that was thrown away, and it fails silently: the number is
plausible, nothing raises. Three fields were caught doing it in one evening (`restatement` reporting
1.00 on visibly different shots, `n_shots` reporting 4 against 0 timed cuts, and `_split_written`
dropping the whole description), plus a fourth in a script written *after* the other three were fixed.

**Second half of the same rule: measure it in the CONFIGURATION that ships.** A harness knob whose
default is anything other than production's turns every run into a truthful report about a pipeline
nobody uses. `RunConfig.compose_prompt` defaulted to an explicit composer name, overrode the mode
selection, and reported `clean_rate` 0.167 where the real figure is 1.000. Harder to catch than the
first half, because a wrong-configuration run is internally consistent.

**And read the artifact back; never trust the code that wrote it.** Four faults found this way,
including the provenance record that was written to say which pipeline produced a result and recorded
neither. The field was on the dataclass and missing from the serialiser, and the writing code read
perfectly.

**So: "the number moved" is not evidence until you can name the artifact that produced it.** When you
add a metric, name a second field it must agree with and check the pair. That is what caught all four,
and no test caught any of them. `n_shots` vs `n_timed_cuts` must satisfy `shots = cuts + 1` because T4
enforces it; `restatement` near 1.0 contradicts `shot_distinctness` near 1.0; `words = 0` contradicts
`errors = 0` because S9 and T1 would both fire. Design doc §24.

**Falsify every test you write. It is not ceremony. It is what distinguishes a test from a
comment.** Break the code the test covers, on purpose, and watch it go red. Two tests passed a
deliberate break in one evening, and the two failure modes are different, so know both:

- **A fixture that cannot discriminate.** The video frame-distinctness test used a clip generated from
  a `drawbox` expression that silently rendered nothing, so all three frames were identical 1604-byte
  images. The assertion compared them and passed. Fixed by using `testsrc`, whose frames must differ.
- **A cache short-circuiting the code under test.** Breaking `VIDEO_FRAME_FRACTIONS` to `(0.5,0.5,0.5)`
  left the test green because frames from an earlier correct run were already cached under that key.
  The cache answered, not the sampler. Fixed by keying the cache on the fractions AND giving the test
  its own key.

**A cache keyed on its inputs but not on the logic that transformed them will serve stale results
across a code change and look correct.** That is what `ANALYZER_VERSION` is for, and it has now been
needed three times (pose split, video frames, audio characterisation) plus once for frame fractions.
**If a compiled brief is ever cached, the prompt version belongs in the key**, because the compose prompts are
the transforming logic and they change more often than anything else here.

**A passing control is not proof of correctness.** The rule L5 arrived here as a false positive:
it flagged every standalone `<Picture N>` line, while the spec forbids them only when the label is
not separately analysed. It passed the official control the whole time. When you write a rule, also
write the input that must NOT trip it. `test_hardening.py` has four such cases for G2 alone,
because my first draft of that rule fired on "he gives an okay sign".

## Where things are

| file | what it owns |
|---|---|
| `config.py` | every host-specific value. Nothing else may hardcode one. |
| `grid.py` | the 17k+5 frame grid and all duration maths. `effective_seconds` vs `nominal_seconds` is a real distinction, so read the docstring. |
| `tokens.py` | exact token counts using H3's own vocab (vendored under `h3ir/data/`). |
| `models.py` | the contract. Every stage boundary is a dataclass here. |
| `backend.py` | the LLM client and the three silent endpoint failures. |
| `analyse.py` | AssetCards, cached on content hash. Audio needs a transcript, see below. |
| `mode.py` | which of the five modes, and how it fails safe. |
| `lora.py` | the registry, `howtouse.md` parsing, ingest-time trigger validation. |
| `plan.py` | all structure. The four solved problems live here. |
| `prose.py` | the only two places a model writes anything. |
| `render.py` | deterministic rendering. Must be byte-reproducible. |
| `validate.py` | the rules. Proved by `evalloop/controls.py` in both directions. |
| `compile.py` | the orchestrator and the stage order (with the reason for that order). |
| `service.py` | the HTTP surface and the three response layers. |
| `uploads.py` | the content-addressed store behind `PUT /v1/assets/{sha256}`: the digest is computed as the bytes arrive, the ceilings and the age limit come from `config.py`, and eviction is least-recently-used. Write-only by design, so every other method on an asset is a 405. |
| `comfy.py` | ComfyUI over HTTP; graph prompt substitution that refuses to guess. |
| `acceptance.py` | the five-arm comparison, built without touching the GPU. |

The ComfyUI pack in `comfyui/` is six Python files plus a `web/` folder of three JS files, and imports nothing from `h3ir`:

| file | what it owns |
|---|---|
| `h3ir_client.py` | the service protocol, the option lists, the report. No ComfyUI, no torch, no third-party packages. |
| `media.py` | tensors and mappings to files on disk, content-addressed. No ComfyUI at module scope. |
| `nodes.py` | the five schemas, the model loaders and the socket-to-file mapping. This is the only file that needs a canvas. |

## ComfyUI frontend mechanics, measured rather than assumed

Four of these cost a rebuild of the node surface to discover. They are recorded so nobody re-derives
them, and each has a test in `tests/test_comfyui_schema.py` that fails if the surface stops respecting
it. Measured against `comfyui_frontend_package 1.48.7` and `comfy_api/latest/_io.py`.

- **`advanced` is not a hide.** The per-node expander exists only under Nodes 2.0 and is gated on the
  setting `Comfy.Node.AlwaysShowAdvancedWidgets`. Under the legacy canvas renderer it does nothing at
  all. Design as if every input is visible; treat the collapse as a bonus.
- **A label and its value share one row of about 38 characters.** So a long display name makes both
  unreadable. This is why every label in the pack is one or two words.
- **A multiline STRING with no placeholder prints its own input id** on the canvas:
  `addMultilineWidget` calls `createMultilineInputElement(default, placeholder || name)`. On a
  multiline widget the placeholder is the only label there is, so it has to be the label and the
  example at once and its first line has to stand alone under truncation. A **single-line** STRING's
  placeholder is not drawn at all on the legacy canvas, so there the display name carries everything.
- **Autogrow socket labels come from `names[ordinal]`, or from `prefix + ordinal` zero-based, and they
  overwrite whatever the template declared.** `autogrowOrdinalToName` returns
  `{name, display_name: s}` and `s` wins. So `TemplatePrefix` gives you `reference_0` on the canvas no
  matter what the template's `display_name` says, and `TemplateNames` is the only way to get one-based
  readable labels. Ids with a space in them (`pictures.picture 1`) round-trip through the API format
  and the workflow save without trouble; verified by running one.
- **The frontend already supports several inputs per grown item** (`inputSpecs` is a list and
  `ensureWidgetForInput` runs when its length is not 1), but the Python side takes a single template
  input and `_expand_schema_for_dynamic` reads only the first. That is the mechanical reason the
  picture notes are one positional block and a clip's role lives on a satellite node, not a preference.
- **An AUDIO is a Mapping, not necessarily a dict.** Load Video (Upload) hands out a `LazyAudioMap`
  that shells out to ffmpeg on first key access. `isinstance(audio, dict)` refuses it.

## Known gaps, honestly

- **The committed sample media cannot be rebuilt.** The two comparisons in `docs/media/` were
  produced by hand. Deferred on purpose, to be done only when the compiler improves enough to be
  worth re-shooting the samples, and only if it is. The risk being accepted is that the clips
  silently become evidence of an older version while the front page still claims a difference.

  The recipe survives without a script, which is why deferring is safe: both compiled briefs ship
  beside the clips, both reference plates ship, the dial command is in the README, and the seed and
  render settings are in the commit that added them.

  If it is ever written it cannot live in CI, because it needs ComfyUI with H3 loaded and a live
  endpoint. And it would not reproduce the same clips: renders are not identical across model or
  driver versions, so the check is whether the claim still holds, not whether the pixels match.

- **The style-LoRA registry is read but not usable end to end. `--lora` crashes, both ways.** TODO,
  deliberately deferred: proving this out needs the application that consumes it to exist first, so
  it can be tested against real weights and a real render rather than against a placeholder.

  What works: the registry loads a folder correctly and `h3ir loras` reports id, triggers, strength
  bounds, variants, conflicts and the author prose. `GET /v1/loras` serves it.

  Two separate defects behind that, and they fail differently:

  ```
  # variant mismatch, raised as an internal invariant instead of told to the caller
  h3ir compile "a fox in tall grass" --lora handpainted-anim-v2
  -> CompilerInvariantError: W11-lora-variant: handpainted-anim-v2 is trained for
     ['ref2va'] but this request routes to the fl2va checkpoint

  # variant matches, and the trigger splice produces text the validator rejects
  h3ir compile "the car rolls in" --image plate.jpg --lora handpainted-anim-v2
  -> CompilerInvariantError: R16-style-opening-malformed: a spliced clause keeps its
     capital mid-sentence ('with Hi'):
     'The target video is in hndpntd_anim_v2 style with High-contrast automotive...'
  ```

  The first is the validator being **right** and the handling being wrong: asking for a ref2va-only
  style on a request that routes elsewhere is a real user error and deserves a sentence saying so,
  not an invariant crash. The second is a genuine text bug: the trigger is spliced into the style
  opening without lowercasing what follows.

  And the part nobody has built at all: **nothing matches a request's own words to a registered
  style.** The owner's intent was that mentioning a look in plain language pulls the LoRA in and says
  so. Today only an explicit id does anything, and `"hand-painted animation look"` in the request
  text is ignored. `docs/design.md` and `docs/calling-the-api.md` both expose the surface, so an
  agent will discover styles and try to use them before this is fixed.

- **Audio references have no transcript source wired in.** `analyse_audio` accepts a transcript
  and the plumbing for it exists, but nothing calls whisper yet. Until it does, an attached audio
  reference is described from the caller's note alone. This matters more than it looks: the
  tokenizer emits `"<Audio j>: "` and nothing else, so the IR text is the *only* channel by which
  the conditioning encoder learns what that audio is. Wire whisper before shipping audio refs.
- **Video references now sample real frames**, at 10/50/90% of the clip, cached on content hash
  **and** on the fractions. `ffmpeg`/`ffprobe` are hard runtime dependencies of video references, not
  conveniences. The analyser raises rather than producing a card. Audio still has no transcript
  source wired in, so an attached audio reference is still described from the caller's note alone.
- **`refine()` re-runs the whole compile.** The cache keys make a prose-only refinement cheap in
  principle, but the fast path is not implemented. It re-analyses nothing (cards are cached) but
  does redo the beat sheet and all prose.
- **The eval suite is six briefs.** Enough to catch the regressions we have seen; not a broad
  quality benchmark. Add briefs when a new failure mode appears, not preemptively. It is also the
  only thing that found the mode-split bug below, and it found it the first time it was ever run
  end-to-end on the write-first path. Five of six briefs were falling back and no single-brief test
  could see it, because the one brief anybody had been testing by hand was the ref2va one.
- **One prompt per mode, and check that when you add a stage.** `compose.v2.txt` carries the
  full-reference guide; `compose_base.v1.txt` carries the base guide. `compose_prompt=None` picks by
  mode and that is the right default. Passing an explicit name overrides the choice for BOTH modes,
  which is what you want for an A/B and never what you want in production.
- **Thinking ON costs about 45 s on the planning call** versus ~5 s off. Whether it earns that is
  an open A/B, not a settled question; the eval loop is how to answer it.
- **`camera_style: "prose"` renders no camera sentence at all.** It exists as the A/B arm for
  "does the closed vocabulary matter"; it is not a finished alternative rendering. Do not ship it
  as a user-facing option until the A/B is run and the losing arm is either fixed or removed.
- **LoRA weights are never loaded here.** This layer plans the trigger injection and records what
  was chosen; patching the graph is the graph owner's job. `stacks_with_turbo: unknown` in a
  `howtouse.md` is waiting on someone's measurement.
- **`compose.v3.txt` is written but not the default.** It differs from v2 in exactly one paragraph:
  the "decide the edit" instruction, which in v2 ends *"and it is the worst outcome available"*:
  invented severity on a discretionary call, the same fault the validator audit removed from the
  rules. v3 states the spec's sentence instead. It is not the default because the arm5/arm6 pair
  (same pipeline, direct vs not) was generated against v2 and flipping the default mid-comparison
  changes two variables at once. Switch with `--compose compose.v3.txt`, score it, then promote it.

## The acceptance comparison

```bash
h3ir acceptance --image-a character.png --image-b creature.png --out acceptance/
```

Writes five prompt files, a wiring manifest for each, and a README explaining what each outcome
would mean. It does not submit anything. Arm D is the control that decides whether the labels
bind or the position does. Do not drop it, because without it a difference between arms A and B
has two explanations.

---
> Source: [ruashots/open-h3-ir](https://github.com/ruashots/open-h3-ir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
