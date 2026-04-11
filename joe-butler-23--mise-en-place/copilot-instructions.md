## mise-en-place

> All Tauri app work (building, running, testing) on Linux requires the provided Nix shell for GTK/GLib, WebKit, and Chromium runtime dependencies:

# Agent Notes

## Nix shell requirements

All Tauri app work (building, running, testing) on Linux requires the provided Nix shell for GTK/GLib, WebKit, and Chromium runtime dependencies:

```bash
cd ~/development/mise-en-place
nix-shell
```

The shell provides: nodejs_20, pkg-config, glib, gtk3, webkitgtk_4_1, libsoup_3, libappindicator-gtk3, cairo, pango, harfbuzz, gdk-pixbuf, atk, openssl, clang, llvm lld, plus Chromium/Puppeteer runtime libs (nss, nspr, dbus, at-spi2-core, cups, expat, libxkbcommon, mesa, alsa-lib, systemd, X11 libs, patchelf).

It also sets `PKG_CONFIG_PATH` and `LD_LIBRARY_PATH` for cargo/Chromium runtime discovery.

**Important:** Always run commands inside `nix-shell` (or via `nix-shell --run "..."`) when working with Tauri commands/builds/tests.

---

## Skill References

Skills are ad hoc overlays only. Domain behavior should be defined here and in `mep --help`.

Optional ad hoc references:
- `~/development/ai/tooling/skills/create-skill/SKILL.md`
- `~/development/ai/tooling/skills/writing-style/SKILL.md`
- `./.agents/skills/mep-ai-led-qa/SKILL.md` (project-local, read-only QA workflow)

## Domain Interface Contract (CLI-First)

1. `mep` CLI and `--help` output are the canonical interface for cooking/shopping workflows.
2. `AGENTS.md` should explain routing/safety; skills are ad hoc overlays and should stay concise.
3. If `--help` does not explain a repeated workflow, patch `mep` help/docs in this repo.

## Troubleshooting References

Detailed troubleshooting and historical lessons were moved to docs:

- `docs/lessons-learned-debugging-2026-02-17.md`
- `docs/lessons-learned-recipe-capture-2026-02-19.md`
- `docs/engineering-guardrails.md`

---

## Testing protocols

### 1. Unit tests (root project — Obsidian plugin)

```bash
npm test          # vitest run (one-shot)
npm run test:watch  # vitest watch mode
```

No nix-shell needed for unit tests.

### 2. Tauri standalone — dev server (manual testing)

```bash
# Use the helper script
./scripts/start-tauri.sh
```

**Note:** If you see old errors or styles not updating, press **Ctrl+Shift+R** (or Cmd+Shift+R) in the app window to force a hard reload.

### 3. Tauri standalone — Playwright smoke tests

The Playwright config (`playwright.config.ts`) auto-builds and previews the web app before running tests.

**LIMITATION:** These tests run in a web browser context without access to the real file system or `~/vault`. They catch rendering errors and basic logic, but **miss** errors related to real data (YAML parsing, Moment.js dates, file stats).

```bash
nix-shell
npx playwright install --with-deps  # first time only
npm run test:smoke
```

### 4. Testing with Real Data (Console Errors)

To catch runtime errors that only happen with real vault data (like YAML parsing issues):

1.  **Unit Tests:** Write Vitest tests for data processing logic (e.g. `field-manager.test.ts`).
    ```bash
    nix-shell
    npm test src/modules/organiser/tests/field-manager.test.ts
    ```

2.  **Console Capture Script:** Runs the Tauri app and captures stderr/console output.
    ```bash
    nix-shell
    ./scripts/test-tauri-console.sh
    ```

### 5. Tauri standalone — production preview

```bash
nix-shell
npm run build:web
npm run preview:web    # serves on http://127.0.0.1:4173
```

### 5. Cline browser testing (Puppeteer)

Cline's built-in `browser_action` tool uses a bundled Chromium binary. On NixOS this binary needs its RPATH patched to find system libraries. If the browser fails to launch with "cannot open shared object file" errors:

```bash
# Check what's missing
ldd /path/to/puppeteer/chrome | grep "not found"

# Patch with patchelf (find the store path for missing libs, append to RPATH)
CHROME="$HOME/.config/VSCodium/User/globalStorage/saoudrizwan.cline-nightly/puppeteer/.chromium-browser-snapshots/chromium/linux-1549714/chrome-linux/chrome"
CURRENT_RPATH=$(nix-shell -p patchelf --run "patchelf --print-rpath '$CHROME'")
MISSING_LIB_DIR=$(find /nix/store -maxdepth 3 -name 'libXXX.so*' 2>/dev/null | sort -V | tail -1 | xargs dirname)
nix-shell -p patchelf --run "patchelf --set-rpath '$CURRENT_RPATH:$MISSING_LIB_DIR' '$CHROME'"
```

**Note:** `mesa-libgbm` is not available as a nixpkgs attribute — it only exists as a NixOS system derivation. Reference it via store path directly (e.g. `find /nix/store -name 'libgbm.so.1' -path '*mesa-libgbm*'`).

When using `browser_action` to test the Tauri standalone app:
1. Start the dev server inside nix-shell first: `nix-shell --run "npm run dev -- --host 127.0.0.1 --port 5173"`
2. Use `browser_action launch` with the URL from step 1
3. Always close the browser before using other tools

---

## Key file locations

| What | Where |
|------|-------|
| Nix shell config | `shell.nix` |
| Frontend app entry | `src/entry.tsx` → `src/App.tsx` |
| Tauri command bridge | `src/tauri-api/commands.ts` |
| Tauri backend entry | `src-tauri/src/main.rs` |
| Vite config | `vite.config.ts` |
| Playwright config | `playwright.config.ts` |
| Smoke tests | `tests/smoke.spec.ts` |
| Weekly organiser CSS | `styles.css` |
| Standalone shell CSS | `src/standalone.css` |
| Platform abstraction | `src/platform.ts` |
| Unit test config | `vitest.config.ts` |

---

## mep CLI (Recipe Import + Shopping Sync)

The `mep` CLI is the canonical cooking/shopping interface for this repo.

### Structure

| Component | Location |
|-----------|----------|
| Shared primitives (config + pipe parsing) | `mep-core/` |
| CLI commands and workflows | `mep-cli/` |
| Contract docs | `docs/mep-cli-contracts.md` |
| AI-led QA protocol | `docs/mep-ai-led-qa.md` |

### Build & Install

```bash
cargo build -p mep-core -p mep-cli
cargo install --path mep-cli
```

### Active Commands

```bash
mep recipe import-url <url>... --recipes-dir <dir>
mep recipe upgrade [<slug>...] --recipes-dir <dir> [--all]
mep sl add-recipe [<slug>...] --recipes-dir <dir> [--all-scheduled] [--mode plan|apply]
mep sl sync snapshot --project <id>
mep sl sync validate --desired /path/to/desired.json
mep sl sync plan --desired /path/to/desired.json
mep sl sync apply --plan /path/to/plan.json
mep sl sync rollback --snapshot /path/to/snapshot.json --mode plan|apply
```

### Architecture Notes

- **LLM-first semantics**: import uses one model call and strict output validation.
- **Deterministic shopping flow**: parse pipe lines, apply `ignore`/`merge`, dedupe/sum, then sync.
- **No legacy fallback stack**: removed registry/parser/resolver/adjudicator/memory command families.
- **Config-only customization**: `$MEP_CONFIG_DIR/cooking.toml` controls labels, ignore list, merge rules, and optional prompt overlay.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joe-butler-23)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joe-butler-23)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
