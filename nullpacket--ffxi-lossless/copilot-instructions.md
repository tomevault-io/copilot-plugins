## ffxi-lossless

> For any LLM-based assistant (Claude, ChatGPT/Codex, Gemini, Copilot, Cursor, …) helping

# AGENTS.md — instructions for AI assistants

For any LLM-based assistant (Claude, ChatGPT/Codex, Gemini, Copilot, Cursor, …) helping
someone set up 32-bit frame generation with this repo. Humans want [README.md](README.md).

You are working on someone's **live game install and GPU settings**. Everything here is
reversible and documented, but confirm before changing system state, and prefer this
repo's scripts over ad-hoc commands.

## Rule 0 — talk them out of it if the numbers say so

Before building anything, establish two facts:

1. **Their display refresh rate.** `xrandr | grep '\*'`, or the `modes` files under
   `/sys/class/drm/*/`.
2. **Their current in-game framerate**, and whether the GPU is idle while it happens.

lsfg-vk uses a **fixed multiplier** with forced V-Sync, so `rendered = refresh / multiplier`.
On a 60 Hz panel at `multiplier = 2` the game renders **30** and presents 60. If they are
already getting 46-55 fps, this trades real frames for interpolated ones and adds latency —
often a downgrade. Say so plainly rather than building it because they asked.

Frame generation is a good fit when: framerate is well below refresh, steady rather than
stuttery, and the GPU has headroom. It is a poor fit when: they're near refresh already, or
the framerate is spiky (interpolation needs consistent input frames).

## Rule 0.5 — warn about black screens on swapchain recreation

Confirmed in FFXI: **black screens on every zone change** with the layer enabled, gone the
moment it is disabled. Not a crash — no access violations, no device loss, clean logs. The
layer does not recover from swapchain teardown/recreate.

Tell the user this up front, and have them test it deliberately: enable, then zone or change
resolution several times *before* concluding it works. A first impression of "visually
smoother" standing still can hide it completely.

There is no configuration workaround — `none` is the only pacing mode, and the fault is in
swapchain handling.

Fastest way to disable without touching a launcher config (no need to close Faugus/Lutris):
rename the manifest, e.g. `mv ~/.local/share/vulkan/implicit_layer.d/lsfgvk_i386.json{,.disabled}`.
The env var then finds no layer and is inert.

## Rule 1 — check GPU power state before blaming anything else

Old games frequently fail to make the driver raise clocks, so the card idles while playing.
On the machine this repo came from, fixing that was worth **~35%** — far more than frame
generation. The README's "Before you reach for frame generation" section has the commands
for both vendors. **Do this first.** Interpolation masks the symptom; clocks fix the cause.

Also check whether anything is actually saturated. Low GPU utilization *and* no CPU thread
near 100% means the bottleneck is draw-call submission, where `dxvk.conf`
(`maxFrameLatency=1`) can beat frame generation outright.

## Rule 2 — a wrong-architecture Vulkan layer fails SILENTLY

This is the single most important debugging fact here. If the layer library is the wrong
architecture, the loader **skips it without an error**. The environment variable is set,
the manifest is valid, nothing logs a failure, and the layer simply never loads.

Never conclude "it's enabled" from configuration alone. Always confirm it is *loaded*:

```bash
./verify.sh <pid-or-process-name>          # checks /proc/<pid>/maps
```

Two real examples of this failure mode:

- **NVIDIA Smooth Motion** (`NVPRESENT_ENABLE_SMOOTH_MOTION=1`) can never work with a
  32-bit game — NVIDIA ships `libnvidia-present.so` 64-bit only, with no `lib32` variant.
  The variable sets cleanly and does nothing. Don't recommend it for 32-bit titles.
- Installing distro `lsfg-vk` packages for a 32-bit game. They are `arch=('x86_64')` and
  depend on the 64-bit loader only. Same silent skip.

## Rule 3 — never read `/proc/<pid>/cmdline` of a game process

Some launchers pass account credentials as command-line arguments. `pgrep -af`,
`ps aux`, and `ps -f` all expose them and will leak them into the transcript. Use
`pgrep -x <name>`, or `ps -o pid=,comm=`. Reading `/proc/<pid>/environ` for a specific
variable is fine; dumping it wholesale is not.

## Hard-won details that will otherwise cost time

- **`version = 2` is mandatory** in `conf.toml`. Without it: `unsupported configuration
  version`, and the layer deactivates.
- **`Failed to find 'vkGetInstanceProcAddr' in layer`** usually means *no profile matched*,
  not a broken build. lsfg-vk's negotiate function returns an error when inactive and the
  loader reports it this way. Check `active_in` before suspecting the build.
- **Wine/Proton process matching**: lsfg-vk reads `/proc/self/exe` (the Wine loader), then
  falls back to scanning `/proc/self/maps` for a mapped `.exe`, and also checks
  `/proc/self/comm` — which **the kernel truncates to 15 characters**, often leaving a
  trailing dot. List both forms in `active_in`.
- **`allow_fp16 = true` on AMD** is a large uplift; on NVIDIA it does nothing, and on
  GTX 1000-series and older it is a slowdown.
- **`gpu` must be the GPU the game renders on.** Dual-GPU is unsupported; omitting it picks
  the primary, which is wrong on hybrid laptops.
- **The FPS counter will read LOWER, and that is correct.** Overlays count rendered frames,
  beneath the interpolation layer. Expect ~half. Do not "fix" this. The game's own frame
  limiter also becomes a no-op, since lsfg-vk gates the render loop downstream of it.
- **A 32-bit Vulkan ICD is required**, not just the loader. See the README table.
- **Don't run a full system upgrade** to get `cmake`/`ninja`/Vulkan headers. `build.sh`
  fetches them locally on purpose — a distro upgrade can pull a new GPU driver as a side
  effect and disturb a working setup. If package downloads 404 on Arch-family systems, that
  is a stale local database, and it is the user's call whether to `-Syu`.

## Verifying a change actually worked

Prefer measurement over inference:

- `./verify.sh <pid>` — is the layer really loaded?
- The game's own output should log `lsfg-vk: using profile with name '<name>'`.
- GPU utilization should *rise* once interpolation runs (it's real GPU work).
- The offline CLI benchmark (see README) exercises the DLL reader, shader extraction and
  GPU compute without launching the game — use it to separate a build problem from a game
  integration problem.

## Scope

Do not redistribute a compiled layer binary. lsfg-vk is GPL-3.0-or-later; `build.sh` clones
and builds it locally so no upstream code ships here, and the source-provision obligation
does not arise. Lossless Scaling is proprietary and must not be copied or bundled.

---
> Source: [nullpacket/ffxi-lossless](https://github.com/nullpacket/ffxi-lossless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
