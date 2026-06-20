## yomimushi

> Guidance for working in this repo. Read this before changing the download logic.

# CLAUDE.md

Guidance for working in this repo. Read this before changing the download logic.

## What this is

A standalone **Electron desktop app** that logs into the store, lists a user's
purchased library, and downloads + decrypts titles into DRM-free EPUB (everything,
including comics — see the "always EPUB" gotcha). It is a port of the relevant half of the PyQt6 Python
downloader [Lucijan/ebookdownloader-py](https://codeberg.org/Lucijan/ebookdownloader-py),
which lives, read-only, in `original/`.

Stack: Electron Forge + Vite, React 19, shadcn/ui (new-york, JSX), Tailwind v4,
`radix-ui` (unified package), `lucide-react`, `sonner`. Node 24 / Electron 42.

## Commands

```bash
yarn start            # dev: launches the app with HMR (restart for main-process changes)
yarn package          # build an unpackaged app
yarn make             # build installers
npx vite build --config vite.renderer.config.mjs   # fast renderer-only compile check

# Crypto/decryption regression harness (Python reference ↔ Node port):
python verify/gen_vectors.py      && node verify/verify.mjs        # primitives, 9 vectors
python verify/gen_file_vectors.py && node verify/verify_files.mjs  # per-file decryptors, 9
```

The `verify/` Python scripts need `pycryptodome numpy xmltodict pathvalidate`
(`pip install …`). They import the **genuine** functions from `original/` (stubbing
PyQt6/requests) so the vectors are authoritative, not hand-copied.

## Architecture

- **Main process** — all networking and crypto. `src/main.js` registers IPC
  (`ym:*` channels) and builds the path context; `src/main/yomimushi/`:
  - `endpoints.js` — runtime endpoint config. Store-specific identifiers (domains,
    OAuth `clientId`/`clientSecret`, `appId`, protocol header values) are NOT
    hardcoded; they're fetched from a remote JSON (`CONFIG_URL`), cached to
    `<userData>/endpoints.json`, and served via `ensureEndpoints(dataDir)` /
    `getEndpoints()`. Remote-first with disk fallback when offline. `main.js`
    warms it on launch (`initEndpoints`); login/book-list/download call
    `ensureEndpoints` before any network op. See "endpoint config" gotcha.
  - `crypto.js` — primitives: `genCommonKey`, `genEncToken`, salted/xor/rsa/sha512.
  - `login.js` — OAuth + device registration. Has its own cookie jar + manual
    redirect handling (`fetch` has no `requests.Session` equivalent).
  - `book-list.js` — paginated delivery fetch; cache is a per-account JSON file
    (NOT SQLite — avoids a native dep) storing raw API rows.
  - `download.js` — downloads the EPUB payload + OPF/cover + license unit, then
    calls the wrapper. **Always EPUB** (see "always EPUB" gotcha below).
  - `decryption-wrapper.js` — derives content keys from the license unit and
    decrypts every entry; still has book vs comic branches (the comic branch is
    currently dead code — `download.js` only ever passes `comicFlag: false`).
  - `cover-cache.js` — downloads + disk-caches cover art, **downscaling to a 320px
    JPEG thumbnail** via Electron `nativeImage`; served to the renderer through the
    `ym-cover://` protocol registered in `main.js`.
  - `zip.js` — unzip / EPUB repackaging (fflate) / `makeArchive`.
- **Preload** — `src/preload.js` exposes `window.ym` via contextBridge
  (`listAccounts/login/fetchBooks/download/openDownloads` + `onLog/onProgress`).
- **Renderer** — `src/app.jsx` (shell, state, IPC orchestration) +
  `src/components/` (sidebar, library, book-cover, activity-panel, login-dialog)
  + `src/components/ui/` (shadcn). Default theme is dark; accent is violet/fuchsia.

Data flows: renderer → `window.ym.*` (invoke) → `ipcMain.handle` → yomimushi
modules. Progress/log stream back via `webContents.send` → `onLog`/`onProgress`.

## Porting rules

- `original/*.py` is the **source of truth**. When changing decryption logic, mirror
  the Python exactly and re-run the `verify/` harness — the crypto must stay
  byte-exact. Don't "improve" the algorithms.
- Paths: ctx is `{ dataDir, assetsDir, tempDir, downloadsDir }`. The RSA keys ship
  in `assets/yomimushi/` (tracked in *this* repo — `original/` is a nested git repo
  whose contents don't push to GitHub, so keys must not live there). They load from
  `assets/` in dev (`app.getAppPath()/assets`) and `process.resourcesPath` when
  packaged (forge `extraResource: ./assets/yomimushi`); both resolve to
  `<base>/yomimushi/{private.der,public.pem}`.

## Gotchas (learned the hard way — keep these intact)

- **ZIP prepend**: the store prepends ~64 bytes before the real ZIP. Python's
  `zipfile` tolerates it; fflate does not. `stripZipPrepend()` in `zip.js`
  recomputes the offset and slices. Always unzip through `unzip()`.
- **Path sanitizing**: only sanitize the series/book *name* segment
  (`sanitizeSegment`), never the absolute base path — stripping a Windows drive
  colon (`C:`→`C`) silently turns it relative.
- **`requests` vs `fetch` semantics**: Python treats any status `< 400` as ok (so a
  302 with redirects disabled is ok); `fetch` only counts 2xx. `download.js` mirrors
  the Python rule with `isOk()`.
- **base64**: `genEncToken` must keep `=` padding (Python `urlsafe_b64encode`);
  Node's `'base64url'` strips it.
- **`int(bytes)` in Python** parses ASCII decimal — the sha512 deobfuscation keys
  are decimal strings; `asInt()` reproduces this.
- **ESM/CJS**: do not add `"type": "module"` to package.json — `forge.config.js`
  is CommonJS. The Vite configs are already `.mjs`.
- **Always EPUB**: the store delivers everything — including comics — as
  fixed-layout EPUB through the member API. The old comic-viewer ZIP endpoint
  returns **404** and is dead. Upstream shipped this as a hidden, always-on
  "Force EPUB" checkbox; we dropped the toggle entirely and `download.js` always
  takes the EPUB path. Don't resurrect the comic-ZIP download branch unless the
  store revives that endpoint.
- **Cover thumbnails**: covers come straight from the delivery API at full size
  (up to ~4000×3000 / ~5MB). Decoding a 12-megapixel JPEG to paint a 120px tile
  costs ~48MB RAM each and stalls scroll, so `cover-cache.js` downscales to a 320px
  JPEG on cache (and self-heals pre-existing full-size files in place on first read).
- **Endpoint config**: store identifiers (domains, OAuth client id/secret, app id,
  header values) are NOT hardcoded in source — they live in a remote JSON fetched
  by `endpoints.js` (`CONFIG_URL`, hosted on the maintainer's R2) and cached to
  `<userData>/endpoints.json`. Source keeps only `CONFIG_URL`. The cache is the
  offline fallback, so the FIRST run with no network and no cache fails on the
  first login/fetch with a clear error. Adding a new store value? add it to
  `REQUIRED` in `endpoints.js`, the remote JSON, AND its consumer. This is
  *relocation, not secrecy* — the JSON is publicly fetchable; don't treat it as a
  vault.
- **Renderer scroll perf**: the library grid isn't virtualized. Keep `<img>` on
  `loading="lazy" decoding="async"`. Do NOT put `content-visibility:auto` on the
  `BookCover` card — its paint containment clips the selection `ring` and hover
  shadow (they're `box-shadow`, drawn outside the box). The selection ring uses
  `inset-ring-*` so it survives the section-level `content-visibility` in series view.

## Status

EPUB download path verified end-to-end against a real account, and is now the only
download path (comics included). The comic-ZIP endpoint is dead (404); the comic
branch in `decryption-wrapper.js` remains but is unused dead code. Renderer covers
are downscaled to 320px thumbnails for scroll performance.

---
> Source: [meokisama/yomimushi](https://github.com/meokisama/yomimushi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
