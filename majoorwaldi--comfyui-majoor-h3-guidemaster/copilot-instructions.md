## comfyui-majoor-h3-guidemaster

> This file is the maintenance contract for every coding agent or contributor touching this repository.

# AGENTS.md — ComfyUI-Majoor-H3-GuideMaster

This file is the maintenance contract for every coding agent or contributor touching this repository.

## 1. Mission

`ComfyUI-Majoor-H3-GuideMaster` is a thin, maintainable UI/compiler layer over **native MiniMax H3 behavior in ComfyUI**. It must never become a fork of the H3 model, sampler, VAE, or conditioning implementation.

The public product surface is **one node only**:

- `MajoorH3GuideMaster`

The node compiles:

- visual H3 guides -> native `conditioning[...]["minimax_keyframes"]`
- audio H3 guides -> native keyframe `audio_latent`

Do not monkey-patch ComfyUI core.

### Current node surface

Sockets, in schema order. Changing this list is a compatibility-sensitive modification.

| Socket | Type | Role |
| --- | --- | --- |
| `positive` | CONDITIONING | in/out; carries `minimax_keyframes` |
| `latent` | LATENT | in/out; passed through unchanged |
| `vae` / `audio_vae` | VAE | optional; required only by visual / audio guide slots |
| `first_frame` / `last_frame` | IMAGE | optional AddGuide-style IN / OUT anchors |
| `reference_media` | IMAGE, VIDEO (`io.MultiType`) | optional, **preview only**; discarded in `execute` |
| `audio_reference` | AUDIO | optional, **preview only**; waveform under the ruler, discarded in `execute` |
| `guide_image_N` | IMAGE | Autogrow visual guide slots |
| `guide_audio_N` | AUDIO | Autogrow audio guide slots |
| `timeline_json` | STRING | the only persisted UI state |

---

## 2. Mandatory source-reading gate — BEFORE EVERY MODIFICATION

Before changing Python behavior, node schemas, H3 timing, guide logic, frontend socket handling, or serialization, **re-read the relevant current upstream source**. Do not rely on memory, old local copies, or this document alone.

### Always read

1. Current ComfyUI repository / current target commit:
   - https://github.com/Comfy-Org/ComfyUI
2. Current native MiniMax H3 nodes:
   - https://github.com/Comfy-Org/ComfyUI/blob/master/comfy_extras/nodes_minimax_h3.py
3. Native Add Guide PR and all follow-up commits touching the H3 node/model since it merged:
   - https://github.com/Comfy-Org/ComfyUI/pull/15439
4. Current H3 packed model / timing behavior:
   - https://github.com/Comfy-Org/ComfyUI/blob/master/comfy/ldm/minimax/model.py
5. Current H3 `BaseModel` integration, especially `MiniMaxH3.extra_conds`:
   - https://github.com/Comfy-Org/ComfyUI/blob/master/comfy/model_base.py

### Additionally read for V3 node/schema changes

6. Current `comfy_api.latest` examples in ComfyUI. Prefer an actively maintained core node using `io.ComfyNode`, `io.Schema`, `io.Autogrow`, `io.NodeOutput`, and `ComfyExtension`.
6b. Before touching `reference_media`, re-read `io.MultiType` in the installed `comfy_api/latest/_io.py`. Its `Input(id, types=[...])` signature and the comma-joined `io_type` it produces are what make one socket accept both `IMAGE` and `VIDEO`.
7. Official custom-node V3 / migration documentation if the API contract has changed:
   - https://docs.comfy.org/custom-nodes/overview
   - https://docs.comfy.org/custom-nodes/backend/server_overview

### Additionally read for frontend changes

8. ComfyUI frontend extension architecture and current hooks:
    - https://github.com/Comfy-Org/ComfyUI_frontend/blob/main/docs/extensions/core.md
    - https://docs.comfy.org/custom-nodes/js/javascript_overview
    - https://docs.comfy.org/custom-nodes/js/javascript_hooks
9. Current DOM widget implementation before changing `addDOMWidget`, widget visibility, sizing, or serialization:
    - https://github.com/Comfy-Org/ComfyUI_frontend/blob/main/src/scripts/domWidget.ts
10. Before changing drag/drop or file-open handling, re-read the frontend's own document-level drop handler (`addDropHandler` in the installed frontend bundle). It claims dropped media unless the panel calls `preventDefault()` **and** `stopPropagation()` first.
11. Before changing `web/ui/preview_sources.js`, re-read the loader nodes it inspects — native `LoadVideo` (`comfy_extras/nodes_video.py`, widget `file`), `LoadImage`, and any VHS loader in use. This module reads other nodes' widget/preview state, so it is the most upstream-fragile file in the repository.

### Record what changed upstream

For any compatibility-sensitive modification, note in the PR/commit message or development notes:

- upstream ComfyUI commit SHA inspected;
- whether PR #15439 semantics changed;
- any changed values/types for `FRAME_PER_TOKEN`, `FRAME_RESCALE`, or keyframe fields.

If the current upstream behavior conflicts with this repository, **update tests first**, then adapt the smallest compatibility layer. Never copy a large upstream file into this repo.

---

## 3. Architecture invariants

### Backend boundaries

- `h3_guidemaster/core/constants.py` — only H3 constants mirrored for validation/fallback logic.
- `h3_guidemaster/core/types.py` — immutable-ish data contracts/dataclasses.
- `h3_guidemaster/core/state.py` — JSON schema parsing and validation only.
- `h3_guidemaster/core/slots.py` — Autogrow socket mapping only.
- `h3_guidemaster/core/latent.py` — H3 AV latent inspection/timeline geometry only.
- `h3_guidemaster/core/media.py` — visual/audio encode adapters only.
- `h3_guidemaster/core/guides.py` — timeline -> native H3 keyframes only.
- `h3_guidemaster/core/compat.py` — upstream feature/contract detection only.
- `h3_guidemaster/core/compiler.py` — orchestration; it must stay thin.
- `h3_guidemaster/nodes/guide_master.py` — ComfyUI V3 schema and execute bridge only.
- `h3_guidemaster/extension.py` — extension registration only.

### Frontend boundaries

- `web/guide_master.js` — extension registration only.
- `web/ui/state.js` — state normalization/serialization and time conversions only.
- `web/ui/controller.js` — widget-state lifecycle synchronization and the graph-signature refresh tracker only; never persist from `nodeCreated`.
- `web/ui/slots.js` — graph socket discovery only.
- `web/ui/local_media.js` — local browser video/image-sequence loading, sampling, and object-URL lifetime only.
- `web/ui/filmstrip.js` — filmstrip frame selection/rendering and IN/OUT/guide overlays only.
- `web/ui/previews.js` — connected IMAGE socket preview resolution and `/view` URL building only.
- `web/ui/preview_sources.js` — read-only resolution of an upstream loader's own preview state (native `LoadVideo`, VHS loaders, `LoadImage`, `GetVideoComponents`) into a media descriptor; pure, no `api` import, no DOM.
- `web/ui/reference_media.js` — turning a resolved media descriptor into filmstrip reference frames; no graph execution.
- `web/ui/guide_sync.js` — connected guide-slot synchronization and guide display helpers only.
- `web/ui/timeline.js` — timeline visualization + guide-marker interaction only.
- `web/ui/waveform.js` — audio envelope maths, decoding, and canvas drawing only; no timeline layout knowledge beyond the frame-axis mapping.
- `web/ui/panel.js` — compose controls and synchronize state with `timeline_json`.
- `web/ui/styles.js` — scoped styles only.

### File-size discipline

A file becoming difficult to understand in one screen/context is a design warning. As a soft limit:

- Python production files: target < 250 lines.
- JS production files: target < 300 lines.

Split by responsibility before adding a second unrelated subsystem to a file.

---

## 4. Native H3 contracts that must remain true

### Guides

GuideMaster must emit the same keyframe payload the native Add Guide path consumes:

```python
{
    "resolved_frame_index": int,
    "latent": optional_video_latent,
    "audio_latent": optional_audio_latent,
}
```

Rules:

- Image/clip guide and audio guide at the same frame may be merged into one keyframe.
- Multi-frame visual batches follow native H3 valid guide lengths: `5, 22, 39, ...` (`17k + 5`) by snapping **down**, matching `MiniMaxH3AddGuide`.
- H3 video timing is 24 fps.
- Negative frame support, if exposed, must resolve exactly like native Add Guide.
- Audio guides start on the same target timeline coordinate and crop to remaining H3 audio duration using current native timing.
- Preserve pre-existing `minimax_keyframes` on incoming conditioning and append GuideMaster keyframes deterministically.

Never invent a `strength` field in `minimax_keyframes` unless upstream H3 officially adds one.

### Existing latent/conditioning metadata

- The latent is passed through untouched (a shallow dict copy); GuideMaster never writes latent keys.
- Preserve conditioning metadata and existing H3 keyframes.

---

## 5. Compatibility policy

GuideMaster intentionally depends on native H3 capabilities rather than patching them.

At runtime, `core/compat.py` must fail with a clear actionable message if required contracts are missing, including:

- `MiniMaxH3AddGuide` / PR #15439 generation;
- H3 `FRAME_PER_TOKEN` / `FRAME_RESCALE` compatibility.

When upstream intentionally changes a contract, change the tests and adapter together. Do not silently disable features.

---

## 6. State and migration rules

Persistent UI state lives only in `timeline_json`.

- Current schema version: `2`.
- Never change the meaning of an existing field in place.
- A breaking schema change requires `version += 1` and an explicit migration function.
- Unknown/corrupt state must fail safely or reset to defaults in the frontend; backend validation remains authoritative.
- H3 fps in persisted state is fixed at `24` unless upstream H3 itself changes.
- v1 migration maps legacy `reference.selected_frame` to `timeline.selected_frame` and drops obsolete Guide Intent and `mask` fields.
- Local preview media paths/blobs are session-only browser state and must never be serialized into `timeline_json`.

---

## 7. UX rules

- Keep one visible node, not a chain of helper nodes.
- The timeline must remain optional convenience; backend inputs and saved workflows remain deterministic.
- Filmstrip preview media comes from browser Drop/Open, from the optional `reference_media` input, or is represented as an empty N-frame timeline.
- `reference_media` (`io.MultiType`, `IMAGE, VIDEO`) and `audio_reference` (`AUDIO`) are preview-only inputs. Both are deleted in `execute` and must never reach the compiler, `timeline_json`, or a saved workflow's semantics.
- The waveform builds automatically on source change, exactly like the reference filmstrip: keyed by the resolved source, guarded by a busy flag with a retry, and forced by its own round button. `AUTO_AUDIO_BYTES` (64 MB) caps the automatic pass; a forced reload passes `maxBytes: 0`.
- The size guard reads `Content-Length` and rejects **before** buffering the body: `decodeAudioData` cannot decode a fragment, so an oversized file must never be pulled at all.
- The waveform is decoded client-side from the loader's file via `/view`, keyed by the resolved source so a graph refresh never re-fetches it. Audio longer than the timeline is cropped, never squeezed: the horizontal axis stays the frame axis.
- `audio_reference` also accepts the AUDIO output of a video loader; the file resolved is then the video itself. A video loader's frame trim is converted to a source-time window (`videoAudioWindow`) and applied to the decoded samples, so the waveform matches the audio the graph receives.
- Building reference frames must never execute the graph: the UI reads the connected loader's existing preview state and samples it client-side. Loader inspection is best-effort — an unresolved source reports an actionable message and leaves the timeline untouched.
- Sample the loader's **original file** via `/api/view` (`206`, byte-range seekable), never its own preview URL. VHS `/viewvideo` is a chunked live transcode with no `Content-Length` and no `Accept-Ranges`, so a `<video>` cannot seek it and `duration` is unusable. Verified against a running server on 2026-08-27.
- A loader's trim is replayed client-side (`skip_first_frames` -> offset, `select_every_nth` -> stride, `frame_load_cap` -> bound, `force_rate` -> sampling rate). Filmstrip frame numbers are output-batch frames, not source-file frames.
- Filmstrip thumbnail budgets are `MAX_PREVIEW_FRAMES` (forced, 800) and `AUTO_PREVIEW_FRAMES` (automatic, 120) in `web/ui/local_media.js`. Change them there only; never hardcode a third limit.
- The reference filmstrip builds automatically when its source changes, at the automatic budget; the round button forces the full one. An automatic pass must never replace Drop/Open media, must not resurrect a filmstrip the user cleared, and reports failure as a suggestion rather than an error.
- Any status message that outlives a `renderAll()` must go through the sticky notice, never straight to `mediaInfo.textContent`: every loader re-renders after finishing.
- `first_frame` and `last_frame` are real AddGuide-style endpoint anchors at frame 0 / final target frame and must remain separate from preview-only local media.
- No media path may fail silently. Every import (Drop, Open Media, reference build) reports its outcome in the filmstrip status line and logs the underlying error to the console.
- Every wait on browser media (`loadedmetadata`, `seeked`, presented frame) must be bounded by a timeout. An undecodable source must produce a message, never an endless "loading" state.
- Every visible control must have a real backend effect, except the explicitly preview-only filmstrip controls (Open Media, reference build, Clear Preview, Condense).
- Do not persist `timeline_json` during `nodeCreated`; saved workflow widget values are restored later. Re-sync in `loadedGraphNode`.
- Never expose fake model controls such as unsupported guide strength.
- Use subdued UI styling. Accent is for interaction/selection, not decoration.
- Round action buttons report progress through `data-state` (`idle | busy | partial | ready | error`) styled from the existing muted palette. Colour must never be the only carrier: the same state belongs in the button's tooltip and in the status line. Honour `prefers-reduced-motion` for the busy animation.
- A long-running `busy` pass (filmstrip sampling) reports live progress (`onProgress(done, total)`) into the button tooltip and status line, throttled (~80ms) and text-only. Never repaint the filmstrip/waveform itself per frame — that fights the sampler for the main thread.
- A control with nothing to act on is disabled, not silently inert.
- Any drag-only action must have a keyboard/numeric alternative.
- The ruler itself scrubs the playhead (click/drag on its background, excluding markers/endpoints) and keeps the filmstrip in view. Live drag mutates `state.timeline.selected_frame` and does a cheap DOM-only follow (`updateFilmstripSelection` in panel.js); a full `commit()` happens only on release/keypress, mirroring the existing setGuideFrame/commitGuideFrame split. Never rebuild the filmstrip's thumbnails on every pointer move.
- Error messages should name the guide/frame that caused the failure.

---

## 8. Testing rules — mandatory

Use red-green-refactor for behavior changes.

Before production code:

1. add/change the smallest failing test;
2. run it and verify the expected failure;
3. implement the minimal behavior;
4. run focused tests;
5. run the full suite before completion.

Required commands:

```bash
PYTHONPATH=. python -m pytest -q
node --test "tests/js/*.test.mjs"
for f in web/guide_master.js web/ui/*.js; do node --check "$f"; done
python -m compileall -q h3_guidemaster __init__.py
```

Quote the JS glob: the shell must not expand it before `node --test` sees it.

For changes touching real Comfy contracts, also perform a smoke import against a current ComfyUI checkout whenever practical.

Do not declare compatibility based only on mocked tests.

---

## 9. Dependency policy

- No third-party custom-node dependency.
- Reuse PyTorch, torchaudio, and ComfyUI-provided modules already required by native H3.
- Frontend stays vanilla ES modules; do not add React/Vue/build tooling for this single-node UI.
- Do not vendor ComfyUI source.

---

## 10. Definition of done

A modification is done only when:

- upstream source gate completed;
- architecture boundaries respected;
- focused tests pass;
- full Python tests pass;
- JS state tests pass;
- every JS module passes `node --check`;
- Python compile check passes;
- no cache/build artifacts are committed;
- README/AGENTS compatibility notes are updated if behavior changed.

---
> Source: [MajoorWaldi/ComfyUI-Majoor-H3-GuideMaster](https://github.com/MajoorWaldi/ComfyUI-Majoor-H3-GuideMaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
