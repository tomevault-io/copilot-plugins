## autoscript

> `run.sh` adalah launcher installer, `setup.sh` menangani provisioning VPS, dan `manage.sh` adalah CLI operasional setelah instalasi. Simpan logic installer di `opt/setup/`, logic CLI runtime di `opt/manage/`, kode bot Telegram di `bot-telegram/`, source edge gateway di `opt/edge/go/`, dan utilitas tambahan di `tools/`.

# Repository Guidelines

## Struktur Proyek dan Organisasi Modul

`run.sh` adalah launcher installer, `setup.sh` menangani provisioning VPS, dan `manage.sh` adalah CLI operasional setelah instalasi. Simpan logic installer di `opt/setup/`, logic CLI runtime di `opt/manage/`, kode bot Telegram di `bot-telegram/`, source edge gateway di `opt/edge/go/`, dan utilitas tambahan di `tools/`.

Modul shell dipisah per domain:
- `opt/setup/install/*.sh`: langkah provisioning seperti `xray.sh` dan `sshws.sh`
- `opt/manage/features/*` dan subfoldernya: action menu dan operasi runtime
- `opt/setup/templates/` dan `opt/setup/bin/`: template config, unit systemd, dan helper script

## Perintah Build, Test, dan Pengembangan

Gunakan source repo lokal saat menguji perubahan installer:

```bash
RUN_USE_LOCAL_SOURCE=1 bash run.sh
bash -n setup.sh opt/setup/core/*.sh opt/setup/install/*.sh
shellcheck -x -S warning setup.sh opt/setup/core/*.sh opt/setup/install/*.sh
python3 -m py_compile opt/setup/bin/sshws-qac-enforcer.py opt/setup/bin/xray-speed.py
go -C opt/edge/go test ./...
bash bot-telegram/scripts/gate-all.sh
sudo /opt/bot-telegram/scripts/smoke-test.sh
```

`gate-all.sh` memvalidasi kode Python bot. Test Go berada di `opt/edge/go/**` dan `opt/adblock/go/**`.

## Gaya Kode dan Konvensi Penamaan

Ikuti idiom Bash, Python, dan Go sesuai area masing-masing. File shell memakai `set -euo pipefail`, indentasi 2 spasi, nama fungsi `snake_case` huruf kecil, dan anotasi `shellcheck` jika memang dibutuhkan. Jaga file aggregator tingkat atas tetap tipis; tambahkan logic baru ke child module yang relevan, bukan menumpuk lagi di `manage.sh` atau `setup.sh`.

Python mengikuti penamaan standar PEP 8. Paket Go tetap kecil dan fokus di bawah `internal/` atau `cmd/`. Utamakan edit ASCII kecuali file target memang sudah memakai karakter non-ASCII.

## Panduan Testing

Tambahkan test dekat dengan implementasinya:
- Go: `*_test.go`
- Python: validasi minimal lewat `py_compile` dan script gate/smoke yang sudah ada
- Shell: cek sintaks dengan `bash -n` dan lint dengan `shellcheck`

Untuk perubahan menu atau flow yang terlihat user, verifikasi baik jalur repo lokal maupun perilaku runtime yang sudah terpasang.

## Panduan Commit dan Pull Request

Riwayat commit repo ini memakai subjek singkat berbentuk imperatif, misalnya `Fix SSHWS runtime sync and metadata drift` atau `Harden Telegram bot sensitive actions`. Ikuti pola itu: satu baris singkat, diawali aksi, dan jelas area perubahannya.

PR harus menyertakan ringkasan perubahan, area yang terdampak, perintah test manual yang dijalankan, dan screenshot hanya jika output menu/Telegram berubah secara material.

## Catatan Keamanan dan Konfigurasi

Pertahankan asumsi trusted-path dan eksekusi root-only saat mengubah installer atau runtime script. Kredensial Telegram dan backup harus tetap dikelola lewat environment atau file runtime yang sesuai.

Guardrail khusus proyek:
- Jangan hapus default token Cloudflare hardcoded yang sudah ada di entrypoint runtime atau installer kecuali maintainer meminta secara eksplisit.
- Jangan hapus flow cleanup DNS A record yang tetap berjalan walau IP VPS tidak berubah; perilaku ini harus tetap dipertahankan saat refactor domain flow.
- Jangan menambahkan atau mewajibkan verifikasi checksum SHA256 pada flow installer, release bundle, atau archive bot kecuali maintainer memintanya secara eksplisit.

---
> Source: [superdecrypt-dev/autoscript](https://github.com/superdecrypt-dev/autoscript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
