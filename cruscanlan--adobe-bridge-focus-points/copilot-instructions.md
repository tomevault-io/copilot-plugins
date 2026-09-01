## adobe-bridge-focus-points

> A CEP plugin for Adobe Bridge that reads the camera's autofocus point(s) from

# Bridge Focus Points

A CEP plugin for Adobe Bridge that reads the camera's autofocus point(s) from
maker-note metadata and draws them as an overlay on the selected photo. The
Bridge equivalent of digiKam's native display and the Focus-Points Lightroom
plugin.

Full design rationale lives in [project.md](project.md). This file is the
working reference for how the code is actually put together.

## The one idea that makes this tractable

**We do not parse maker notes.** That problem is solved twice already and we
delegate to both:

- **exiftool** (bundled, shelled out) does all metadata extraction.
- **The Focus-Points Lr plugin** holds the per-manufacturer geometry that turns
  exiftool output into pixel coordinates. We port that, manufacturer by
  manufacturer (Lua -> JS). digiKam (`reference/digikam-focuspoint-metadataengine/`)
  is a secondary cross-reference only — it uses Exiv2, not exiftool, so its
  extraction layer does not transfer.

## Architecture (decided — do not re-evaluate)

- **CEP**, not UXP (Bridge has no UXP). Host id `KBRG`. Bridge 16 ships CEP 12.
- **Node.js enabled** in the panel (`--enable-nodejs --mixed-context` in the
  manifest) so we can `child_process` to exiftool. CEP 12 ships Node 17.7.1.
- **Bundled exiftool**, shelled out. Never reimplement.
- **HTML `<img>` + SVG overlay** for rendering (resolution-independent; drops
  the Lr plugin's ImageMagick dependency).
- **Windows** target (Cru runs Adobe in a Windows VFIO VM). Mac later.

### Data flow

```
Bridge selection
  -> host/index.jsx (ExtendScript)  : reports selected file path; pings panel on change
  -> client/main.js (Node/JS)       : spawns exiftool, picks delegate, renders
  -> vendor/exiftool                : returns maker-note / AF tags as JSON
  -> client/delegates/<mfr>.js      : computes AF box(es) in image-pixel coords
  -> SVG overlay                    : draws the box(es) on the preview
```

ExtendScript (ES3) is the *only* thing that can talk to Bridge but is feeble, so
`host/index.jsx` stays tiny: it answers "what file is selected?" and dispatches a
`CSXSEvent` (`com.cru.bridgefocuspoints.selectionChanged`) when the selection
changes. Everything else is in the Node/JS panel. The two talk via
`CSInterface.evalScript()` and CSXS events.

## Layout

```
CSXS/manifest.xml          extension config, host KBRG, Node enable
.debug                     remote-debug port 8088
host/index.jsx             ExtendScript: selection -> path (+ change event)
client/
  index.html               panel shell
  main.js                  Node/JS: exiftool -> delegate -> SVG overlay
  styles.css
  lib/
    CSInterface.js         Adobe CEP 12 library (vendored)
    exiftool.js            spawn wrapper: readTags() / extractPreview()
  delegates/
    fujifilm.js            ported Fuji geometry (getAfPoints)
    dji.js                 DJI: preview + flight info, no AF data (none exists)
vendor/exiftool/           bundled Windows exiftool 13.55 (complete standalone)
tools/                     dev-only: ZXPSignCmd.exe + cert.p12 + sign-install.sh
reference/                 read-only porting sources (Lr plugin, digiKam)
fixtures/                  test RAFs; expected/ holds rendered AF-box oracles
```

## How the Fuji geometry works (the ported core)

Ported from `reference/focuspoints.lrplugin/FujifilmDelegates.lua`
(`getAfPoints`) into `client/delegates/fujifilm.js`. For the X-series / GFX:

- `FocusPixel` is a single `(x, y)` in the coordinate system of the **embedded
  JPEG**, whose dimensions are reported by `ExifImageWidth`/`ExifImageHeight`.
- The transform is a pure scale to the displayed image:
  `xScale = displayW / ExifImageWidth`, same for y. The box is a square of side
  `min(displayW, displayH) * 0.04` (the Lr "medium" default) centred on the
  scaled point.
- **We render the embedded preview**, extracted with exiftool. On the X-H2 that
  preview is exactly `ExifImageWidth x ExifImageHeight` (4416x2944 on the test
  bodies), so `xScale = yScale = 1.0` and `FocusPixel` maps directly. The SVG
  overlay uses `viewBox="0 0 imgW imgH"` so coordinates are in image-pixel space
  regardless of on-screen zoom.
- Faces (`FacesDetected`/`FacesPositions`), subject detection
  (`FaceElementPositions`, exiftool >= 12.44) and tele-converter crop
  (`CropSize`/`CropTopLeft`, >= 12.82) use the same scale and are already ported.
  None of the current fixtures exercise them.

**OOC caveat:** the `FocusPixel`/`ExifImageWidth` reference is lost or corrupted
on RAF->DNG conversion. Feed straight-out-of-camera `.RAF` (or the OOC JPEG).
`fujifilm.makerNotesFound()` rejects DNGs and files missing `InternalSerialNumber`.

### Tag access convention

exiftool is invoked with `-json -s`, so tag keys are short PascalCase
(`FocusPixel`, `ExifImageWidth`, `FacesDetected`, `InternalSerialNumber`).
Delegates read tags by those names. `FocusPixel` comes back as the string
`"2515 1164"`; dimensions come back numeric.

## Orientation handling (done — and NOT in the delegate)

AF coordinates are relative to the camera's native (unrotated) orientation.
Rotated/portrait shots are handled, but **not** the way the Lr plugin does it.
Because we draw a live SVG overlay (not Mogrify-baked pixels), we don't rotate
each point — we rotate the *whole image+overlay container* and leave the points
in native space. The delegate is orientation-agnostic and untouched. The logic
lives entirely in `client/main.js` + `styles.css`:

- `#preview` gets CSS `image-orientation: none` so CEF never auto-rotates from
  the extracted preview's own Orientation tag. This is essential: a RAF's
  embedded preview carries an Orientation tag (CEF would rotate it) while an OOC
  JPEG's tiny preview does not (CEF would leave it sideways) — so without this we
  get inconsistent behavior between the two. Forcing `none` makes the decoded
  pixels always native, and `naturalWidth/Height` always the native frame.
- `orientationOf(tags)` maps the exiftool `Orientation` string to `{rotate,
  mirror}`; `layout()` fits the native preview into the stage (swapping bounds
  for 90/270 so the *rotated* footprint fits) and applies the same CSS
  `transform` to image and overlay so they rotate as one unit about a shared
  centre. The SVG `viewBox` stays in native pixel space, so faces/crop boxes ride
  along for free.
- Mirrored EXIF variants (2/4/5/7) are implemented but not yet visually verified
  (no mirrored fixture). The four rotations (incl. `Rotate 270 CW`, the
  `DSCF2659` fixtures) are verified in Bridge.

**Crop caveat (open):** an OOC JPEG's embedded `PreviewImage` can be a small
letterboxed thumbnail (e.g. 640×480 4:3 for a 3:2 frame), so `FocusPixel`/dim
scaling is slightly off on the *JPEG* path. The RAF preview is a clean full-frame
3:2 and maps exactly — feed RAFs. Real in-camera crop (`CropSize`/`CropTopLeft`,
digital tele-converter) is drawn but untested. Getting the box in the wrong place
is almost always a transform bug, not a parse bug.

## ⚠️ Bridge 2026 requires a SIGNED extension (the big gotcha)

Bridge 2026 (16.0.3, CEP 12) **will not render an unsigned CEP extension**, even
with `PlayerDebugMode=1`. It *lists* the extension in Window > Extensions but
loading fails — an `Embedded` panel **hard-crashes Bridge** ("A nullptr was
dereferenced"), and dialog types (`Modeless`/`ModalDialog`) show "Unable to
load". The CEP log only says "Signature verification failed". The built-in
`HelloBridge` sample works **because it is code-signed**. Note: the symptoms
differ by `UI Type` but the cause is the same (unsigned). Once signed, **all
types work — including `Embedded`, which we ship** (it docks like the native
Metadata panel). The earlier "Embedded crashes" was just the unsigned rejection.

Consequence baked into the project:

- **Every install must be signed.** `tools/sign-install.sh` does it: stage the
  shippable dirs → `ZXPSignCmd -sign` with a self-signed cert
  (`tools/cert.p12`) → unzip the `.zxp` into
  `%APPDATA%\Adobe\CEP\extensions\com.cru.bridgefocuspoints`. A self-signed cert
  is enough because `PlayerDebugMode` is honored. **Editing any file invalidates
  the signature — re-run the script after every change.** (`tools/ZXPSignCmd.exe`
  is Adobe's official 4.1.103 build; dev-only, never shipped.)
- **Claude runs the install, not Cru.** `bash tools/sign-install.sh` works in
  Claude's environment, so whenever a change is ready to test in Bridge, Claude
  runs the sign-install script itself and *then* asks Cru to test — never tell
  Cru to run it. Cru's only manual steps are restarting Bridge and reporting what
  the panel shows.

## Selection updates by polling, not events

Bridge's ExtendScript selection-changed event is unreliable, so `client/main.js`
**polls** `getSelectedFile()` (~600 ms) and only re-runs exiftool/render when the
path changes. `host/index.jsx` still dispatches a `CSXSEvent` too, but that is a
bonus path — polling is what actually drives live updates.

## Status

Working end-to-end in Bridge: select a Fuji RAF → preview + green AF box render
on the focus point, and the panel updates live as the selection changes.
exiftool bundle verified (13.55); Fuji primary-point geometry ported & visually
verified against all three fixtures (`fixtures/expected/*-afbox.jpg`); signed
`Embedded` panel docks in Bridge 2026 like a native panel. Faces / AI subject
boxes confirmed working with live data. **Orientation** (incl. portrait
`Rotate 270 CW`) verified on the `DSCF2659` fixtures.

Bottom-bar UI: per-layer toggles (Focus / Faces / Crop, persisted via
localStorage; a layer with no data in the current image is greyed/disabled) and
a shooting-info readout (model · focal · aperture · shutter · ISO · focus-mode ·
AF area+size · detected subject). Toggling re-renders the cached result; no
re-run of exiftool.

Crop: **aspect-ratio crop** (`RawImageAspectRatio`, e.g. 16:9 → centred dashed
box, trim top/bottom) verified on `DSCF2561.RAF`. Digital tele-converter crop
(`CropSize`/`CropTopLeft`) is drawn but still untested (no fixture). The two are
mutually exclusive in the delegate (TC crop wins).

AF-frame size: the green box is sized from the camera's real Single-Point
`AFAreaPointSize` ordinal via the tunable `AF_POINT_SIZE_FRACTION` table in
`fujifilm.js` (each step = +2% of the short side), falling back to the fixed
`FOCUS_BOX_SIZE` when absent. **Relative** sizing verified across fixtures
(SP 3 < 5 < 6); **absolute** scale is an unconfirmed approximation — needs
in-camera ground-truth testing to calibrate the table. No prior art: neither the
Lr nor digiKam ports use this ordinal at all.

**DJI** (`client/delegates/dji.js`): no geometry exists to port — the Lr plugin
and digiKam have zero DJI support, and exiftool decodes no DJI AF tags; the only
focus-related output is `AFDebugInfo`, an undecoded 256-byte binary blob.
Diffing the three Air 2S (FC3411) fixtures (`DJI_0426/0638/0685.DNG`): u32s at
0x00/0x08 constant (18, 200), 0x10/0x14 always 0x7FFF ("invalid"), and only
0x04 (68–77), 0x0C (76–78) and 0xD0 (2600/2700 — likely focus-motor position)
vary, drifting like AF metrics rather than swinging like tap-to-focus
coordinates. No field looks like a focus *position*; deciding for sure needs
two shots with focus deliberately tapped at opposite corners. The delegate
therefore draws nothing by design: it renders the DNG's embedded 960×630
SubIFD1 preview, a drone-flavoured info bar (model-code map FC3411→Air 2S etc.,
35mm-eq focal, AGL altitude, gimbal pitch from XMP-drone-dji), and an honest
"DJI does not record AF-point coordinates" status. Delegates may now provide
optional `matches(tags)` (dispatch; Mavic 2/3 report Make=Hasselblad),
`formatShootingInfo(tags)` and `NO_AF_MESSAGE` hooks — main.js falls back to
its generic versions when absent. Decoding `AFDebugInfo` would need several
samples with known (tap-to-focus) positions to diff.

**Unsupported cameras degrade gracefully**: no delegate is no longer fatal —
the panel still renders the embedded preview (or, for JPEGs with no embedded
PreviewImage, the file itself) plus the generic shooting-info line, with a
"<Make> is not supported yet" status. Only the overlay needs a delegate.

Not yet done / next steps:
- Calibrate `AF_POINT_SIZE_FRACTION` against real in-camera AF-frame testing
  (absolute scale only; shape confirmed). Zone mode (`AFAreaZoneSize`) unmodelled.
- Mirrored EXIF orientations (2/4/5/7) — coded, no fixture to verify.
- JPEG-path letterbox crop (see orientation caveat) + digital-TC crop test.
- Distinguish face-element types (Head/Body/Face/Eye via `FaceElementTypes`)
  instead of drawing all as one yellow box; also `FacePositions` vs the
  delegate's `FacesPositions` tag-name mismatch (latent dead branch).
- Remaining manufacturers (Canon, Nikon, Sony, …), one at a time.
- A proper `.zxp` for distribution (the dev self-sign is fine for personal use).

## Install / debug (Windows VM)

1. Enable debug mode: `regedit` ->
   `HKEY_CURRENT_USER\Software\Adobe\CSXS.12`, add string `PlayerDebugMode` = `1`.
   (Required, but NOT sufficient on its own — the extension must also be signed,
   see above.)
2. Build + sign + install: run `bash tools/sign-install.sh` from the repo. It
   installs only the shippable dirs (`CSXS/`, `client/`, `host/`, `vendor/`,
   `.debug`) — never `reference/` or `fixtures/`. `client/lib/exiftool.js` finds
   the binary at `<extension-root>/vendor/exiftool/exiftool.exe`.
3. Restart Bridge (fully quit first). Open from **Window > Extensions > Focus
   Points**.
4. DevTools: with the `.debug` file present, browse to `http://localhost:8088`
   in Chrome/Edge. CEP logs: `%TEMP%\CEP12-KBRG.log`; Bridge's own log:
   `%APPDATA%\Adobe\Bridge 2026\BridgeLog.log`.

## Tooling notes

- **No system Node** on the dev machine — the panel uses CEP's bundled Node, so
  that is fine at runtime. It means the delegate JS cannot be unit-tested via a
  plain `node` invocation here without installing Node first.
- exiftool: `vendor/exiftool/exiftool.exe` (complete standalone, 13.55). The
  copy originally dropped in was missing its Perl runtime; it was replaced with
  the complete bundle from the Lr plugin's `bin/`.
- ImageMagick (`reference/focuspoints.lrplugin/bin/ImageMagick/magick.exe`) is
  used **only** for dev-time fixture verification (drawing oracle boxes), never
  shipped or required at runtime.
```

---
> Source: [CruScanlan/adobe-bridge-focus-points](https://github.com/CruScanlan/adobe-bridge-focus-points) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->
