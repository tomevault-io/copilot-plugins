## rental-baju

> - Mendefinisikan sistem desain UI/UX utama untuk aplikasi rental software berbasis gaya Flat Modern Minimalist dengan sentuhan soft neumorphism dan aksen gold/yellow.

# UI/UX Design System – Rental Software (v2)

## Rules & Instruksi Utama

**Tugas:**

- Mendefinisikan sistem desain UI/UX utama untuk aplikasi rental software berbasis gaya Flat Modern Minimalist dengan sentuhan soft neumorphism dan aksen gold/yellow.
- Menjadi acuan utama seluruh tim dalam pengembangan, review, dan evaluasi UI/UX.
- Menjamin konsistensi, aksesibilitas, dan kemudahan scaling di seluruh aplikasi.

**Instruksi Penggunaan:**

- first of All selalu gunakan compoentn SHadcn UI terlebih dahulu, kemudian custom sesuai dengan kebutuhan.
- Selalu rujuk dokumen ini sebelum membuat, mengubah, atau mereview komponen UI/UX.
- Semua token warna, radius, shadow, dan font WAJIB didefinisikan di Tailwind config.
- Gunakan Shadcn UI sebagai basis, custom hanya jika tidak tersedia.
- Lakukan review berkala dan dokumentasikan perubahan besar di dokumen ini.

---

## 1. **Gaya Visual**

**Flat Modern Minimalist dengan Sentuhan Soft Neumorphism**

- **Flat Design:**
  - Hampir seluruh elemen menggunakan warna solid tanpa efek 3D, emboss, atau glassmorphism.
  - Tidak ada shadow berat, hanya shadow sangat halus untuk depth minimal.
- **Minimalist:**
  - Banyak whitespace, layout sangat bersih, tidak ada elemen dekoratif berlebihan.
  - Informasi disajikan secara ringkas dan jelas.
- **Soft Neumorphism Touch:**
  - Beberapa card memiliki sudut rounded yang sangat lembut, dan ada sedikit efek inner/outer shadow yang sangat subtle, memberi kesan “floating” tanpa berat.
- **Modern UI:**
  - Komponen card, button, dan badge tampil dengan bentuk rounded dan outline tipis.
  - Ikon-ikon minimalis, micro-interaction implied (misal: tombol dengan lingkaran solid, dropdown dengan arrow sederhana).

**Ciri Khas Visual:**

- Sudut membulat (rounded corners) pada semua card dan komponen.
- Grid modular, setiap informasi dalam card terpisah.
- Visual hierarchy sangat jelas: judul besar, subjudul kecil, data/statistik menonjol.
- Penggunaan icon dan avatar untuk memperkuat identitas dan affordance.

---

## B. Palet Warna

| Warna         | Hex Contoh      | Penggunaan Utama                |
| ------------- | --------------- | ------------------------------- |
| Gold          | #FFD700         | Highlight, CTA, progress, badge |
| Putih         | #FFFFFF         | Background utama, card          |
| Abu-abu muda  | #F8FAFC         | Background sekunder, card       |
| Abu kehijauan | #E2E8F0         | Background grid, panel          |
| Hitam         | #111827         | Teks utama, angka, tombol       |
| Biru/Olive    | #3B82F6/#84CC16 | Status sekunder, badge          |

**Palet Warna:**

- **Dominan:**
  - Gold (#FFD700 atau sejenis) sebagai warna aksen utama (highlight, progress, CTA).
  - Putih dan abu-abu muda untuk background card dan area utama.
  - Abu-abu kehijauan sebagai background utama (bukan putih polos, memberi nuansa modern dan tenang).
- **Kontras:**
  - Hitam untuk teks utama dan elemen penting (judul, angka, tombol utama).
  - Aksen hitam solid pada tombol utama dan beberapa elemen interaktif.
- **Hierarki Visual:**
  - Warna aksen digunakan untuk menyorot status, progress, dan elemen penting (misal: tanggal terpilih, progress bar, AI Assistant).
  - Warna netral untuk background agar konten mudah dibaca.

**Tipografi:**

- **Jenis Huruf:**
  - Sans-serif modern, kemungkinan besar menggunakan font seperti Inter, Poppins, atau Graphik.
- **Ukuran & Konsistensi:**
  - Judul besar, tebal, sangat readable.
  - Subjudul dan label lebih kecil, tipis, dan konsisten.
  - Angka/statistik sangat menonjol (besar, bold).
- **Konsistensi:**
  - Semua teks rata kiri, spasi antar elemen sangat rapi.
  - Tidak ada variasi font yang berlebihan, menjaga kesan profesional.

---

## C. Usage Guidelines

- **Primary:** Gold untuk tombol utama, highlight, dan progress.
- **Secondary:** Biru/olive untuk status sekunder, badge, atau notifikasi.
- **Background:** Putih/abu-abu muda untuk latar utama dan card.
- **Accent:** Gunakan warna aksen hanya untuk elemen penting, jangan berlebihan.
- **Kontras:** Pastikan teks hitam di atas warna terang, dan sebaliknya.

---

## E. Border Radius & Shadow

- **Radius:**
  - `rounded-lg` untuk card/panel utama
  - `rounded-full` untuk avatar, icon, dan button bulat
- **Shadow:**
  - `shadow-md` untuk card utama (subtle, soft neumorphism)
  - `shadow-sm`/`shadow-none` untuk data card atau elemen sekunder
- Semua radius dan shadow didefinisikan sebagai token di Tailwind config.

---

## H. Interactive States

**Interaksi:**

- **Affordance Jelas:**
  - Button dan dropdown sangat jelas bisa diklik (warna solid, ikon arrow).
  - Card dengan highlight warna menandakan status aktif/terpilih.
- **Micro-interaction (Implied):**
  - Hover state kemungkinan berupa perubahan warna background atau shadow subtle.
  - Tidak ada animasi berat, menjaga performa dan fokus pada konten.

- **Hover:**
  - Button: bg-gold-400, scale(1.02), shadow-lg
  - Card: shadow-xl, border-gold-100
- **Focus:** Outline 2px solid gold/olive, offset 2px
- **Active:** scale(0.98), bg-gold-600
- **Disabled:** opacity-50, cursor-not-allowed
- Semua state diatur di Tailwind config dan utility class.

---

## J. Komponen Utama

**Struktur Layout:**

- **Grid System Modular:**
  - Layout menggunakan grid modular, setiap card adalah satu modul informasi.
  - Spasi antar card besar, memberi ruang napas (breathing space).
- **Card-Based:**
  - Semua informasi utama ditempatkan dalam card dengan sudut membulat.
  - Setiap card punya fungsi spesifik: user info, meetings, roadmap, statistik, AI assistant.
- **Komponen:**
  - **Button:**
    - Bulat, solid, dengan ikon sederhana (arrow, plus).
    - Ada juga tombol dengan outline tipis.
  - **Dropdown:**
    - Sangat minimal, hanya teks dan ikon panah.
  - **Progress Bar:**
    - Flat, tipis, warna aksen.
  - **Avatar & Icon:**
    - Avatar bulat, icon minimalis (notifikasi, waktu, telepon).
  - **Form/Selector:**
    - Dropdown dan date selector sangat sederhana, mudah di-scan.
  - **Chart:**
    - Line chart sederhana dengan warna aksen, tanpa grid berat.

- **Card:** Rounded, shadow, padding, grid/list, modular.
- **Button:** Solid, outline, ghost, icon button, bulat.
- **Table:** Responsive, zebra, sortable.
- **Navbar/Sidebar:** Sticky, minimalis, icon + label.
- **Form/Input:** Large touch target, clear label, error state.
- **Chart:** Bar, line, pie, status color, minimalis.
- **Badge/Status:** Gold, olive, biru, merah.
- Semua komponen diusahakan reusable dan konsisten.

## K. Responsive & Accessibility

- **Kontras Cukup:**
  - Teks hitam di atas warna terang/putih, mudah dibaca.
  - Angka/statistik sangat menonjol.
- **Ukuran Font:**
  - Cukup besar untuk keterbacaan, baik untuk desktop maupun tablet.
- **Icon & Label:**
  - Semua icon didampingi label, memudahkan pemahaman.
- **Whitespace:**
  - Spasi antar elemen besar, memudahkan navigasi visual.

## L. Performance & Utility

- **Use Transform:** Untuk animasi
- **Lazy Loading:** Untuk gambar dan komponen berat
- **Skeleton:** Untuk loading state
- **Critical Path:** Prioritaskan konten utama

---

## Inspirasi & Referensi

- **Aplikasi SaaS Modern:** Notion, Asana, Linear, Monday.com (untuk modular card, grid, dan warna aksen).
- **Google Material Design (versi minimal):** Penggunaan card, shadow subtle, dan icon minimalis.
- **Apple iOS Dashboard:** Rounded card, modular grid, dan tipografi besar.
- **Tren Dribbble/Behance 2022-2024:** Flat, soft color, modular dashboard, micro-interaction, dan AI assistant card.

## Ringkasan Akhir

Desain baru ini mengadopsi gaya Flat Modern Minimalist dengan sentuhan soft neumorphism, palet warna netral dengan aksen lime/yellow, tipografi sans-serif besar, layout grid modular, dan komponen card-based yang sangat clean dan profesional. Pendekatan ini sangat sesuai untuk aplikasi bisnis modern, SaaS dashboard, dan rental management yang membutuhkan visual yang hidup, profesional, dan mudah di-scale.

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EviewNicks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
