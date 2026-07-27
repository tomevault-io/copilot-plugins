## kitkatprotansa

> Sistem ini memiliki sistem rilis cerdas yang mendeteksi perubahan antara Hot Release (OTA Patch) dan Full Release (Installer).

# Aturan Khusus Proyek Prolanis Tansa

Sistem ini memiliki sistem rilis cerdas yang mendeteksi perubahan antara Hot Release (OTA Patch) dan Full Release (Installer).

## KODE PEMICU (TRIGGER PHRASES)

Setiap kali User mengetikkan salah satu dari kode pemicu berikut, Anda **WAJIB** merespon dengan menjalankan perintah terminal yang sesuai tanpa bertanya lagi.

### 1. "cek persiapan update release"
**Aksi yang dilakukan:**
Jalankan perintah ini di terminal:
`npm run release -- --check-only`
Tunggu perintah selesai, lalu bacakan outputnya kepada User (sebutkan versi yang akan dibuat dan tipe rilis yang dipilih oleh sistem, apakah Hot Release atau Full Release).

### 2. "kerjakan update release" (tanpa spesifikasi versi)
**Aksi yang dilakukan:**
Jalankan perintah rilis utama yang akan otomatis menentukan nama versi:
`npm run release`
Tunggu proses build dan packaging selesai, lalu beritahu User bahwa rilis telah sukses dilakukan.

### 3. "kerjakan update release versi [X.X.X]"
**Aksi yang dilakukan:**
Jika User menyebutkan versi spesifik dalam perintahnya (misalnya "kerjakan update release versi 2.0.0"), Anda wajib mengekstrak nomor versinya dan menjalankannya dengan parameter:
`npm run release -- --version=[X.X.X]`
Tunggu proses selesai dan berikan konfirmasi.

## ATURAN PENGEMBANGAN LAUNCHER (.EXE)

Jika Anda (AI) menganalisis dan merasa bahwa update/kebutuhan proyek mengharuskan adanya perubahan pada sistem instalasi atau file di dalam folder `launcher`, Anda berhak melakukannya dengan 3 syarat wajib:
1. **Hanya ubah isi dari folder `launcher/`** (seperti `Launcher.cs` atau file terkait di dalamnya).
2. **Jangan mengubah alur dasar instalasi.** Semua proses instalasi, animasi progress bar, loading screen, dan cara Launcher bekerja harus tetap mengikuti pola yang sudah ada (tampilan borderless, tema gelap, otomatis ekstrak dan run).
3. Anda bebas memperbaiki bug atau meningkatkan logika di dalamnya, asalkan tetap mematuhi pola desain UI/UX bawaannya.

---
> Source: [m4doen948788/KitKatProTansa](https://github.com/m4doen948788/KitKatProTansa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
