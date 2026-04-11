## proyek3-perpustakaan-polban

> Dokumentasi ini memandu developer untuk melakukan setup dan menjalankan aplikasi menggunakan Docker. Proyek ini menggunakan arsitektur kontainerisasi penuh untuk memisahkan concern antara Web Server (Nginx), Aplikasi (Laravel/PHP-FPM), dan Database (PostgreSQL).

# Proyek 3 - Sistem Informasi Perpustakaan Polban

Dokumentasi ini memandu developer untuk melakukan setup dan menjalankan aplikasi menggunakan Docker. Proyek ini menggunakan arsitektur kontainerisasi penuh untuk memisahkan concern antara Web Server (Nginx), Aplikasi (Laravel/PHP-FPM), dan Database (PostgreSQL).

## 🛠 Tech Stack

- **Backend Framework:** Laravel 12
- **Frontend:** Vue.js 3 (via Inertia.js)
- **Styling:** Tailwind CSS
- **Database:** PostgreSQL
- **Web Server:** Nginx (Alpine)
- **Containerization:** Docker & Docker Compose

## 📋 Prasyarat

Pastikan mesin lokal Anda telah terinstal:

- Docker dan Docker Compose (versi terbaru disarankan)
- Git

> **Catatan:** Jangan mencoba menjalankan `php artisan serve` atau `npm run dev` di mesin lokal jika Anda berniat menggunakan Docker sepenuhnya. Kita akan menjalankannya dalam lingkungan yang terisolasi agar konsisten.

## 🚀 Instalasi & Setup (Docker)

Ikuti langkah-langkah ini secara berurutan. Jangan melompat-lompat.

### 1. Clone Repository

```bash
git clone <repository_url>
cd proyek3-perpustakaan-polban
```

### 2. Konfigurasi Environment

Salin file contoh environment:

```bash
cp .env.example .env
```

#### ⚠️ PENTING: Konfigurasi Database untuk Docker

Secara default, `.env.example` dikonfigurasi untuk pengembangan lokal (host). Agar aplikasi di dalam container bisa berkomunikasi dengan container database, Anda wajib mengubah `DB_HOST`.

Buka file `.env` dan sesuaikan variabel berikut:

```ini
# Ganti localhost menjadi 'db' (nama service di docker-compose.yml)
DB_CONNECTION=pgsql
DB_HOST=db
DB_PORT=5432
DB_DATABASE=perpustakaan_db
DB_USERNAME=postgres
DB_PASSWORD=postgres

# Sesuaikan URL aplikasi agar asset loading (Vite) berjalan benar
APP_URL=http://localhost
```

### 3. Jalankan Container

Jalankan perintah berikut untuk membangun image dan menyalakan container di background:

```bash
docker-compose up -d --build
```

### 4. Proses Automatisasi (Entrypoint)

Anda tidak perlu menjalankan `composer install`, `npm install`, atau `php artisan migrate` secara manual. Script `scripts/php-fpm-entrypoint` telah dikonfigurasi untuk menangani hal berikut secara otomatis saat container pertama kali berjalan:

- **Permissions:** Memperbaiki permission folder `storage` dan `bootstrap/cache`
- **Dependencies:** Menjalankan `npm ci` dan `npm run build` untuk meng-compile aset frontend (Vite)
- **Database:** Menunggu service database siap, lalu menjalankan migrasi (`php artisan migrate`)
- **Optimization:** Membersihkan dan melakukan cache config

⏳ Tunggu beberapa saat (3-5 menit tergantung kecepatan internet) pada run pertama kali agar proses build aset dan instalasi dependensi selesai. Anda bisa memantau progress dengan:

```bash
docker-compose logs -f app
```

## 🌐 Akses Aplikasi

Setelah container berjalan stabil (tidak ada error di logs), buka browser:

- **URL Utama:** http://localhost

## 🔧 Perintah Pengembangan (Development Cheatsheet)

Karena aplikasi berjalan dalam container, Anda harus mengeksekusi perintah artisan atau npm di dalam container `app`, bukan di terminal host Anda.

### Masuk ke Shell Container

```bash
docker-compose exec app bash
```

### Menjalankan Perintah Artisan

Contoh membuat controller baru tanpa masuk ke shell:

```bash
docker-compose exec app php artisan make:controller ArticleController
```

### Rebuild Frontend Assets

Jika Anda mengubah file `.vue` atau `.css`, Anda perlu me-rebuild aset (karena kita tidak mengekspos port dev server Vite di konfigurasi `docker-compose.yml` saat ini):

```bash
docker-compose exec app npm run build
```

### Reset Database

Jika ingin mereset database sepenuhnya:

```bash
docker-compose exec app php artisan migrate:fresh --seed
```

## 🐛 Troubleshooting Umum

### 1. Error "Connection Refused" pada Database

Pastikan Anda sudah mengubah `DB_HOST=db` di file `.env`. Container aplikasi tidak mengenal `localhost` sebagai database, karena `localhost` bagi container adalah dirinya sendiri.

### 2. Halaman Putih / 500 Server Error

Cek permission folder `storage`:

```bash
docker-compose exec app chmod -R 775 storage
```

Pastikan `APP_KEY` sudah digenerate. Jika kosong di `.env`, jalankan:

```bash
docker-compose exec app php artisan key:generate
docker-compose restart app
```

### 3. Aset (CSS/JS) Tidak Berubah

Karena Dockerfile ini menjalankan `npm run build` (mode produksi), Hot Module Replacement (HMR) tidak aktif. Setiap perubahan kode frontend memerlukan perintah `npm run build` ulang di dalam container.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MaulIshak)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MaulIshak)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
