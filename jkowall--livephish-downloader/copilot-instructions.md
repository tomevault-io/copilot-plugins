## livephish-downloader

> Guidance for coding agents working in this repository.

# AGENTS.md

Guidance for coding agents working in this repository.

## Scope and instruction priority

- These instructions apply to the entire repository.
- Follow direct user instructions first, then this file.
- Read `README.md` and the relevant code before changing behavior.
- Keep changes focused. Preserve unrelated work in the checkout.

## Project summary

LivePhish Downloader is a single-file Python command-line tool that uses a
visible Chrome session to download content available through a user's
authenticated LivePhish account. Selenium drives the site and reads Chrome
performance logs, `python-dotenv` loads optional local credentials, and
`requests` downloads recognized audio stream URLs. Selenium Manager resolves a
compatible ChromeDriver.

Repository: <https://github.com/jkowall/LivePhish-Downloader>

### Repository map

- `livephish_browser_downloader.py`: CLI, Selenium automation, stream URL
  capture, filename generation, checkpoints, and downloads
- `pyproject.toml`: canonical package, Python, dependency, and entry-point
  metadata
- `requirements.txt`: bounded runtime dependency compatibility manifest
- `.env.example`: safe template for optional automatic-login credentials
- `tests/`: standard-library unit tests for isolated helper behavior
- `README.md`: installation, usage, operating modes, and troubleshooting
- `CHANGELOG.md`: user-visible release history
- `CONTRIBUTING.md`: contributor setup and validation checklist
- `docs/MANUAL_SMOKE_TEST.md`: required live-browser validation checklist
- `.github/workflows/ci.yml`: cross-platform Python validation
- `LICENSE`: Apache License 2.0
- `.gitignore`: local environments, generated files, and downloads

`pyproject.toml` is the dependency source of truth. Keep the compatible bounds
in `requirements.txt` synchronized with it, and update installation or
troubleshooting documentation when dependencies change. The program must not
install packages or create an environment at runtime.

## Behavioral model

The CLI has three operating paths:

1. With no mode flag, the user logs in, opens one Stash item manually, and the
   script attempts to process every detected track. Login can use an ignored
   `.env` file or remain manual.
2. With `--all`, the script selects a Stash tab and processes every detected
   item. `--type` applies only to this path and defaults to `playlists`.
3. With `--interactive`, the user starts each track and confirms it in the
   terminal. Automatic single-item mode falls back to this workflow when it
   cannot detect tracks unless bounded automatic-selection flags are active.

Automated selection can be previewed with `--dry-run`, resumed from a 1-based
global position with `--start-at`, and capped with `--limit`. In `--all`, the
ordered track stream spans item boundaries. `--retry-failed` first selects
failed, expired, or unprocessed keys from
`<output>/.livephish-checkpoint.json`, then applies start and limit.

Important implementation constraints:

- Optional automatic login may read `LIVEPHISH_EMAIL` and
  `LIVEPHISH_PASSWORD` from the process environment or an ignored `.env` file.
- Enter credentials only on `https://id.livephish.com`. Never send them to a
  different host or over plain HTTP.
- Never print credentials, include them in exceptions, add real values to
  `.env.example`, or commit any `.env` variant other than `.env.example`.
- Preserve manual login as the fallback for missing credentials, MFA, rejected
  credentials, or identity-page changes.
- Quality behavior is explicit: `best` enables HiFi, prefers FLAC, and accepts a
  player fallback; `lossless` enables HiFi and fails or skips non-FLAC capture;
  `player` leaves the account setting unchanged and accepts the player stream.
  The default is `best`.
- Keep the first supported M4A or MP3 request as a fallback only when the quality
  policy permits it, and allow only a short grace period for FLAC so each track
  does not incur the full capture timeout.
- Never transcode a lossy source or describe it as lossless.
- Do not add authentication bypasses, DRM circumvention, or access to content
  outside the user's valid subscription.
- Stream capture must remain limited to validated HTTPS audio media hosts and
  paths. Preserve the case-insensitive exclusion for preview clips unless a
  verified behavior change requires otherwise.
- Never print or persist credentials, cookies, authorization headers, signed
  stream URLs, URL queries, or fragments. Checkpoints are URL-free.
- Existing destination files are skipped only after validating their audio
  signature. Invalid existing files must not be overwritten. Write new
  downloads to a same-directory `.part` path, remove failed or interrupted
  partials, and use an atomic replace only after completion.
- Dry runs must not start playback, download media, write checkpoints, or mark
  tracks successful.
- Keep checkpoints contained in the selected output directory and update them
  after each track result. `--retry-failed` requires an existing compatible
  checkpoint.
- LivePhish is a dynamic web application. Re-query DOM elements after
  navigation or playback changes instead of retaining stale Selenium elements.
- Selector changes should keep specific selectors ahead of broad fallbacks and
  should not silently turn unrelated buttons or page elements into tracks.
- Browser cleanup must remain deterministic for every workflow, including
  interactive completion, early return, failure, and terminal interruption.

## Development setup

```bash
python3 -m venv venv
source venv/bin/activate
python -m pip install -e .
```

On Windows PowerShell, activate the environment with
`venv\Scripts\Activate.ps1`. In Command Prompt, use
`venv\Scripts\activate.bat`.

## Change guidelines

- Follow PEP 8 and use descriptive names.
- Add or update docstrings when function behavior changes.
- Prefer focused functions over adding more logic to an existing long method.
- Catch specific exceptions when changing error handling. The existing broad
  catches are not a pattern to extend without a concrete Selenium reason.
- Avoid new dependencies unless the standard library or an existing dependency
  cannot reasonably solve the problem.
- If a dependency changes, update `pyproject.toml`, `requirements.txt`,
  installation docs, and any relevant troubleshooting guidance together.
- Keep CLI help, defaults, examples, and `README.md` synchronized.
- Do not bump `__version__` for documentation-only changes. For a user-visible
  feature or behavior change, update it as part of the same change.

## Validation

Run the smallest checks that cover the change.

### Documentation-only changes

```bash
git diff --check
```

Also verify that commands, defaults, filenames, and mode descriptions match the
current script.

### Python changes

```bash
python -m pip install -e .
python -m pip check
python3 -m py_compile livephish_browser_downloader.py
python livephish_browser_downloader.py --help
python livephish_browser_downloader.py --version
python -m unittest discover -s tests -v
```

### Browser automation or download changes

Follow `docs/MANUAL_SMOKE_TEST.md` with credentials and a valid subscription.
At minimum, exercise the changed login, Stash selection, quality, capture,
filename, atomic download, checkpoint, resume, and cleanup paths.

Never claim the browser smoke test passed unless it was actually run against
LivePhish. Report `PASS`, `PARTIAL`, or `NOT RUN` and list skipped checks.

## Documentation requirements

- Update `README.md` for CLI, setup, output, compatibility, or workflow changes.
- Update `CONTRIBUTING.md` when contributor setup or validation changes.
- Update `pyproject.toml` and `requirements.txt` together when runtime
  dependencies change.
- Update `.env.example` when supported environment variables change, using
  placeholders only.
- Keep examples copyable and use `python -m pip` inside an activated virtual
  environment.
- Describe known limitations rather than presenting UI-dependent automation as
  guaranteed behavior.

## Git and commits

- Inspect `git status` before and after editing.
- Do not discard or overwrite unrelated user changes.
- Use clear Conventional Commit-style subjects, such as `fix: Handle expired
  stream URLs gracefully` or `docs: Clarify all-items mode`.
- All commits must be GPG signed. Use `git commit -S` or configure
  `commit.gpgsign=true`.
- Do not commit generated downloads, virtual environments, logs, credentials,
  cookies, or captured stream URLs.
- Before staging an authentication change, use `git check-ignore .env` and
  inspect the staged diff for credential material.

## Legal and responsible use

This project is for personal use with a valid LivePhish subscription. Changes
must respect copyright, rate limits, authentication boundaries, and LivePhish's
terms of service.

---
> Source: [jkowall/LivePhish-Downloader](https://github.com/jkowall/LivePhish-Downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
