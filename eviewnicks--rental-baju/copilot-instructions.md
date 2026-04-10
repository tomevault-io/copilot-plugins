## rental-baju

> Dokumen ini berisi panduan standar untuk format dokumentasi hasil implementasi task di project Maguru. Panduan ini mengacu pada best practice JIRA Agile Scrum dan memastikan konsistensi pelaporan hasil pengembangan.


# Panduan Dokumentasi Hasil (result-ops-xxx.md)

## Tujuan Panduan

Dokumen ini berisi panduan standar untuk format dokumentasi hasil implementasi task di project Maguru. Panduan ini mengacu pada best practice JIRA Agile Scrum dan memastikan konsistensi pelaporan hasil pengembangan.

# [OPS-XXX] Hasil Implementasi [Nama Fitur]

**Status**: [🟢 Complete | 🟡 Partial | 🔴 Blocked]  
**Diimplementasikan**: [Tanggal Mulai] - [Tanggal Selesai]  
**Developer**: [Nama Developer]  
**Reviewer**: [Nama Reviewer]  
**PR**: [Link Pull Request]

---

## Daftar Isi

1. [Ringkasan Implementasi](mdc:#ringkasan-implementasi)
2. [Perubahan dari Rencana Awal](mdc:#perubahan-dari-rencana-awal)
3. [Status Acceptance Criteria](mdc:#status-acceptance-criteria)
4. [Detail Implementasi](mdc:#detail-implementasi)
5. [Kendala dan Solusi](mdc:#kendala-dan-solusi)
6. [Rekomendasi Selanjutnya](mdc:#rekomendasi-selanjutnya)

## Ringkasan Implementasi (opsional Jika berhbungan dengan pembuatan fitur)

[Ringkasan singkat (3-5 kalimat) tentang fitur yang diimplementasikan, highlight utama, dan nilai bisnis yang dihasilkan]

### Ruang Lingkup

[Deskripsi singkat tentang apa yang tercakup dalam implementasi dan apa yang tidak]

#### 1. React Components

**Server Components**:

- [Nama Komponen]: [Deskripsi singkat]
- ...

**Client Components**:

- [Nama Komponen]: [Deskripsi singkat]
- ...

#### 2. State Management

**Context Providers**:

- [Nama Context]: [Deskripsi singkat]
- ...

**React Query/State**:

- [Nama Hook/Query]: [Deskripsi singkat]
- ...

#### 3. Custom Hooks

**Feature Hooks**:

- [Nama Hook]: [Deskripsi singkat]
- ...

**Utility Hooks**:

- [Nama Hook]: [Deskripsi singkat]
- ...

#### 4. Data Access

**Adapters**:

- [Nama Adapter]: [Deskripsi singkat]
- ...

**API Endpoints**:

- `[METHOD] /api/[path]` - [Deskripsi singkat]
- ...

#### 5. Server-side

**Services**:

- [Nama Service]: [Deskripsi singkat]
- ...

**Database Schema**:

- [Nama Model/Perubahan]: [Deskripsi singkat]
- ...

#### 6. Cross-cutting Concerns

**Types**:

- [Nama Type]: [Deskripsi singkat]
- ...

**Utils**:

- [Nama Util]: [Deskripsi singkat]
- ...

## Perubahan dari Rencana Awal

[Deskripsi perubahan signifikan dari rencana awal (task-ops-xxx.md) dan justifikasinya]

### Perubahan Desain

| Komponen/Fitur  | Rencana Awal | Implementasi Aktual | Justifikasi |
| --------------- | ------------ | ------------------- | ----------- |
| [Nama Komponen] | [Deskripsi]  | [Deskripsi]         | [Alasan]    |
| [API Endpoint]  | [Deskripsi]  | [Deskripsi]         | [Alasan]    |

### Perubahan Teknis

| Aspek           | Rencana Awal | Implementasi Aktual | Justifikasi |
| --------------- | ------------ | ------------------- | ----------- |
| [Struktur Data] | [Deskripsi]  | [Deskripsi]         | [Alasan]    |
| [Teknologi]     | [Deskripsi]  | [Deskripsi]         | [Alasan]    |

## Status Acceptance Criteria

| Kriteria     | Status | Keterangan                  |
| ------------ | ------ | --------------------------- |
| [Kriteria 1] | ✅     | [Keterangan jika perlu]     |
| [Kriteria 2] | ⚠️     | [Keterangan keterbatasan]   |
| [Kriteria 3] | ❌     | [Alasan tidak implementasi] |

## Detail Implementasi

> **⚠️ PENTING**: Dokumentasi ini harus fokus pada detail implementasi yang jelas dan ringkas. **HINDARI MENAMPILKAN PSEUDOCODE ATAU IMPLEMENTASI KODE YANG RUMIT**. Berikan penjelasan tingkat tinggi tentang pendekatan yang diambil, pola yang digunakan, dan alasan di balik keputusan teknis. Pengecualian hanya untuk Schema Database dan TypeScript Interface yang perlu ditampilkan sebagaimana diimplementasikan.

### Arsitektur Folder

Implementasi mengikuti struktur folder standar yang didefinisikan dalam arsitektur Maguru:

```
/features/[feature-name]/
├── components/         # Komponen React
│   ├── ComponentName/  # Komponen dengan sub-komponen
│   │   └── ...
│   └── ...
├── context/            # React Context Providers
│   └── ...
├── hooks/              # Custom React Hooks
│   ├── feature/        # Feature-specific hooks
│   └── shared/         # Shared hooks
├── adapters/           # Data access adapters (API calls)
│   └── ...
├── services/           # Business logic
│   └── ...
├── types/              # TypeScript type definitions
│   └── ...
└── utils/              # Utility functions
    └── ...
```

> **Catatan**: Jika implementasi Anda menyimpang dari struktur standar di atas, jelaskan alasannya di sini.

### Komponen Utama

#### [Nama Komponen]

**File**: `/path/to/component.tsx`

**Deskripsi**:
[Deskripsi fungsi dan tanggung jawab komponen]

**Pattern yang Digunakan**:

- [Pattern 1]: [Deskripsi]
- [Pattern 2]: [Deskripsi]

### Alur Data

[Jelaskan alur data dari API ke UI atau sebaliknya dengan fokus pada:

1. Bagaimana data dimuat dari backend
2. Bagaimana state dikelola (React Query, Context, dll)
3. Bagaimana state mempengaruhi UI
4. Bagaimana mutasi/update data dilakukan]

### Database Schema

[Jika ada perubahan skema database, jelaskan di sini]

```prisma
model UpdatedModel {
  // Skema yang diimplementasikan
}
```

### API Implementation

#### [Endpoint URL]

**File**: `/path/to/api/file`

**Method**: [GET/POST/PUT/DELETE]

**Authentication**: [Required/Optional]

**Error Handling**:

- [Status Code]: [Deskripsi kondisi]
- [Status Code]: [Deskripsi kondisi]

## Kendala dan Solusi

### Kendala 1: [Judul Kendala]

**Deskripsi**:
[Penjelasan detail kendala yang dihadapi]

**Solusi**:
[Bagaimana kendala diselesaikan, atau workaround yang diterapkan]

**Pembelajaran**:
[Pelajaran yang bisa diambil untuk implementasi di masa depan]

### Kendala 2: [Judul Kendala]

[Dan seterusnya...]

## Rekomendasi Selanjutnya

### Peningkatan Fitur

1. [Rekomendasi 1]: [Deskripsi dan justifikasi]
2. [Rekomendasi 2]: [Deskripsi dan justifikasi]

### Technical Debt

1. [Technical Debt 1]: [Deskripsi dan dampak]
2. [Technical Debt 2]: [Deskripsi dan dampak]

### Potensi Refactoring

1. [Refactoring 1]: [Deskripsi dan manfaat]
2. [Refactoring 2]: [Deskripsi dan manfaat]

## Lampiran

- [Link dokumen teknis tambahan]
- [Link PR review]
- [Link ke test report]

> **Catatan**: Untuk detail pengujian (Unit, Integration, E2E, Performance), silakan merujuk ke dokumen test report di `features/[feature-name]/docs/report-test/test-report.md`. Dokumen ini berfokus pada implementasi, bukan hasil pengujian.

## Panduan Penggunaan

### Penamaan File

- Format: `result-ops-[ID].md` (contoh: `result-ops-173.md`)
- Simpan di direktori `features/[feature-name]/docs/result-docs/`

### Waktu Penulisan

Dokumentasi hasil harus dibuat langsung setelah implementasi fitur selesai dan sebelum code review, dengan update tambahan setelah code review jika diperlukan.

### Panduan Penulisan

1. **Faktual dan Objektif**: Tuliskan hasil implementasi secara faktual, tidak hanya rencana atau harapan.
2. **Tunjukkan Perbandingan**: Selalu bandingkan hasil aktual dengan rencana awal yang didokumentasikan di task-ops.
3. **Dokumentasikan Keputusan**: Jelaskan alasan di balik perubahan rencana atau pendekatan alternatif yang diambil.
4. **Lengkap dan Detil**: Berikan informasi yang cukup untuk developer lain dapat memahami keseluruhan implementasi.
5. **Visual**: Sertakan screenshot, diagram, atau video demo jika memungkinkan.
6. **Hindari Pseudocode**: Jangan menampilkan implementasi kode yang rumit, fokus pada penjelasan pendekatan dan pola yang digunakan.

### Integrasi dengan JIRA dan GitHub

1. Lampirkan link ke dokumen hasil di komentar task JIRA
2. Sertakan link dalam deskripsi Pull Request
3. Referensikan nomor issue/task menggunakan format OPS-XXX di commit messages

### Status Implementasi

Gunakan format status berikut untuk kejelasan:

- 🟢 **Complete**: Semua acceptance criteria terpenuhi tanpa kompromi
- 🟡 **Partial**: Sebagian besar AC terpenuhi, dengan beberapa keterbatasan yang sudah disetujui
- 🔴 **Blocked**: Implementasi terhambat oleh kendala yang memerlukan diskusi lebih lanjut

## Template Sections

### Template Status Acceptance Criteria

```markdown
## Status Acceptance Criteria

| Kriteria                                      | Status | Keterangan                                           |
| --------------------------------------------- | ------ | ---------------------------------------------------- |
| Pengguna dapat membuat folder baru            | ✅     | Implementasi sesuai rencana                          |
| Pengguna dapat mengedit judul page            | ✅     | UI lebih sederhana dari rencana awal                 |
| Pengguna dapat menghapus folder dan kontennya | ⚠️     | Konfirmasi hanya 1 level, tidak recursive            |
| Drag & drop untuk reordering                  | ❌     | Dipindahkan ke sprint berikutnya karena kompleksitas |
```

### Template Kendala dan Solusi

```markdown
## Kendala dan Solusi

### Kendala 1: Integrasi Library X dengan Component Y

**Deskripsi**:
Library X ternyata tidak mendukung fitur Z yang kita butuhkan untuk implementasi sesuai rencana awal.

**Solusi**:
Membuat wrapper custom yang menambahkan fungsionalitas yang diperlukan, dengan trade-off performa yang sedikit menurun.

**Pembelajaran**:
Untuk fitur kompleks, lakukan spike/research lebih mendalam terhadap kapabilitas library sebelum finalisasi desain.

### Kendala 2: Performance Issue saat Large Dataset

**Deskripsi**:
Terjadi lag signifikan saat loading dan rendering data lebih dari 1000 items.

**Solusi**:

- Implementasi virtualized list untuk menampilkan hanya items yang visible
- Tambahkan pagination di API level
- Cache hasil query dengan React Query

**Pembelajaran**:
Selalu pertimbangkan edge cases dengan dataset besar sejak awal desain.
```

## Contoh Dokumentasi Hasil

Untuk melihat contoh dokumentasi hasil yang baik:

- [result-ops-173.md](mdc:docs/result-docs/result-ops-173.md)

## Integrasi dengan Task Documentation

Dokumentasi hasil harus memiliki hubungan yang jelas dengan dokumentasi task asli. Praktik terbaik untuk memastikan integrasi ini adalah:

1. **Cross-referencing**: Selalu referensikan task-ops-xxx.md di result-ops-xxx.md dan sebaliknya
2. **Shared Structure**: Gunakan struktur yang paralel antara task dan result untuk memudahkan perbandingan
3. **Explicit Diff**: Jelaskan secara eksplisit perbedaan antara rencana dan implementasi aktual
4. **Shared Terminology**: Gunakan istilah yang konsisten antara task dan result documentation

## Keterkaitan dengan Dokumentasi Pengujian

Dokumentasi hasil tidak perlu mencantumkan detail testing secara lengkap karena sudah didokumentasikan dalam format terpisah:

1. **Test Report**: Hasil pengujian didokumentasikan dalam `features/[feature-name]/docs/report-test/test-report.md`
2. **Cross-reference**: Cukup referensikan dokumen test report dalam bagian lampiran jika diperlukan
3. **Highlight Issues**: Jika ada temuan penting dari pengujian yang perlu diketahui, cukup rangkum di bagian kendala

## Review Guidelines

### Developer Self-Review Checklist

- [ ] Semua acceptance criteria didokumentasikan dengan status yang akurat
- [ ] Perubahan dari rencana awal dijelaskan dengan justifikasi
- [ ] Struktur dan pola yang digunakan dijelaskan dengan jelas
- [ ] Kendala dan solusi didokumentasikan secara komprehensif
- [ ] Arsitektur folder diuraikan dengan jelas sesuai implementasi

### Reviewer Checklist

- [ ] Dokumentasi hasil akurat dan sesuai dengan implementasi aktual
- [ ] Justifikasi untuk perubahan dari rencana masuk akal dan didokumentasikan
- [ ] Status acceptance criteria transparan dan realistis
- [ ] Technical debt dan rekomendasi selanjutnya masuk akal
- [ ] Referensi ke task asli dan dokumentasi pendukung tersedia

## FAQ

**Q: Kapan sebaiknya menulis result documentation?**  
A: Mulailah menulis draft sejak implementasi dimulai, dan finalisasi segera setelah implementasi selesai sebelum code review.

**Q: Bagaimana jika implementasi sangat berbeda dari rencana?**  
A: Dokumentasikan perbedaan dengan jelas, berikan justifikasi yang kuat, dan lakukan cross-reference ke diskusi atau keputusan yang menyebabkan perubahan.

**Q: Apakah perlu mendokumentasikan implementasi yang gagal?**  
A: Ya, dokumentasikan semua upaya implementasi termasuk yang gagal, karena ini menjadi pembelajaran berharga untuk tim dan mencegah pengulangan kesalahan.

**Q: Siapa yang harus membaca dokumentasi hasil?**  
A: Developer yang mengimplementasi untuk self-check, reviewer untuk validasi, product owner untuk verifikasi hasil, dan developer lain yang mungkin bekerja dengan kode tersebut di masa depan.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EviewNicks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
