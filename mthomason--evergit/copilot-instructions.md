## evergit

> This document defines the objectives and requirements for an automated agent tasked with creating and maintaining the **evergit** backup script.

# AGENTS.md

## Purpose

This document defines the objectives and requirements for an automated agent tasked with creating and maintaining the **evergit** backup script.
The script’s purpose is to download and maintain local backups of GitHub repositories in a **safe, non-destructive** manner across all major operating systems.

## Environment

* **Supported Operating Systems:** macOS, Windows, Linux, Unix
* **Python Version:** 3.13
* **Scheduler Options:**

  * macOS/Linux/Unix: `cron` or `systemd` timer
  * Windows: Task Scheduler
* **Dependencies:**

  * Python standard library only (no external modules)
  * The `git` command must be installed and available in the system `PATH`

## Task Summary

Create a single, modular Python script that automatically backs up GitHub repositories to a local disk.

The script should:

1. **Clone repositories** if they do not already exist locally.
2. **Pull updates** (non-destructive) for repositories that already exist.
3. **Pause briefly** between each operation to reduce API or network load.
4. **Use tabs for indentation** consistently throughout the file.
5. **Follow modern Python best practices**, including:

   * Type hints
   * Docstrings
   * Modular, function-based structure (in a single `.py` file)
   * A `main()` entry point and `if __name__ == "__main__":` guard
   * Graceful error handling and clear logging
6. **Support configuration files in both JSON and TOML formats**, automatically detecting file type by extension example (`.json`, `.toml`).

   * If no configuration file is found, the script should fall back to a hardcoded list of example repositories.
7. **Be non-destructive:**

   * The script must **never delete**, overwrite, or reset repositories.
   * It should only perform safe operations (`git clone` and `git pull`).
   * If a repository directory is currently in use (e.g., has uncommitted changes), the script must skip it safely and continue.
8. **Be platform independent**, functioning wherever Python 3.13 and `git` are installed.

## Configuration File Examples

### JSON Example (`repos.json`)

```json
{
	"backup_root": "/path/to/backup/folder",
	"repos": [
		"https://github.com/username/repo1.git",
		"git@github.com:username/repo2.git"
	]
}
```

### TOML Example (`evergit.toml`)

```toml
backup_root = "./evergit_backups"
sleep_seconds = 3.0
randomize_sleep = true
repos = [
	"https://github.com/github/docs.git",
	"https://github.com/mthomason/ObjectiveMorality.git",
]
```

## Script Behavior Overview

1. Load configuration from a user-specified file (JSON or TOML).

   * If none is found, use a built-in example list of repositories.
2. Verify the existence of the backup directory.
3. For each repository:

   * If the directory does **not** exist: perform a `git clone`.
   * If it **does** exist:

     * Verify that it’s a valid Git repository.
     * Attempt a `git pull` to fetch updates.
     * If there are uncommitted changes, skip with a warning (do not overwrite).
   * Sleep for a few seconds between operations.
4. Log all operations and errors to the console (and optionally to a file).

## Output and Logging

The script should provide clear, timestamped console output indicating progress and results.
It should be readable both in a terminal and when redirected to a log file.

### Format

Each log entry should include:

* **Timestamp** in ISO format (`YYYY-MM-DD HH:MM:SS`)
* **Log level** (`INFO`, `WARNING`, `ERROR`)
* **Message** describing the action or result

### Example Output

```
[2025-10-14 03:00:00] INFO  Starting backup run
[2025-10-14 03:00:01] INFO  Processing repo: repo1
[2025-10-14 03:00:05] INFO  repo1 - already up to date
[2025-10-14 03:00:10] INFO  Processing repo: repo2
[2025-10-14 03:00:15] WARNING  repo2 - uncommitted changes, skipping
[2025-10-14 03:00:20] INFO  Backup run complete
```

### Logging Implementation Notes

* Use Python’s built-in `logging` module.
* Log to `stdout` by default; optionally allow a `--log-file` argument.
* Use at least three levels: `INFO`, `WARNING`, and `ERROR`.
* The format string can follow this pattern:

  ```python
  logging.basicConfig(
      level=logging.INFO,
      format='[%(asctime)s] %(levelname)-7s %(message)s',
      datefmt='%Y-%m-%d %H:%M:%S'
  )
  ```

## Example Cron Setup (Linux/macOS)

Run once daily at 3 AM:

```bash
0 3 * * * /usr/bin/python3 /path/to/evergit.py --config /path/to/repos.toml >> /path/to/evergit.log 2>&1
```

### Example Task Scheduler Setup (Windows)

Run daily using a scheduled task that executes:

```
python.exe C:\path\to\evergit.py --config C:\path\to\repos.json
```

## Deliverable Expectations

The agent’s output should include:

* A complete, runnable Python script named `evergit.py`
* Modular function structure within a single file
* Inline comments and docstrings explaining each part
* Example usage for both JSON and TOML configurations
* Safe handling of partial failures (e.g., one repo fails, others continue)
* Consistent tab indentation
* Well-formatted, timestamped logging output

## Notes for the Agent

* Use only standard Python libraries (no external packages).

  * For TOML parsing, check whether the `toml` module is included in the environment (Python 3.13+); if not, handle gracefully or print a helpful message.
* Use `subprocess.run()` for Git commands.
* Use `pathlib.Path` for filesystem operations.
* Handle exceptions gracefully and ensure the process continues for other repositories.
* Never delete or modify user data beyond safe `git pull` operations.
* Prioritize readability and maintainability.

## Future Enhancements (Optional)

* Parallel or asynchronous updates using `asyncio` or `concurrent.futures`
* Remote backup support (e.g., external drives, network shares, or S3)
* Command-line arguments for extra control (`--verbose`, `--skip-errors`, etc.)
* Optional compression or pruning for old clones
* Configurable sleep intervals or randomized delay range

---
> Source: [mthomason/evergit](https://github.com/mthomason/evergit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
