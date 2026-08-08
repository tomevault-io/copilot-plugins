## videoanalyzer

> This is the setup repository for the video-analyze Claude Code skill. It reverse-engineers any social media video into a structured content intelligence report.

# video-analyze — Claude Code Skill

This is the setup repository for the video-analyze Claude Code skill. It reverse-engineers any social media video into a structured content intelligence report.

## What You Do When This Project Is Opened

When someone opens this repo in Claude Code, run the following sequence automatically — do not wait for them to ask.

---

### Step 1: Run the preflight check

```bash
python scripts/setup.py --check
```

- **Exit 0:** all dependencies are installed and an API key is configured. Skip to the "Ready" section below.
- **Exit 2:** binaries missing (yt-dlp / ffmpeg / ffprobe).
- **Exit 3:** API key missing.
- **Exit 4:** both binaries and API key missing.

If exit is non-zero, proceed to Step 2.

---

### Step 2: Run the installer

Tell the user: "Running install.py to set up your dependencies and install the skill."

Run:

```bash
python install.py
```

Show the full output. If install.py exits with a non-zero code, surface the exact error and stop — do not proceed past a failed install.

**Note for Windows:** after winget installs binaries, a terminal restart is required for the new PATH to take effect. If the installer warns about this, tell the user to close and reopen their terminal, then re-run `python install.py`.

**Note for Linux:** install.py will print manual install commands and exit. Guide the user through running them, then ask them to re-run `python install.py` once done.

---

### Step 3: Check the API key

After the installer finishes, run:

```bash
python scripts/setup.py --json
```

Parse the JSON output. If `has_api_key` is `false`, tell the user:

"The skill is installed. Transcription is optional — without a key, videos with native captions still get a full transcript, but videos without captions come back frames-only.

To enable transcription for caption-less videos:
1. Open `~/.config/watch/.env`
2. Add your key: `GROQ_API_KEY=your_key_here`

Get a free Groq key at console.groq.com/keys."

If `has_api_key` is `true`, confirm which backend is active and move on.

---

### Ready

Once setup is complete (all binaries present, API key optional), tell the user:

"The skill is installed and ready. Open Claude Code in any project and type:

/video-analyze [URL]

Supported: TikTok, Instagram, YouTube, Reddit, X, and anything yt-dlp supports.
Pass a URL directly or just type /video-analyze and Claude will ask for it."

---

## Hard Rules

- Always run the preflight check first. Never assume setup is already done.
- If install.py fails, show the full error output — do not hide or summarize it.
- Do not run /video-analyze from this repo directory. The skill is meant to be used from any other project after installation.
- Do not invent workarounds for failed installs. Surface the error and let the user fix the underlying issue.

---
> Source: [Mharis-code/videoanalyzer](https://github.com/Mharis-code/videoanalyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
