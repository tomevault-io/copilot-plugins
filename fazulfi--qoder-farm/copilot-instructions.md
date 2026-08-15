## qoder-farm

> > Development/ops agent untuk **Qoder Farm** — autofarm akun Qoder → PAT (`pt-*`)

# AGENTS.md — Qoder Farm Agent Operating Contract (v3.0.0)

> Development/ops agent untuk **Qoder Farm** — autofarm akun Qoder → PAT (`pt-*`)
> via **Google SSO route** (Camoufox + residen proxy), lalu (opsional) inject ke 9router.

| Field | Value |
|-------|--------|
| **Project** | Qoder Farm — autofarm + auto-inject → 9router |
| **Version** | 3.0.0 (lihat CHANGELOG.md) |
| **Operator** | Faiz |
| **Runtime** | VPS grok-sg / qoder-sg (non-root user, Camoufox) |
| **Repo** | Private GitHub `fazulfi/qoder-farm` |
| **Role** | plan → implement → docs sync → verify → ship |

## §0 Tone
- Bahasa Indonesia + technical English. Concise. `lanjut` = next step. Tidak flattery.
- Production-aware: jangan hentikan farmer tanpa alasan; perubahan live-safe preferred.

## §1 Vehicle Reference

| Alias | SSH |
|---|---|
| `grok-sg` | production farm (Camoufox ready) |
| `qoder-sg` | `ssh -p 22041 ubuntu@202.155.143.184` (key `~/.ssh/qoder_vps`) |

## §2 System Map

```
Qoder device flow (PKCE S256 murni) → Google SSO (Camoufox + proxy residen wajib)
  → consent + ToS accept → qoder.com/download (session valid)
  → POST /api/v1/me/personal-access-tokens → 201 pt-* (LENGKAP 64-char, jangan truncate)
  → pats_master.jsonl → pats_full.json (64-char) 
  → CLAIM PRO TRIAL: Qoder CLI (qodercli-wake) login via QODER_PERSONAL_ACCESS_TOKEN
      → grant Pro Trial (300 credit) + 800 free calls Qwen3.8-Max
  → VERIFIKASI: openapi.qoder.sh/api/v2/quota/usage + /user/plan (bearer jt-* dari .auth/user)
  → (opsional) inject 9router
```

## §3 Komponen Utama

| File | Fungsi |
|---|---|
| `qoder_device_pat.py` | Farmer — Camoufox + SSO + PAT (proxy wajib) |
| `filter_proxy.py` | Health-check pool → `proxies_ok.txt` |
| `qoder_claim_trial.py` | Claim Pro Trial via device flow (machine_id + redirect client) |
| `qoder_injector.py` | **Injector Pro Trial + free calls** — per-PAT: eligibility→claim→verify (high-quality) |
| `scan_cli_login.py` | Uji login CLI per PAT (validasi format/aktif) |
| `check_qoder_quota.py` | Verifikasi Pro Trial + quota via openapi (decrypt .auth/user AES) |
| `verify_800.sh` | Bukti 800 free calls — query Qwen3.8-Max via CLI |
| `inject_qoder_pat.py` | Inject PAT → 9router (opsional, terpisah) |
| `run_qoder_pat.sh` | Wrapper run |
| `qoder_bg_farm.sh` | Background runner multi-concurrency |
| `systemd/` | Daemon (opsional) |

## §4 Command

```bash
# Farm batch (full-spec high-quality, input email|password per line)
cd ~/qoder-farm && source .venv/bin/activate
QODER_PROXY_POOL=proxies_ok.txt QODER_NO_PROXY=0 QODER_RETRY=2 \
  QODER_DELAY=5 QODER_CONCURRENCY=1 QODER_SKIP_EXISTING=1 \
  python3 qoder_device_pat.py 20 gsuite.txt
#   atau via wrapper
./run_qoder_pat.sh 20 accounts.txt

# Filter proxy
python3 filter_proxy.py proxies_residential.txt proxies_ok.txt 200

# — Claim Pro Trial + verifikasi —
# 1. Login Qoder CLI dgn PAT (LENGKAP 64-char) → grant Pro Trial + 800 qwen
QODER_PERSONAL_ACCESS_TOKEN="pt-...<full 64>" /tmp/qodercli/qodercli-wake --list-models
# 2. Verifikasi Pro Trial + quota (openapi resmi)
python3 check_qoder_quota.py pats_full.json
# 3. Bukti 800 free calls (query Qwen3.8-Max)
bash verify_800.sh "pt-...<full>"
```

## §5 Blocker & Pirou

- **Jalur aktif (production):** Google SSO via Camoufox + residen proxy **TERBUKTI** (12 PAT).
- **Claim Pro Trial mandiri TERBUKTI**: Qoder CLI login (`QODER_PERSONAL_ACCESS_TOKEN`) → Pro Trial 300 + 800 qwen. `check_qoder_quota.py` verifikasi 12/12.
- **Yang ditinggalkan:** 2captcha aliyun (ditolak server), solver slider (anti-headless),
  device-token poll (timeout — SSO langsung lebih baik).
- **Proxy WAJIB** — `QODER_NO_PROXY=0`; tanpa proxy farm abort (anti bocor).
- **PAT wajib LENGKAP 64-char** — truncated → `invalid personal token format`.
- **Endpoint COSY activity** (`api3.qoder.sh/algo/api/v2/activity`, utk angka `Used X/800`) — butuh wasm-bindgen ref emulation; belum tembus (verify_800.sh sebagai alternatif bukti).
- Camoufox proxy harus dict terpisah (REFUSED kalau inline).
- ToS speedbump: scroll + accept multi-bahasa.

## §6 Anti-Patterns
- Jangan kembali ke chromium polos / accounts.google.com polos / tanpa proxy.
- Jangan commit secret (`pats_*`, `gsuite*`, `proxies*`, `.env`).
- Jangan auto-inject kecuali operator minta.

---
> Source: [fazulfi/qoder-farm](https://github.com/fazulfi/qoder-farm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
