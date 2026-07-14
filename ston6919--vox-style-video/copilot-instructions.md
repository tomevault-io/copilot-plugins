## vox-style-video

> This project now uses the Kie.ai sequential-scene workflow only.

# VOX Kie.ai Video Project

This project now uses the Kie.ai sequential-scene workflow only.

Before doing video work, read:

1. `VOX-STYLE.md` - the locked visual rules for the VOX-style editorial collage look.
2. `VOX-SEQUENTIAL-SCENE-WORKFLOW.md` - the Kie.ai process for script, audio, sections, first image, scene generation, handoff frames, and stitching.

## Core Workflow

All VOX-style videos should be made as sequential Kie scenes:

1. Write or confirm the narration script.
2. Generate or choose the narration audio.
3. Split the transcript into 5-10 second sentence-safe sections.
4. Generate the first VOX-style image.
5. Generate each video scene from the previous scene's final frame.
6. Extract stable handoff frames between scenes.
7. Stitch scenes into the final MP4.

Keep the continuous paper-collage world intact: warm off-white background, grayscale halftone cutouts, restrained accent color, short labels, rack-focus style blur, overlapping paper pieces, and no hard visual resets.

## Commands

```bash
npm run kie:check      # check Kie credit balance
npm run kie:prompts    # write first-image and scene prompts from config/sections
npm run kie:run        # generate first image, scenes, and stitched output
```

Direct script usage:

```bash
KIE_API_KEY=... node scripts/kie-sequential-video.mjs check
node scripts/kie-sequential-video.mjs write-prompts
KIE_API_KEY=... node scripts/kie-sequential-video.mjs generate-first-image
KIE_API_KEY=... node scripts/kie-sequential-video.mjs generate-scenes --audio-id <kieAudioId>
node scripts/kie-sequential-video.mjs stitch
```

Use `--dry-run` to write task payloads without calling Kie.

## Project Structure

- `scripts/kie-sequential-video.mjs` - Kie.ai automation script.
- `storyboard/STORYBOARD-TEMPLATE.md` - reusable planning template.
- `storyboard/example.config.json` - neutral sample Kie workflow config.
- `storyboard/example-sections.json` - neutral sample timed narration sections.
- `prompts/` - generated Kie prompts, ignored by git.
- `outputs/` - Kie task payloads/results, generated images, handoff frames, and videos, ignored by git.
- `assets/audio/` - local narration audio and transcripts, ignored by git.

## Kie Rules

- Set `KIE_API_KEY` before commands that call the API.
- Do not let the model invent important readable text. Explicitly list allowed words, or generate blank cards and add exact text later.
- Each scene section should be longer than 5 seconds and no longer than 10 seconds.
- Do not split a sentence across scenes.
- Each scene prompt should preserve the previous frame and ask for a brief still hold before motion.
- Prefer PNG handoff frames when possible; use a stable near-final frame if the very last frame has motion blur.
- Review generated joins and use crossfades when continuity is imperfect.

---
> Source: [ston6919/vox-style-video](https://github.com/ston6919/vox-style-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
