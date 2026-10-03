## thimble

> install.sh asks up to four questions before it installs anything, and each flag below answers one without asking. Without a terminal it refuses to run until every question it would ask has its flag, and lists the missing ones. `thimble uninstall` takes back the plugin and the trust:

# thimble — notes for Claude Code sessions in this repo

## What install.sh asks

install.sh asks up to four questions before it installs anything, and each flag below answers one without asking. Without a terminal it refuses to run until every question it would ask has its flag, and lists the missing ones. `thimble uninstall` takes back the plugin and the trust:
- **Browser** (`--browser bundled | system | off`): thimble takes screenshots of the cards and views it draws, to check and improve them. For the best experience it downloads Playwright's headless Chromium (about 210 MB, 650 MB on disk). With a no it uses the Chrome or Edge installed on the machine, which install.sh test-launches, since a policy can block automation; with neither, thimble can't check and improve its cards and views.
- **Sandbox** (`--sandbox-deps` / `--no-sandbox-deps`, Linux only, asked while the sandbox can't run): thimble's agents run their Bash only in Claude Code's sandbox, which needs bubblewrap and socat, and on Ubuntu 23.10 or later an AppArmor profile for bwrap. A yes installs them with sudo. With a no, thimble's agents won't run until the sandbox works.
- **Plugin** (`--plugin` / `--no-plugin`): adds thimble to `~/.claude/settings.json` and `~/.claude/plugins`, so thimble is available in every `claude` session from startup. The `thimble` command works either way, and so does thimble-cc-mod: when a no left thimble's marketplace out, `thimble cc-mod on` registers it, on the yes to its own question.
- **Trust** (`--trust-workspaces` / `--no-trust-workspaces`): adds thimble's workspaces folder (`~/.thimble/app/workspaces` for a Global install, `<clone>/workspaces` for a Dev install), where thimble keeps each workspace and runs its agents, to `~/.claude.json`. The orientation, its critic, the writers and view builds need it: they run as Claude Code background agents, which Claude Code starts only in a trusted folder.

## Installing

When the user asks you to install thimble (for example "install thimble from https://github.com/safety-research/thimble"), do the whole install for them:

1. If `thimble doctor` already runs, say what is installed and offer `thimble update` instead. Otherwise ask which install they want (with AskUserQuestion where it exists):
   - **Global** (recommended): the latest release into `~/.thimble/app` (`$THIMBLE_HOME/app` when THIMBLE_HOME is set), Node 20+ only for custom views and the sandbox card code and code tickets run in, updated from releases by `thimble update`.
   - **Dev**: a git clone in a folder they choose, installed in place with the test tools, needs Node 20.19+, 22.13+ or 24+, updated with `git pull` (which `thimble update` runs).
2. Get the files. Global: in a `mktemp -d` folder download the latest release's `thimble-*.zip`: with gh, `gh release download --repo safety-research/thimble --pattern 'thimble-*.zip'`; without gh, fetch the `browser_download_url` of that asset from `https://api.github.com/repos/safety-research/thimble/releases/latest` with `curl -fsSL`. Unzip it. If there is no release yet, say so and offer Dev. Dev: `git clone https://github.com/safety-research/thimble.git <folder>`; if this session already runs in a checkout, that checkout can be the folder.
3. Before installing anything, run `bash scripts/install.sh --dry-run` in the unzipped folder or the clone. It changes nothing. It prints what install.sh installs and where, then the questions it would ask on a terminal, each with the flag for every answer. Show the user that list, then ask each of its questions with AskUserQuestion, in its words and with its answers as the options. Also tell them every question install.sh skipped and why (an earlier answer, or the folder trusted already), with the folder and the flag that changes it, as its list says. Ask nothing else, and don't answer a question for the user, even one whose answer looks obvious or harmless: each changes their Claude Code setup, and only they know what they want of it.
4. Run `bash scripts/install.sh` there with one flag per answer, then (Global) delete the download folder. If install.sh says sudo needs a password it can't ask for, show the user the commands it printed and ask them to run those in their own terminal. If it, or its dry run, says Node is older than a checkout needs, tell the user and offer to upgrade Node the way this machine manages it (Homebrew, nvm, their package manager); do it only on a yes, then run install.sh again.
5. Run `thimble doctor` (`~/.local/bin/thimble doctor` while `~/.local/bin` is not on PATH). If the installer's last step asks for lines in the shell startup file (PATH, THIMBLE_HOME), offer to add them for the user. End with a short reply: where thimble is installed and that they run `thimble` in a folder of transcripts. Mention doctor's output only for a check that failed.

To change an answer later, run install.sh again with its flag (ask as in step 3). When `thimble update` stops because a question has no flag, ask the questions it lists as in step 3 and run `thimble update` again with their flags, which it passes on to install.sh. An install older than 0.3.0 has a `thimble update` that takes no flags: unzip the latest release and run its `bash scripts/install.sh --dir <the install folder>` with the flags instead. A re-run skips the steps that are done and never downloads the browser again once it is there.

If anything goes wrong that you can't fix, tell the user they can reach the maintainer, [@mjoerke](https://github.com/mjoerke) on GitHub, and that `thimble feedback "<what went wrong>"` writes a problem report to attach.

If install.sh fails, follow INSTALL.md. A Dev install uses the checkout in place (marketplace `thimble`, `backend/.venv`, the UI in `frontend/dist`); a Global install copies the release to `~/.thimble/app` (marketplace `thimble-local`). Both link `~/.local/bin/thimble` to the install's `plugin/bin/thimble`.

CONTRIBUTING.md says how to set up a checkout, run it in development and run the tests.

---
> Source: [safety-research/thimble](https://github.com/safety-research/thimble) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
