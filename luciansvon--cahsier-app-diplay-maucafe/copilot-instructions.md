## cahsier-app-diplay-maucafe

> Urutan acuan saat bekerja:

# AGENTS.md

## Sumber kebenaran

Urutan acuan saat bekerja:

1. behavior server dan source code saat ini;
2. automated tests;
3. README dan dokumentasi terbaru;
4. requirement eksplisit user.

Jangan menebak requirement besar. Jika ada konflik dokumen lama dengan source/test terbaru, verifikasi runtime sebelum mengubah behavior.

## Prinsip kerja

- Reproduce masalah sebelum memperbaiki.
- Cari root cause, jangan patch gejala saja.
- Buat perubahan terkecil yang benar.
- Jangan rewrite framework/arsitektur tanpa kebutuhan terverifikasi.
- Jangan menghapus data atau fitur user tanpa instruksi eksplisit.
- Setelah setiap perubahan penting: jalankan test, build, lalu smoke test flow terkait.
- Jangan melemahkan test supaya perubahan terlihat lulus.

## Role dan authorization

### Public / Display

Hanya boleh menerima data yang diperlukan TV/customer:

- outlet info publik;
- produk publik tanpa HPP;
- active call;
- promo media;
- revision/freshness metadata.

Dilarang mengekspos order history, payment method, HPP/unitCost, credential, atau laporan internal.

### Admin / Kasir

Boleh untuk outlet session-nya sendiri:

- create order;
- call/recall;
- complete;
- melihat antrean aktif;
- mengelola media outlet sesuai behavior saat ini.

Kasir tidak boleh CRUD produk/HPP atau membaca laporan Owner.

Cancel membutuhkan session Admin/Owner dan approval Owner. Jangan menghapus histori cancelled order.

### Owner

Boleh:

- semua laporan/outlet state internal;
- CRUD produk/HPP;
- PIN Admin per outlet;
- PIN Owner global;
- operasi reset/purge/clear;
- media dan pengaturan sensitif.

Authorization wajib dilakukan server-side. Menyembunyikan tombol frontend bukan security.

## Credential

- Jangan menyimpan PIN plaintext.
- `data/outlets.json` hanya menyimpan `adminPinHash`.
- `data/security.json` menyimpan `ownerPinHash` global.
- Jangan menambahkan fallback credential production.
- Jika credential Owner hilang/rusak, startup harus fail closed.
- Jangan pernah mengirim hash credential ke browser.

## Order dan laporan

Server adalah source of truth untuk harga, HPP snapshot, total, status, dan queue number.

Client hanya mengirim product ID + quantity + payment method untuk order.

Snapshot order item minimal harus mempertahankan:

- productId;
- productName;
- category;
- unitPrice;
- unitCost;
- quantity;
- subtotal.

Laporan historis tidak boleh berubah ketika produk saat ini diedit.

Definisi finansial jangan dicampur:

- Penjualan Bersih = subtotal transaksi dibayar;
- Total Diterima = penjualan bersih;
- payment totals = uang yang benar-benar diterima per metode.

## Queue/display

- `nextCallEventId` harus monotonik dan tidak ikut reset queue.
- Migrasi state legacy harus memastikan event baru lebih besar dari event lama yang mungkin tersimpan di localStorage TV.
- Display stale tidak boleh menampilkan nomor lama seolah masih valid.
- Nomor queue boleh reset sesuai business-day/reset policy, event ID suara tidak.

## UI

Kasir:

- desktop/tablet: produk + cart berdampingan;
- mobile: sticky cart summary tanpa menutupi content;
- menu punya search/category filter;
- Ready dan Waiting dipisah;
- Waiting oldest-first;
- destructive action tidak boleh se-prominent primary action.

Owner:

- financial labels eksplisit;
- tab responsif;
- danger zone terpisah;
- pengaturan sensitif hanya aktif untuk outlet spesifik jika memang outlet-scoped.

Display:

- pertahankan fokus nomor antrean dan promo;
- jangan menambah dekorasi yang mengurangi keterbacaan jarak jauh.

## Security coding rules

- Hindari `innerHTML` untuk dynamic/user-controlled data.
- Validate input server-side.
- Gunakan safe integer/range untuk price, cost, quantity.
- Upload: auth, size, magic/signature validation, generated filename, quota/rate control, cleanup.
- Jangan percaya `X-Forwarded-For` tanpa trusted-proxy configuration.
- Jangan log PIN, session token, atau credential hash.

## Verification wajib

Sebelum menyatakan selesai:

```bash
npm test
npm run build
```

Lalu smoke test flow yang diubah.

Definition of done:

- root cause jelas;
- fix ada;
- regression test ditambah/diubah jika relevan;
- seluruh test pass;
- build pass;
- tidak ada syntax error baru;
- tidak ada `innerHTML` dynamic baru;
- tidak ada plaintext credential baru;
- dokumentasi yang terdampak diperbarui.

Dokumentasi bug wajib:

- Setiap bugfix wajib menambah atau memperbarui `docs/ERROR_SOLUTIONS.md`.
- Catatan bug minimal memuat kondisi/gejala, root cause, solusi, dan bukti verifikasi aktual.
- Dilarang menyatakan bugfix selesai jika catatan tersebut belum dibuat.
- Sebelum final response bugfix, wajib periksa `git diff -- docs/ERROR_SOLUTIONS.md`.

## Batas arsitektur

Jangan over-engineer. Saat ini sistem sengaja satu Node server + JSON store.

Migrasi ke database/session store persisten dilakukan jika deployment membutuhkan multi-instance, durability lebih tinggi, audit penuh, atau concurrency lebih besar. Jangan menambah microservice/Redis/Kafka hanya demi terlihat modern.

---
> Source: [Luciansvon/cahsier-app-diplay-maucafe](https://github.com/Luciansvon/cahsier-app-diplay-maucafe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
