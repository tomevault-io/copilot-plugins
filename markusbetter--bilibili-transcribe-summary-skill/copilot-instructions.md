## bilibili-transcribe-summary-skill

> Use this skill when the user provides a Bilibili video link or asks to analyze, transcribe, summarize, or extract content from a Bilibili video. Trigger on both standard Bilibili video URLs like https://www.bilibili.com/video/BV... and short mobile links like https://b23.tv/.... This skill checks the local environment, tries anonymous subtitle extraction first, otherwise downloads the anonymous audio stream, renames the .m4s file to .mp3 without transcoding, sends it to SiliconFlow ASR when an API key is available, and then summarizes the transcript according to the user's request.


# Bilibili Transcribe Summary

Use this skill when the user gives a Bilibili link and wants to know what the video says or wants a summary.

## Trigger rules

Trigger this skill when both conditions are true:

- The user provides a Bilibili video link, a `BV...` id, or a `b23.tv` short link.
- The user intent is to analyze, summarize, transcribe, extract subtitles, or understand the video content.

## What this skill does

1. Check whether `node` is available and is version 18 or newer.
2. If the user wants ASR, check whether `SILICONFLOW_API_KEY` exists.
3. Run the pipeline script in `scripts/bilibili_pipeline.mjs`.
4. Prefer official subtitles when available.
5. If subtitles are unavailable, use the anonymous audio URL from the page playinfo.
6. Download the audio as `.m4s`, then rename it to `.mp3` without transcoding.
7. If an API key is present, send the renamed `.mp3` to SiliconFlow `TeleAI/TeleSpeechASR`.
8. Read the transcript and answer the user's request.

## First-run readiness marker

After the first successful `probe` or `run`, the script writes `.skill-ready.json` into the output directory.

If that marker already exists from a prior successful run, you can skip repeating setup reminders and go straight to the workflow. Only fall back to dependency or API-key guidance if the script actually fails.

## Quick commands

Probe only:

```bash
node scripts/bilibili_pipeline.mjs probe "https://www.bilibili.com/video/BV1R6PzzAE9k" --output-dir ./output
```

Run the full workflow:

```bash
node scripts/bilibili_pipeline.mjs run "https://www.bilibili.com/video/BV1R6PzzAE9k" --output-dir ./output
```

Run with an explicit key:

```bash
node scripts/bilibili_pipeline.mjs run "https://b23.tv/lsocHNd" --output-dir ./output --api-key "<YOUR_API_KEY>"
```

## Environment checks

Before the first successful run:

- Verify `node --version` works.
- If the user expects transcription, verify `SILICONFLOW_API_KEY` is set, or pass `--api-key` explicitly.
- If the environment blocks network access, request permission before running the script.

If `.skill-ready.json` already exists, do not repeat these setup checks unless the script fails.

If Node is missing, guide the user to install Node.js 18+ and then retry. Read `references/setup.md` for install commands.

If the API key is missing, tell the user to create one here and then retry:

- https://cloud.siliconflow.cn/me/account/ak

You can then suggest one of these commands:

```bash
set SILICONFLOW_API_KEY=your_key_here
```

```powershell
$env:SILICONFLOW_API_KEY="your_key_here"
```

## Output files

The script writes these files into the chosen output directory:

- `probe_result.json`: parsed page info, subtitle info, and candidate media URLs
- `audio.mp3`: the downloaded anonymous audio stream after renaming `.m4s` to `.mp3`
- `transcription_result.json`: SiliconFlow raw JSON result when ASR runs
- `transcript.txt`: plain transcript text from subtitles or ASR
- `.skill-ready.json`: indicates that the workflow has already run successfully at least once in this output directory

## Response rules

After a transcript is available:

- If the user asked for a full transcript, return or save the transcript text.
- If the user asked for a focused summary, summarize only the requested parts.
- If the user gave no summary preference, default to a concise key-point summary.
- Mention whether the transcript came from official subtitles or ASR.
- If subtitles and ASR are both unavailable, explain the blocker and stop.

## Portability

For Codex, OpenClaw, and Claude Code notes, read `references/tool-compatibility.md` when you need to adapt this folder to another agent environment.

## Notes

- This skill has been verified on both full Bilibili links and `b23.tv` short links.
- For the validated workflow in this project, renaming the downloaded `.m4s` audio file to `.mp3` was sufficient for SiliconFlow transcription, with no transcoding step.
- Anonymous access is opportunistic. Some videos may still fail due to Bilibili restrictions or missing page playinfo.

---
> Source: [Markusbetter/bilibili-transcribe-summary-skill](https://github.com/Markusbetter/bilibili-transcribe-summary-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
