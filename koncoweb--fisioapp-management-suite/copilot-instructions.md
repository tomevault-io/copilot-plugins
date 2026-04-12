## fisioapp-management-suite

> 1. [Pendahuluan](#pendahuluan)


# Dokumentasi Fitur Pengelolaan Terapi dan Gaji Terapis

## Daftar Isi
1. [Pendahuluan](#pendahuluan)
2. [Alur Proses](#alur-proses)
3. [Fitur untuk Terapis](#fitur-untuk-terapis)
   - [Pencatatan Sesi Terapi](#pencatatan-sesi-terapi)
   - [Pemilihan Pasien](#pemilihan-pasien)
   - [Pemilihan Layanan](#pemilihan-layanan)
4. [Fitur untuk Admin](#fitur-untuk-admin)
   - [Konfirmasi Sesi Terapi](#konfirmasi-sesi-terapi)
   - [Pencatatan Pembayaran](#pencatatan-pembayaran)
   - [Pengelolaan Gaji Terapis](#pengelolaan-gaji-terapis)
5. [Laporan dan Analitik](#laporan-dan-analitik)
6. [Struktur Data](#struktur-data)

## Pendahuluan

Sistem Pengelolaan Terapi dan Gaji Terapis adalah bagian dari Fisioapp Management Suite yang dirancang untuk memudahkan proses pencatatan sesi terapi, konfirmasi oleh admin, pencatatan pembayaran, dan pengelolaan gaji terapis. Sistem ini mengintegrasikan seluruh alur kerja dari awal sesi terapi hingga pembayaran gaji terapis.

## Alur Proses

Berikut adalah alur proses pengelolaan terapi dan gaji terapis:

1. **Pencatatan Sesi Terapi oleh Terapis**
   - Terapis mencatat sesi terapi yang telah dilakukan
   - Terapis memilih pasien dari database atau menambahkan pasien baru
   - Terapis memilih layanan/produk yang diberikan
   - Terapis mengisi detail sesi terapi (tanggal, durasi, catatan)

2. **Konfirmasi oleh Admin**
   - Admin melihat daftar sesi terapi yang telah dicatat oleh terapis
   - Admin melakukan konfirmasi sesi terapi
   - Admin dapat menolak atau meminta revisi jika diperlukan

3. **Pencatatan Pembayaran**
   - Admin mencatat pembayaran untuk sesi terapi yang telah dikonfirmasi
   - Admin memilih jenis pembayaran (langsung atau masuk gaji terapis)
   - Sistem menghasilkan bukti pembayaran (receipt)

4. **Pengelolaan Gaji Terapis**
   - Sistem secara otomatis mencatat pembayaran tipe "salary" ke dalam rekap gaji terapis
   - Admin dapat menambahkan komponen gaji lainnya (bonus, tunjangan, potongan, pajak, kasbon)
   - Admin dapat mencetak dan mengunduh slip gaji terapis
   - Admin menandai gaji sebagai sudah dibayar

## Fitur untuk Terapis

### Pencatatan Sesi Terapi

Terapis dapat mencatat sesi terapi yang telah dilakukan melalui halaman "Catat Terapi" yang dapat diakses dari menu sidebar. Fitur ini memungkinkan terapis untuk:

- Mencatat detail sesi terapi (tanggal, waktu mulai, waktu selesai, catatan)
- Memilih pasien dari database
- Memilih layanan/produk yang diberikan
- Menambahkan catatan khusus untuk sesi terapi
- Menyimpan sesi terapi untuk dikonfirmasi oleh admin

### Pemilihan Pasien

Terapis dapat memilih pasien dari database atau menambahkan pasien baru jika belum terdaftar. Fitur ini meliputi:

- Dropdown untuk memilih pasien yang sudah terdaftar
- Form untuk menambahkan pasien baru (nama, kontak, alamat)
- Pencarian pasien berdasarkan nama atau ID

### Pemilihan Layanan

Terapis dapat memilih layanan/produk yang diberikan kepada pasien. Fitur ini meliputi:

- Dropdown untuk memilih layanan dari database produk
- Informasi harga layanan secara otomatis ditampilkan
- Opsi untuk menambahkan multiple layanan dalam satu sesi

## Fitur untuk Admin

### Konfirmasi Sesi Terapi

Admin dapat mengonfirmasi sesi terapi yang telah dicatat oleh terapis melalui halaman "Konfirmasi Terapi". Fitur ini meliputi:

- Daftar sesi terapi yang belum dikonfirmasi
- Detail sesi terapi (terapis, pasien, layanan, tanggal, durasi)
- Tombol untuk mengonfirmasi atau menolak sesi terapi
- Opsi untuk menambahkan catatan saat konfirmasi atau penolakan

### Pencatatan Pembayaran

Admin dapat mencatat pembayaran untuk sesi terapi yang telah dikonfirmasi melalui halaman "Pembayaran Terapi". Fitur ini meliputi:

- Daftar sesi terapi yang telah dikonfirmasi
- Form pembayaran dengan pilihan metode pembayaran
- Dropdown untuk memilih jenis pembayaran (langsung atau masuk gaji terapis)
- Tombol untuk mencatat pembayaran
- Generasi bukti pembayaran (receipt)
- Status pembayaran yang diperbarui secara real-time

Saat admin memilih jenis pembayaran "salary", sistem akan secara otomatis mencatat pembayaran tersebut ke dalam rekap gaji terapis untuk bulan yang sesuai.

### Pengelolaan Gaji Terapis

Admin dapat mengelola gaji terapis melalui halaman "Gaji Terapis". Fitur ini meliputi:

- Filter untuk memilih terapis dan periode (bulan/tahun)
- Tampilan rekap gaji terapis dengan rincian komponen
- Fitur untuk menambahkan komponen gaji (bonus, tunjangan, potongan, pajak, kasbon)
- Fitur untuk menandai gaji sebagai sudah dibayar
- Cetak dan unduh slip gaji sebagai PDF

Komponen gaji yang dapat ditambahkan meliputi:
- **Bonus**: Tambahan pendapatan untuk terapis (misalnya bonus kinerja)
- **Tunjangan**: Tambahan pendapatan tetap (misalnya tunjangan transportasi)
- **Potongan**: Pengurangan dari gaji (misalnya potongan keterlambatan)
- **Pajak**: Pengurangan untuk pembayaran pajak
- **Kasbon**: Pengurangan untuk pembayaran kasbon/pinjaman

## Laporan dan Analitik

Sistem menyediakan laporan dan analitik untuk membantu manajemen dalam mengambil keputusan:

- Laporan sesi terapi per terapis
- Laporan pendapatan dari sesi terapi
- Rekap gaji terapis per bulan
- Analisis tren layanan yang paling banyak digunakan

## Struktur Data

### Sesi Terapi
```typescript
interface TherapySession {
  id: string;
  therapistId: string;
  therapistName: string;
  patientId: string;
  patientName: string;
  serviceId: string;
  serviceName: string;
  price: number;
  date: string;
  startTime: string;
  endTime: string;
  duration: number;
  notes?: string;
  status: 'pending' | 'confirmed' | 'rejected';
  createdAt: string;
  updatedAt?: string;
}
```

### Pembayaran Terapi
```typescript
interface TherapyPayment {
  id: string;
  therapySessionId: string;
  therapistId: string;
  therapistName: string;
  patientId: string;
  patientName: string;
  serviceId: string;
  serviceName: string;
  amount: number;
  paymentMethod: string;
  paymentType: 'direct' | 'salary';
  status: 'pending' | 'paid' | 'cancelled';

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/koncoweb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
