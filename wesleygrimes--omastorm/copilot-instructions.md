## omastorm

> Omastorm is an Omarchy-native NEXRAD radar app: a Quickshell/QML view fed by a

# Working on omastorm

Omastorm is an Omarchy-native NEXRAD radar app: a Quickshell/QML view fed by a
headless engine. Read [README.md](README.md) first; it is the user-facing
install and use document, and the product intent lives there too. Then read
[DESIGN.md](DESIGN.md) for decided architecture and the reasoning behind it, and
[docs/protocol.md](docs/protocol.md) for the engine-to-UI contract. Decisions in
those files are settled; do not re-open them without asking.

`PLAN.md` is a local, ignored working file (the current phase and the next
action). It is never committed and is not in the clone. "Implement the next
step" means its **Next action** section. Do that one step, verify it, commit,
then update Next action to the following step per the handoff convention. Do
not start the step after it in the same session.

## Distribution

- The repository is the plugin. `omarchy plugin add` clones the default branch
  in full into `~/.config/omarchy/plugins/com.omastorm.radar`, and
  `omarchy plugin update` fast-forwards it. Everything tracked here lands on
  every user's disk, so keep generated output, media, and working notes out
  of git (see `.gitignore`); README media are assets on the plugin's GitHub
  Release and are linked by URL.
- The engine binary is a GitHub Release asset (`engine-<version>`) pinned by
  sha256 in `engine/release.pin`; `scripts/install-engine.sh` fetches and
  verifies it. Bump the pin only after the release it names exists.
- `manifest.json` `version` is the plugin's version; tag the same commit
  `v<version>`. The engine has its own version in `engine/Cargo.toml` and its
  own `engine-<version>` tag. The two move independently; the pin joins them.
- `site/` is the holding page at <https://omastorm.com>, hosted on Cloudflare
  Pages by direct upload; deploy and DNS notes are in [site/README.md](site/README.md).
- The Omarchy plugin ID is `com.omastorm.radar` (DESIGN.md, plugin identity).

## Run and verify

```sh
bash scripts/setup-fixture.sh                # once per fresh checkout: geography to embed, archived volume for tests
bash run.sh                                  # launch (needs GPU Quickshell); starts lean, goes live on the home station
OMASTORM_ARCHIVE=data/raw/KTLX20130520_201643_V06.gz bash run.sh   # start on the archived scan instead (offline)
OMASTORM_STYLE=STIPPLE bash run.sh           # PIXELS | GLYPHS | STIPPLE
target/debug/omastorm-engine stop            # end the shared daemon by hand (a launch replaces a stale build itself)
bash scripts/check.sh                        # the regression suite against a scratch daemon (~1 min); --gpu adds the rendering test
bash scripts/cargo.sh test --offline --locked -- --ignored rendering   # GPU pixel check on its own
bash scripts/capture-review.sh               # offscreen captures to review/ (ImageMagick; ignored output)
omarchy plugin validate .                    # the manifest check the shell and the marketplace apply
```

Install and launch need Rust and Quickshell only.

The rendering test and the captures need a working desktop OpenGL
context (offscreen platform, RHI OpenGL). A sandbox without a GPU cannot run
them; say so in the handoff rather than skipping silently. The software Qt
Quick backend is unsupported by design.

Run `bash scripts/check.sh` before every commit; it never touches the shared
daemon. After any shader, sampling, or camera change, add `--gpu` for the
ignored rendering test (`engine/tests/rendering.rs`, described in
`engine/README.md`) and run the capture review; the test replays the shader's
rule in Rust, so a deliberate rule change updates both. Rebuild the shaders
with `bash scripts/build-shader.sh` whenever `ui/shaders/radar.frag` or
`ui/shaders/tile.frag` changes and commit the `.qsb` files with them.

## Environment on the development machine

- Arch Linux with Omarchy. Quickshell 0.3.1. Nothing in install, launch, or
  the checks uses Python.
- Rust is a checkout-local toolchain under `.tools/` (ignored), not a system
  install. Always go through `bash scripts/cargo.sh ...`; bare `cargo` is not on
  PATH.
- Go is installed but is not used by this project.
- Omarchy theme files: `~/.local/state/omarchy/current/theme/{colors,shell}.toml`
  (a real directory, not a symlink). Theme-change hooks run from
  `~/.config/omarchy/hooks/theme-set.d/`. The shell's own file-watching pattern
  is in `/usr/share/omarchy/shell/Commons/Color.qml`.
- A Quickshell instance from an earlier session may still be running
  (`pgrep -a quickshell`). Launching can show two windows; kill only your own.
  Closing the window quits its Quickshell process; a `quickshell -p ui/shell.qml`
  process with no window is a leak worth investigating, not normal.
- The engine daemon outlives windows and is shared. After a rebuild, the next
  launch ends the old daemon and starts the new build (logged in
  `engine.log` and on the launcher's stderr); open windows reconnect on their
  own. `target/debug/omastorm-engine stop` ends it by hand; never `kill` it.
- The documented global key is `SUPER + SHIFT + R` with
  `omarchy shell shell toggle com.omastorm.radar '{}'`, free in Omarchy
  defaults. The user adds it to `~/.config/hypr/bindings.lua`; the plugin
  never writes that file.

## Guardrails

- Never modify Omarchy or system configuration. Read theme files; write nothing
  outside the repo, `$XDG_RUNTIME_DIR/omastorm/`, `$XDG_CACHE_HOME/omastorm/`,
  `$XDG_DATA_HOME/omastorm/` (`~/.local/share/omastorm/` by default),
  `~/.config/omastorm/`, and, only when the user runs
  `scripts/install-launcher.sh`, `$XDG_DATA_HOME/applications/omastorm.desktop`.
- Radar values never pass through JSON or QML JavaScript. The UI receives small
  state and GPU-ready textures only. Pan and zoom change uniforms.
- Keep OpenStreetMap (ODbL) and Natural Earth attribution with the data and in
  the UI.
- Python is not a project language. The golden files under `golden/` were
  produced once with pyart and are the decoder's answer key. Do not add Python
  dependencies.
- All three treatments stay. Glyphs is the default. Do not remove stipple; that
  decision is scheduled for the milestone 2 review.
- Show actual scan times and label archived data as archived. Distinguish
  missing, range-folded, and below-threshold from measured values.
- Launch never fetches anything. Network access happens only in the engine's
  live mode or in explicit setup scripts (`scripts/setup-fixture.sh`,
  `scripts/install-engine.sh`). Ordinary `bash run.sh` fetches nothing at launch; the daemon goes live only
  when a station is selected. Plugin `--ensure` may run the installer when
  there is no checkout debug engine. Nothing archived ships in the binary;
  `OMASTORM_ARCHIVE` is the development and check path.

## Conventions

- Small commits with a body that says why. No co-author or tool trailers in
  commit messages. Do not push unless asked.
- Keep the local PLAN.md and DESIGN.md short and current; follow the handoff
  convention at the end of PLAN.md at the close of every session.
- Verify library claims against docs or source before recommending them.
- When a visual decision is on the table, produce an offscreen capture and show
  it rather than describing it.

---
> Source: [wesleygrimes/omastorm](https://github.com/wesleygrimes/omastorm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
