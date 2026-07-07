## media-server

> NixOS configuration for `ahiru.pl`, a home server (Raspberry Pi 4). Host config

# CLAUDE.md — ahiru.pl NixOS config

NixOS configuration for `ahiru.pl`, a home server (Raspberry Pi 4). Host config
in `hosts/ahiru/`, service modules in `modules/services/`, imported by
`hosts/ahiru/default.nix`.

## Deploy

`origin/master` of `github.com/mruwnik/media-server` is the deploy branch — a
root systemd timer (`modules/services/updates.nix`) pulls it daily and runs
`nixos-rebuild switch`. To deploy now: push to master, then on the Pi
`git -C ~/nixos pull --ff-only && sudo nixos-rebuild switch --flake .#ahiru`.
Validate first without touching live: rsync the tree to `/tmp/x` on the Pi and
`nixos-rebuild build --flake path:/tmp/x#ahiru`. `dan` has **passwordless sudo**
(`wheelNeedsPassword = false`, intentional), so the switch runs non-interactively.

`/media/data` uses **POSIX ACLs** (`mask::r-x`) — `chmod g+w` is useless there;
use `setfacl`, and apply it *declaratively* via an activation script
(`stringAfter ["users"]`), not a one-time command, or the next rebuild wipes it
(see `media.nix` calibre + `mcp.nix` Unsorted). Music files are `dan:users`; the
`mcp` service user can't write them (only MPD stickers).

## Music & anime tooling

The music library lives at `/media/data/Music` (MPD root). Playback is MPD
(`:6600`) + myMPD (`:3001`-ish web UI). There is also an animethemes "haul" of
post-2012 anime OP/EDs and a rating/curation system built on top of MPD.

### Ratings — `modules/services/mpd-rating.nix` (`mpd_rating.py`)

Durable, portable song ratings. Two layers kept in sync:
- **file tags** (`FMPS_RATING` for opus/vorbis/flac, `POPM` for mp3) — the
  durable source of truth that travels with the file;
- **MPD `rating` sticker** (0–10 = stars×2, myMPD's scale) — the working copy
  myMPD and the MCP server can set without filesystem access.

Commands (run as `dan`; file writes need the owner):
- `mpdrate N [uri]` — rate the current (or given) track 0–5 stars. **N=1 bins +
  skips** (1 star = "delete"); **N=2/3 rate + skip** to the next (noted, move on);
  N=4/5 just rate.
- `mpdbin` — alias for `mpdrate 1` on the current track; `mpdbin --list` /
  `mpdbin --restore [query]`.
- `mpd-rating reap` — hourly `mpd-reaper.timer`: syncs stickers→tags, then bins
  every 1-star track to `/media/data/Music.bin` (recoverable via
  `mpd-rating restore`).

The MCP server (`raspberry-mcp`, separate repo) exposes `mpd_rate(stars, uri)`
which only sets the sticker (mcp can't write files); the reaper persists it to
the tag. Song-returning MCP tools include the rating when set.

Local SSH wrappers in this repo: `./mpdbin`, `./mpdrate` (call the box commands).

### The haul (TV-size discovery layer)

`/media/data/Music/Anime OPs & EDs (animethemes)/` — ~9000 OP/EDs pulled from
animethemes.moe (Ogg **Opus**, **TV-size ~90s**). It's a *discovery* layer: a
cheap broad set to audition and rate, not full songs. (The old "Yukito" pack, by
contrast, is full-length ~4min.)

### Full-length upgrades — `fullfetch.py` (on-demand script)

Source: `scripts/fullfetch.py` (deploy by `scp`-ing to `~/fullfetch.py` on the
box). Deliberately **not** a system service — it's an occasional tool, so it's
run ad-hoc via nix-shell (keeping `yt-dlp` fresh) rather than baking `yt-dlp`/
`ffmpeg` into the system closure:

    nix-shell -p yt-dlp ffmpeg python3Packages.mutagen \
      --run "python3 ~/fullfetch.py fetch [--dry-run] ..."

`fullfetch.py` replaces a track with the **official full-length audio** from
YouTube. For each selected track it reads the file's artist/title tags, searches
yt-dlp, and **only swaps when a confidence gate passes** (trusted channel —
`- Topic`/official/major J-label — plus an artist or title match, and a sane
duration). The swap is *in place* so the path (and its rating sticker) survives;
the original is backed up to `/media/data/Music.fullfetch-backup/` and dubious
matches go to `~/fullfetch_review.tsv` instead of being applied. Tracks already
longer than 2 min are left alone (they're already full) unless `--force`.

- `fullfetch fetch` — default: rated-≥2★ tracks under the haul. `--dir D`,
  `--all-rated`, `--min-stars N`, `--dry-run`, `--limit N`, or explicit `uri`s.
- `fullfetch revert [query]` — restore a backed-up original.
- `fullfetch list` — what's been upgraded.

Generic — works on any tagged track, not just the haul. (`./mpdbin`/`./mpdrate`
are committed ssh wrappers; `fullfetch` is intentionally script-only.)

## Tests / health

`tests/` is a bash health battery (`./tests/diagnostics.sh`, per-component
`tests/checks/*.sh`); an hourly monitor (`monitoring.nix`) runs a subset and
notifies on failures. Restarting `mcp` causes a ~90s outage (uvicorn waits for
streaming connections to drain) which can trip the mcp health checks.

## Code style

Python: pep-8, functional, early returns, small helpers, imports at top.
Embedded Python in nix modules uses `pkgs.writers.writePython3Bin` reading a
sibling `.py` (lint-clean under the module's `flakeIgnore`).

---
> Source: [mruwnik/media-server](https://github.com/mruwnik/media-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
