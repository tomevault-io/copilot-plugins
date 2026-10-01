## setup-gateway

> Konfigurator **generik** untuk gateway API OpenAI-compatible pada **Claude Code**,

# CLAUDE.md — setup-gateway

Konfigurator **generik** untuk gateway API OpenAI-compatible pada **Claude Code**,
**OpenCode**, dan **Codex CLI**.
Versi netral dari `@genflowai/autosetup`: tidak ada provider/base URL yang di-hardcode —
user memasukkan **base URL + API key + model** sendiri tiap setup.

## Fakta cepat

- **Bahasa/stack:** pure JavaScript ESM (`.mjs`), **zero-dependency**, Node ≥ 18.
- **Struktur:** entry tipis `bin/setup-gateway.mjs` + logika di `src/*.js`.
  Shim `setup-gateway.mjs` di root hanya `import "./bin/setup-gateway.mjs"`
  (agar perintah lama `node setup-gateway.mjs` tetap berfungsi).
- **README** berisi docs lengkap (quick start, flag, env, backup/restore, troubleshooting).

## Perintah umum

```bash
node bin/setup-gateway.mjs                # menu interaktif (pilih tool)
node bin/setup-gateway.mjs claude         # setup Claude Code langsung
node bin/setup-gateway.mjs opencode       # setup OpenCode langsung
node bin/setup-gateway.mjs codex          # setup Codex CLI langsung
node bin/setup-gateway.mjs <tool> --base-url <url> --api-key <key> --model <id> --yes
node bin/setup-gateway.mjs status         # read-only: endpoint/model/key termask
node bin/setup-gateway.mjs restore [tool] # kembalikan backup terakhir
node bin/setup-gateway.mjs --selftest     # uji fungsi murni tanpa disk riil
node bin/setup-gateway.mjs --help
```

Via npm: `npm run setup` / `npm run selftest` / `npm run help`.

## Flag & env

| Flag | Efek |
|---|---|
| `--base-url`, `--api-key`, `--model` | isi langsung (lewati prompt) |
| `--provider-name <nama>` | nama blok provider (default `gateway`) |
| `--yes` | lewati menu + konfirmasi http:// (wajib utk CI) |
| `--skip-backup` | jangan backup sebelum tulis |
| `--config-dir` / `--opencode-config-dir` / `--codex-config-dir` | override dir config (konflik dgn env = error) |
| `--status` / `--restore` | alias subcommand |
| `--selftest` | verifikasi fungsi murni, exit 0/1 |

**Env:** `GATEWAY_BASE_URL`, `GATEWAY_API_KEY`, `GATEWAY_MODEL`.
Precedence nilai: **flag → env → prompt (TTY) → error non-interaktif** (tanpa default hardcoded).

## File config yang ditarget

| Tool | File default (wajib dipahami) |
|---|---|
| Claude Code | `CLAUDE_CONFIG_DIR` atau `~/.claude/settings.json` |
| OpenCode | `OPENCODE_CONFIG_DIR` atau `~/.config/opencode/opencode.json` |
| Codex CLI | `CODEX_HOME` atau `~/.codex/config.toml` (**TOML**, bukan JSON) |
| Approval | `claudeJson path = ~/.claude.json` (`customApiKeyResponses.approved`, `hasCompletedOnboarding`) |

⚠️ JANGAN merusak / menimpa file config **riil** pengguna saat bantu debugging.
Gunakan `--config-dir <tmp>` + `--opencode-config-dir <tmp>` + `--codex-config-dir <tmp>` untuk uji aman.

## Aturan preservasi (kritis)

- **Claude:** spread semua key top-level lama; ubah HANYA `env.ANTHROPIC_BASE_URL` +
  `env.ANTHROPIC_API_KEY`; pertahankan env lain (`ANTHROPIC_AUTH_TOKEN`,
  `ANTHROPIC_DEFAULT_*_MODEL`, dst); hapus `ANTHROPIC_CUSTOM_MODEL_OPTION*` lalu
  set ulang hanya untuk model non-`claude`; buang `availableModels`/`modelOverrides`.
- **OpenCode:** spread semua key lama; tambah/ganti blok `provider[<nama>]`;
  provider baru → `model = <nama>/<model>`; provider lama → migrasi `model` dan
  `agent.*.model` yang menunjuk ref lama. Blok `agent` tidak pernah dihapus.
- **Codex:** upsert string TOML (bukan parse→serialize) lewat `src/toml.js` agar
  komentar & key tak dikenal terjaga; tambah/ganti blok
  `[model_providers.<nama>]` + kunci top-level `model`/`model_provider`;
  `wire_api = "responses"` wajib; key ditulis via `experimental_bearer_token`;
  id provider reserved (`openai`/`ollama`/`lmstudio`) ditolak. Backup & restore
  raw bytes (tidak parse ulang). `restore` menulis string apa adanya.
  **Catalog `/model`:** bila `catalogPath` ada → upsert top-level
  `model_catalog_json`; file `gateway-catalog.json` di-dir config di-generate
  dari `buildCodexCatalog` (`src/catalog.js`) — `slug`=`display_name`=model id,
  `visibility:"list"` WAJIB (tanpa itu fallback metadata Codex `visibility:None`
  → tidak muncul di picker). `fmtValue` & `indexTomlKeys` handle escape backslash
  utk path Windows.
- `normalizeBaseUrl`: trim, buang trailing `/`, pastikan akhiran `/v1`.

## Arsitektur file

| Berkas | Isi |
|---|---|
| `bin/setup-gateway.mjs` | parse → dispatch; gate `--selftest`; picker tools |
| `src/constants.js` | `VERSION`, `TOOLS`, `SCHEMA_URLS`, `POSITIONAL_TOOLS`, `CODEX_RESERVED_PROVIDERS`, regex |
| `src/errors.js` | `ConfigError`, `ModelsError` |
| `src/term.js` | stdin/stdout, ANSI, `emitInputEvents()` (raw-mode), `maskKey` |
| `src/ui.js` | banner/step/ok/warn/err, spinner, box, `ask()`; re-export warna |
| `src/menu.js` | `selectMenu`, `filterMenu`, `multiSelectMenu` (raw-mode keypress) |
| `src/cli.js` | `parseArgs` (flag/env/precedence) |
| `src/config.js` | `stripJsonComments`, `normalizeBaseUrl`, `modelsUrlOf` |
| `src/toml.js` | `indexTomlKeys` (read-only), `upsertToml` (string-preserving) utk Codex |
| `src/config-io.js` | baca/tulis/backup atomik, `listBackups`, `overrideDir`; codex = raw string |
| `src/models.js` | fetch `/v1/models`, `validateModelId`, capability picker |
| `src/catalog.js` | `buildCodexCatalog()` (catalog `/model` picker Codex), `modelEntry`, `catalogFileName` |
| `src/builders.js` | `buildClaudeConfig`, `buildOpenCodeConfig`, `buildCodexConfig` |
| `src/approve.js` | auto-approve key di `~/.claude.json` |
| `src/workflow.js` | resolve input + `runToolSetup` per tool |
| `src/commands.js` | `cmdStatus`, `cmdRestore`, `cmdHelp` |
| `src/selftest.js` | `runSelfTest()` (36 assertions fungsi murni) |

## Bug yang pernah terjadi (penting)

**Hang setelah Enter menu → prompt Base URL tidak menerima input.**
Akar: cleanup `emitInputEvents()` dulu melakukan `removeAllListeners("keypress")` +
`input.pause()` + `input.unref()`, yang mematikan stdin utk prompt berikutnya.
Fix (sudah diterapkan): cleanup HANYA `setRawMode(false)` + `resume()` + show cursor.
Jangan pernah reintroduce `removeAllListeners`/`pause`/`unref` di cleanup menu/ask.

Kedua: TDZ — `const finish` yang merefer `const handler` yang dideklarasi setelahnya.
Pola aman: `let handler; const finish=...; handler=(...)=>{...}`.

## Cara UJI (penting — ulangi sebelum menyimpulkan "berhasil")

Windows **tidak punya pty/termios**, jadi jalur TTY interaktif **tidak bisa**
di-uji otomatis. Gunakan:

1. `node bin/setup-gateway.mjs --selftest` → harus semua `[ok]`, exit 0.
2. Fake gateway (`http` inline) + temp dir:
   ```bash
   node -e "require('http').createServer((q,r)=>{r.setHeader('content-type','application/json');r.end(JSON.stringify({data:[{id:'deepseek-v4'},{id:'claude-sonnet-4-5'}]}))}).listen(20128,'127.0.0.1')" &
   node bin/setup-gateway.mjs claude --config-dir <tmp> --base-url http://127.0.0.1:20128 --yes --api-key sk-test --model deepseek-v4
   node bin/setup-gateway.mjs codex --codex-config-dir <tmp> --base-url http://127.0.0.1:20128 --yes --api-key sk-test --model deepseek-v4
   ```
   Verifikasi: `settings.json` terulis benar, backup dibuat, approval idempotent di
   `~/.claude.json` (suffix dummy `abcd1234`), preservasi env, opencode baru + migrasi
   provider 9router, **codex `config.toml` terulis valid** (kunci top-level sebelum
   header tabel, `wire_api = "responses"`, blok provider tidak duplikat, komentar
   terjaga, idempoten, reserved `openai` ditolak), status read-only (mtime tak
   berubah), restore round-trip (raw bytes untuk toml), CI tanpa nilai → exit 1.
3. Jangan pernah gunakan config riil pengguna sebagai fixture uji.
4. **Catalog Codex** (`--codex-config-dir <tmp>` saja; jangan sentuh `~/.codex`):
   - non-TTY/`--yes` → `config.toml` punya `model_catalog_json`, `gateway-catalog.json`
     valid JSON berisi model utama (slug=display_name, `visibility:"list"`), path
     Windows backslash ke-escape benar di TOML, `status` menampilkan baris Catalog,
     re-run idempoten (config + catalog sama).
   - `buildCodexCatalog(["m1","m2","m1"])` → 2 entry; id invalid di-skip; tanpa model
     → null (catalog dilewati). Full matrix di `src/selftest.js`.
   - **Verifikasi BINARY RIIL (wajib setelah ubah format catalog):** `codex debug models`
     (read-only, OK terhadap config riil; untuk test temp pakai `CODEX_HOME=<tmp>`).
     Catalog harus di-load tanpa "missing field ..." / "failed to parse".
   - **Field Wajib ModelInfo (Codex parse STRICT):** `slug`, `display_name`,
     `supported_reasoning_levels`, `shell_type`, `visibility`, `supported_in_api`,
     `priority`, `support_verbosity`, `truncation_policy`, `supports_parallel_tool_calls`,
     `experimental_supported_tools`, + `base_instructions` (atau
     `model_messages.instructions_template`). Field lain boleh dihilangkan (serde default).
     **Modality:** `input_modalities` hanya `text|image|audio` — `video` TIDAK dikenal
     (error `unknown variant video`); filter dari `createDefaultModelInfo` di
     `src/catalog.js` (`CODEX_INPUT_MODALITIES`).

## Kontribusi singkat

- Jaga zero-dependency; jangan tambah package tanpa confirm.
- Fungsionalitas baru: ikuti pola module `src/permintaan.js` → export fungsi murni,
  `bin/` hanya dispatch.
- Perubahan belum diuji (selftest + fake-gateway min.) → jangan klaim selesai.
</parameter>
<parameter name="file_path">E:\CODING GABUT\setup-gateway\CLAUDE.md</parameter>
</invoke>

---
> Source: [Sinholms/setup-gateway](https://github.com/Sinholms/setup-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
