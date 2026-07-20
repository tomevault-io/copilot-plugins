## bookast

> This is bookast, a simple podcast feed generator for audiobooks. It's goal is to leverage staic asset deployment to easily serve up audiobooks as podcast feeds by specifying a directory which contains audiofiles and optionally a cover image, it shuold generate an rss file that then can be manually uploaded to a staticc filehost.

This is bookast, a simple podcast feed generator for audiobooks. It's goal is to leverage staic asset deployment to easily serve up audiobooks as podcast feeds by specifying a directory which contains audiofiles and optionally a cover image, it shuold generate an rss file that then can be manually uploaded to a staticc filehost.

# CRITICAL: MEMORY MANAGEMENT
**CLAUDE'S PRIMARY RESPONSIBILITY**: Actively maintain this CLAUDE.md file throughout the project. Update it immediately when:
- User makes important decisions
- Implementation details are finalized
- New requirements are added
- Project direction changes
This is ESSENTIAL for maintaining project context across conversations.

# implementation-decisions
- **Language**: Go (chosen for simplicity and easy binary deployment)
- **Audio metadata library**: github.com/dhowden/tag (most popular, actively maintained, supports MP3/M4A/etc)
- **Supported formats**: MP3, M4A, M4B, AAC, FLAC, OGG
- **Directory structure**: Flat directories only (no nested folders for now)
- **Metadata sources**: ID3 tags first, fall back to filenames
- **Episode ordering**: Alphanumeric sorting
- **URL structure**: Directory name becomes part of URL path (e.g., `--base-url https://foo.bar/audiobooks hp1` → `https://foo.bar/audiobooks/hp1/file.mp3`)
- **URL encoding**: Proper escaping for spaces and special characters
- **RSS output**: One RSS file per directory, generated in source directory as `podcast.rss`
- **CLI interface**: `bookast --base-url <url> <directory>` (base-url is required)
- **Error handling**: Unix conventions - errors to stderr, proper exit codes, fail on unparseable files
- **Git workflow**: No branches - commit directly to main
- **Description fallback**: Use title as description when comment is empty or contains technical metadata like "iTunPGAP"
- **Episode pubDate**: Use current time + index (1 second intervals) for consistent chronological ordering in podcast clients

# library-selection-criteria
- **Popularity**: Must be widely used (check GitHub stars, imports, usage)
- **Active maintenance**: Recent commits, active issues/PRs, responsive maintainers
- **These are critical factors when choosing external dependencies**

# debugging-tips
- **Metadata debugging**: Use `mediainfo <audiofile>` to inspect actual metadata fields and compare against RSS output
- **URL encoding verification**: Check generated RSS URLs match expected encoding patterns

# readme-philosophy
Keep READMEs minimal and focused on the essentials that don't change often. Avoid sections like "Features" and "File structure" that become stale as the project evolves. Focus on: description, requirements, installation, usage only.

# testing-strategy
- **Test scope progression**: Unit tests (smallest scope) → Integration tests → End-to-end tests
- **Unit tests**: Cover pure utility functions with table-driven tests
- **Integration tests**: Test functions that interact with files and external tools (ffprobe)
- **End-to-end tests**: Use golden file approach - generate RSS output and compare against committed fixtures
- **Golden file workflow**: Run `./generate_test_fixtures.sh` when intentionally changing RSS output format
- **Test fixtures**: Minimal audio files created with ffmpeg; MP3s use ID3 tags via id3tag tool, M4A uses native metadata
- **Timestamp handling**: Normalize timestamps in golden file comparison since they change on each run
- **Test philosophy**: Golden files are better than brittle string assertions - easier to maintain and review changes

---
> Source: [cjlucas/bookast](https://github.com/cjlucas/bookast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
