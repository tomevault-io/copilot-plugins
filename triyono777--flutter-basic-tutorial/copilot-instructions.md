## flutter-basic-tutorial

> Kurikulum ini dirancang untuk pemula absolut dengan menggunakan **Project IDX**, lingkungan pengembangan berbasis web dari Google. Ini menghilangkan kebutuhan instalasi software yang rumit di komputer siswa.

# Outline Belajar Flutter (SMK) via Project IDX - 8 Pertemuan @ 1 Jam

Kurikulum ini dirancang untuk pemula absolut dengan menggunakan **Project IDX**, lingkungan pengembangan berbasis web dari Google. Ini menghilangkan kebutuhan instalasi software yang rumit di komputer siswa.

**Prasyarat:** Setiap siswa memiliki Akun Google dan koneksi internet.

---

### **Pertemuan 1: Selamat Datang di Project IDX & Flutter**

**Tujuan:** Siswa berhasil masuk ke Project IDX dan menjalankan aplikasi Flutter template.

**Materi:**
1.  **Perkenalan:**
    *   Apa itu Flutter? (Singkat: Satu kode, banyak platform).
    *   Apa itu Project IDX? (Singkat: "Bengkel" coding online, tidak perlu install).
2.  **Setup di Cloud:**
    *   Panduan langkah-demi-langkah untuk login ke `idx.google.com`.
    *   Membuat *workspace* baru dari template Flutter.
3.  **Tur Singkat Project IDX:**
    *   Mengenal area utama: File Explorer, Code Editor, Terminal, dan **Web Preview**.
4.  **Menjalankan Aplikasi Pertama:**
    *   Menjalankan aplikasi template dan melihat hasilnya langsung di Web Preview.

**Praktik:** Setiap siswa berhasil membuat dan menjalankan aplikasi Flutter bawaan di dalam Project IDX.

---

### **Pertemuan 2: Widget Dasar & Modifikasi Tampilan**

**Tujuan:** Siswa mampu memodifikasi kode dan melihat perubahan secara *real-time*.

**Materi:**
1.  **Struktur Kode Awal:** `main.dart`, `runApp()`, `MaterialApp`, `Scaffold`.
2.  **Widget Tampilan Utama:**
    *   `AppBar`: Mengganti judul aplikasi.
    *   `Center` & `Text`: Mengubah tulisan di tengah layar.
    *   `TextStyle`: Mengubah warna, ukuran, dan gaya font.

**Praktik:** Siswa mengubah judul aplikasi dan teks di layar menjadi nama dan kelas mereka, serta mengubah warnanya.

---

### **Pertemuan 3: Menyusun Widget dengan `Column` & `Row`**

**Tujuan:** Siswa mampu menyusun widget secara vertikal dan horizontal.

**Materi:**
1.  **`Column`:** Menyusun widget dari atas ke bawah.
2.  **`Row`:** Menyusun widget dari kiri ke kanan.
3.  **Properti `children`:** Tempat menaruh daftar widget.
4.  **`mainAxisAlignment`:** Mengatur posisi widget di dalam `Column` atau `Row`.

**Praktik:** Membuat tampilan biodata sederhana (Nama, Kelas, Sekolah) yang tersusun rapi ke bawah menggunakan `Column`.

---

### **Pertemuan 4: Menambahkan Gambar & Ikon**

**Tujuan:** Siswa mampu menambahkan aset visual ke dalam aplikasi.

**Materi:**
1.  **`Icon`:** Menggunakan ikon-ikon bawaan Flutter.
2.  **Menambahkan Gambar di Project IDX:**
    *   Membuat folder `assets/images` via File Explorer.
    *   Meng-upload file gambar dari komputer ke folder tersebut.
    *   Mendaftarkan folder `assets/` di file `pubspec.yaml`.
3.  **Widget `Image.asset()` dan `CircleAvatar`**.

**Praktik:** Melengkapi biodata dari pertemuan 3 dengan menambahkan foto profil (di-upload) dan ikon-ikon kecil.

---

### **Pertemuan 5: Membuat Daftar dengan `ListView`**

**Tujuan:** Siswa mampu menampilkan data dalam bentuk daftar yang bisa digulir (scroll).

**Materi:**
1.  **`ListView`:** Widget untuk membuat daftar.
2.  **`ListTile`:** Baris daftar yang praktis (ada slot `leading`, `title`, `subtitle`).
3.  **`Card`:** Membungkus `ListTile` agar terlihat seperti kartu yang rapi.

**Praktik:** Membuat halaman baru yang berisi daftar hobi atau mata pelajaran favorit menggunakan `ListView`, `Card`, dan `ListTile`.

---

### **Pertemuan 6: Aplikasi Interaktif dengan `StatefulWidget`**

**Tujuan:** Siswa mampu membuat aplikasi yang dapat merespon input dari pengguna.

**Materi:**
1.  **Perbedaan `StatelessWidget` vs `StatefulWidget`**.
2.  **Konsep `State` dan `setState()`**: Cara kerja widget yang bisa berubah.
3.  **Widget `ElevatedButton`**: Membuat tombol yang bisa diklik.
4.  **Properti `onPressed`**: Logika yang dijalankan saat tombol ditekan.

**Praktik:** Membuat aplikasi *counter* klasik: sebuah angka dan tombol. Angka akan bertambah setiap kali tombol ditekan.

---

### **Pertemuan 7: Berpindah Halaman (Navigasi)**

**Tujuan:** Siswa mampu membuat aplikasi yang terdiri dari beberapa halaman.

**Materi:**
1.  **Konsep `Navigator`**: Tumpukan halaman di Flutter.
2.  **Membuat File Halaman Baru**: Memisahkan setiap halaman ke dalam file `.dart`-nya sendiri.
3.  **`Navigator.push()`**: Perintah untuk pindah ke halaman baru.
4.  **`Navigator.pop()`**: Perintah untuk kembali ke halaman sebelumnya.

**Praktik:** Membuat aplikasi 2 halaman: Halaman Login dan Halaman Home. Menekan tombol di Halaman Login akan membawa pengguna ke Halaman Home.

---

### **Pertemuan 8: Proyek Mini & Berbagi Aplikasi**

**Tujuan:** Siswa menggabungkan semua konsep dan belajar cara membagikan hasil karyanya.

**Materi:**
1.  **Workshop Proyek Mini**: Membuat aplikasi "Kartu Nama Digital" yang interaktif, menggabungkan layout, gambar, list, dan navigasi.
2.  **Berbagi Aplikasi via Project IDX:**
    *   **Web Preview:** Cara mendapatkan link preview web yang bisa dibagikan ke siapa saja.
    *   **Firebase Integration (Pengenalan):** Penjelasan singkat bahwa IDX bisa dihubungkan ke Firebase untuk membagikan versi Android/iOS (disebutkan sebagai langkah selanjutnya).

**Praktik:** Menyelesaikan proyek mini dan membagikan link Web Preview aplikasi mereka ke teman atau guru.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/triyono777) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
