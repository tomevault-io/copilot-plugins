## bloomcore-github-io

> Static GitHub Pages site. Brand identity is in `DESIGN.md` — palette, typography, motion rules. Source of truth for any composition or graphic work in this repo.

# bloomcore.github.io

Static GitHub Pages site. Brand identity is in `DESIGN.md` — palette, typography, motion rules. Source of truth for any composition or graphic work in this repo.

## Hero video (HyperFrames)

The hero background on `index.html` plays `hf-intro/intro.mp4` via a native `<video autoplay muted loop playsinline>` tag. The MP4 is produced from the HyperFrames composition at `hf-intro/index.html`.

`hf-intro/` is its own HyperFrames project (its `index.html` is the composition) because this repo's own `index.html` is the website.

### Render pipeline

```bash
cd hf-intro
npx hyperframes lint
npx hyperframes render -o intro.mp4    # ~3 min; default quality ≈ 8.5 Mbps (too heavy)

# Recompress for web (≈5× smaller at visually indistinguishable quality for noisy/grainy content)
ffmpeg -y -i intro.mp4 -c:v libx264 -preset slow -crf 28 -pix_fmt yuv420p \
       -movflags +faststart -an intro.web.mp4 \
  && mv intro.web.mp4 intro.mp4
```

Why each flag:
- `-crf 28` — quality target. Lower = bigger + sharper. 23 default, 28 good for grainy/abstract backgrounds, 30+ only for very noisy content.
- `-preset slow` — worth the extra encode time for a one-off shipping asset.
- `-pix_fmt yuv420p` — required for Safari and broad browser compatibility.
- `-movflags +faststart` — moves the moov atom to the front so playback starts before the whole file downloads.
- `-an` — strip audio (our compositions have none; container may still carry silence).

The `--video-bitrate` and `--crf` flags on `npx hyperframes render` are unreliable in the current version — treat ffmpeg recompress as the way to hit a target size.

### Editing the composition

`DURATION` (top of the `<script>` block) must stay in sync with `data-duration` on both the root `[data-composition-id="intro"]` div and the `#content` clip. Currently `20`.

For iteration without re-rendering: open `hf-intro/index.html` directly in a browser. A bootstrap at the bottom of the script auto-plays when the HyperFrames runtime isn't detected, so GSAP runs live in the browser at full fidelity. Only re-render when shipping.

### Common lint fixes

- Timed elements (anything with `data-start` / `data-duration`) need `class="clip"` for the runtime to control visibility.
- The root composition div needs `data-start="0"` and `data-duration="N"` in addition to `data-width` / `data-height`.

## Hero text overlay (`hf-hero-text/`)

Separate composition that renders a WebM with alpha, overlaid on top of `hf-intro/intro.mp4` via the `<video class="hero-text-overlay">` element in `index.html`. The overlay animates the hero H1 + subhead, then fades to hand off to the real DOM text beneath.

### Render — must pass `--format webm` for alpha

```bash
cd hf-hero-text
npx hyperframes lint
npx hyperframes render --format webm -o intro.webm
```

Without `--format webm`, HyperFrames renders H.264/yuv420p into the `.webm` container regardless of the `-o` extension — the file plays with an opaque background and hides the hero video beneath it. Verify alpha after render:

```bash
ffprobe -show_streams intro.webm 2>&1 | grep ALPHA_MODE
# Expect: TAG:ALPHA_MODE=1
```

The composition's `html, body { background: transparent; }` is also required — without it, captured frames come out yuv420p even with `--format webm`.

---
> Source: [bloomcore/bloomcore.github.io](https://github.com/bloomcore/bloomcore.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
