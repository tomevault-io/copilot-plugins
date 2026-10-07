## safarstrack

> Dokumen ini adalah acuan **Wajib & Mengikat** bagi AI Agent (dan developer) dalam membuat, mengubah, memelihara, dan mengembangkan codebase aplikasi ini.

# GEMINI.md - Panduan Utama & Rules Coding AI Agent

Dokumen ini adalah acuan **Wajib & Mengikat** bagi AI Agent (dan developer) dalam membuat, mengubah, memelihara, dan mengembangkan codebase aplikasi ini.

---

## 🚨 MANDATORY WORKFLOW FOR AI AGENTS

Setiap kali AI Agent memulai atau mengerjakan tugas pada aplikasi ini, AI Agent **WAJIB** mengecek, merujuk, dan memperbarui seluruh file dokumentasi berikut secara disiplin:

| File Dokumentasi | Fungsi & Peran | Kewajiban AI Agent |
| :--- | :--- | :--- |
| [GEMINI.md](file:///Users/tnoerman/Herd/hajitrackers/GEMINI.md) | Technical Documentation, Rule Coding, Standard Code & Routing. | **Wajib dibaca** untuk memastikan kepatuhan standar teknis. |
| [docs/PRD.md](file:///Users/tnoerman/Herd/hajitrackers/docs/PRD.md) | Product Requirement Document (Acuan awal fitur & tujuan aplikasi). | **Wajib diverifikasi** sebelum membuat fitur baru. |
| [docs/STRUKTUR_MENU.md](file:///Users/tnoerman/Herd/hajitrackers/docs/STRUKTUR_MENU.md) | Acuan RBAC (Role-Based Access Control) & Menu tiap Role. | **Wajib disesuaikan** jika ada penambahan halaman/menu/role. |
| [docs/ERD.md](file:///Users/tnoerman/Herd/hajitrackers/docs/ERD.md) | Relasi & Skema Database (Entity Relationship Diagram). | **Wajib diupdate** setiap kali ada migrasi / perubahan tabel. |
| [docs/FITUR_APLIKASI.md](file:///Users/tnoerman/Herd/hajitrackers/docs/FITUR_APLIKASI.md) | Pendataan fitur lengkap + Diagram ASCII + Alur Proses + Perubahan Status Data. | **Wajib dibuat / diupdate** setiap ada fitur baru / perubahan alur. |
| [docs/DESAIN.md](file:///Users/tnoerman/Herd/hajitrackers/docs/DESAIN.md) | Standar UI/UX, Skema Warna, Pagination, Cards, Filter, CDN (SweetAlert, Icons). | **Wajib diikuti** dalam pembuatan komponen Blade / Frontend. |
| [docs/DOKUMENTASI_PENGGUNAAN.md](file:///Users/tnoerman/Herd/hajitrackers/docs/DOKUMENTASI_PENGGUNAAN.md) | Manual Book / Panduan Penggunaan Bahasa Awam (Step-by-Step). | **Wajib diperbarui** agar pengguna awam dapat menggunakan fitur baru. |
| [docs/UAT.md](file:///Users/tnoerman/Herd/hajitrackers/docs/UAT.md) | Skenario Pengujian (User Acceptance Testing) untuk pengujian pengguna. | **Wajib ditambahkan** skenario pengujian baru saat penambahan fitur. |
| [docs/TASK_DONE.md](file:///Users/tnoerman/Herd/hajitrackers/docs/TASK_DONE.md) | Log Perubahan & Audit Trail Pekerjaan AI Agent. | **Wajib dicatat** setiap selesai melakukan perubahan code. |

---

## 🛠️ SPESIFIKASI TEKNIS APLIKASI

| Teknologi | Versi / Detail | Keterangan |
| :--- | :--- | :--- |
| **PHP** | `8.3` | Runtime utama |
| **Framework** | `Laravel 13` | Backend framework |
| **Database** | `PostgreSQL` | Database utama (bukan MySQL/SQLite) |
| **Template Engine** | `Blade (Full)` | 100% Blade — **DILARANG** menggunakan Livewire, Inertia, Vue, React |
| **CSS Framework** | `Tailwind CSS v4` | Sesuai standar [docs/DESAIN.md](file:///Users/tnoerman/Herd/hajitrackers/docs/DESAIN.md) |
| **JavaScript** | `jQuery 3.7.1` | Untuk Select2 dan DOM manipulation |
| **Enhanced Select** | `Select2 4.1.0-rc.0` | Dropdown select yang lebih baik |
| **Interaktivitas** | `Alpine.js v3` | **Gunakan seminimal mungkin** — hanya untuk dropdown/modal toggle jika tidak bisa pakai jQuery |
| **Notifikasi** | `SweetAlert2 v11` | Toast & confirmation dialog |
| **Icon** | `Heroicons (SVG inline)` + `RemixIcon (CSS class)` | Sesuai [docs/DESAIN.md](file:///Users/tnoerman/Herd/hajitrackers/docs/DESAIN.md) |
| **Spreadsheet** | `PhpSpreadsheet` | Import & Export data (Excel/CSV) |
| **Testing** | `Pest 4` | Unit & Feature testing |

> **PENTING — Alpine.js:** Gunakan Alpine.js **seminimal mungkin**. Jika sebuah interaksi (show/hide, toggle, dll) bisa diselesaikan dengan jQuery atau vanilla JS, maka **JANGAN** menggunakan Alpine.js. Alpine.js hanya boleh digunakan jika memang tidak ada alternatif lain yang lebih sederhana (misalnya: `x-cloak` untuk anti-flash, atau reactive binding yang sangat kompleks).

---

## 📐 ATURAN CODING & ARSITEKTUR APLIKASI

### 1. Filosofi Coding — Eksplisit, Sederhana, Mudah Dibaca

> **Posisikan diri sebagai senior programmer yang menulis code agar mudah dibaca oleh junior programmer yang masih awam.**

- **Tulis code secara eksplisit** — hindari penggunaan magic function, implicit binding, atau shortcut yang menyembunyikan logika.
- **Jangan terlalu modular** — jangan memecah code ke banyak file/class/service jika tidak perlu. Lebih baik satu file controller yang jelas daripada 5 file yang saling panggil.
- **Setiap fungsi ditulis jelas & to the point** — tidak berbelit-belit. Nama fungsi dan variabel menggunakan **bahasa Inggris** yang deskriptif.
- **Setiap baris code diberi komentar singkat dalam bahasa Indonesia** — menjelaskan "fungsi ini / proses ini melakukan apa". Tidak perlu terlalu detail, tetapi tetap menjelaskan proses.

**Contoh komentar yang benar:**
```php
// Ambil semua data jamaah beserta relasi paketnya
$jamaahList = Jamaah::with('jamaahPakets.paketHaji')->paginate(10);

// Validasi input dari form tambah jamaah
$validated = $request->validate([
    'nik' => 'required|string|size:16|unique:jamaahs,nik',
    'nama_lengkap' => 'required|string|max:255',
]);

// Simpan data jamaah baru ke database
$jamaah = Jamaah::create($validated);

// Redirect ke halaman index dengan pesan sukses
return redirect()->route('jamaah.index')->with('success', 'Data jamaah berhasil ditambahkan.');
```

---

### 2. Penamaan File & Class (Naming Conventions)

#### 2.1 Model — `PascalCase`, Singular

Model menggunakan PascalCase singular. Nama tabel di database menggunakan `snake_case` plural.

| Model | File | Tabel Database |
| :--- | :--- | :--- |
| `Jamaah` | `app/Models/Jamaah.php` | `jamaahs` |
| `PaketHaji` | `app/Models/PaketHaji.php` | `paket_hajis` |
| `JamaahPaket` | `app/Models/JamaahPaket.php` | `jamaah_pakets` |
| `Pembayaran` | `app/Models/Pembayaran.php` | `pembayarans` |
| `DokumenJamaah` | `app/Models/DokumenJamaah.php` | `dokumen_jamaahs` |
| `TrackingProses` | `app/Models/TrackingProses.php` | `tracking_proses` |
| `User` | `app/Models/User.php` | `users` |
| `Role` | `app/Models/Role.php` | `roles` |

**Contoh Model (Laravel 13 style):**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Jamaah extends Model
{
    use HasFactory;

    // Kolom yang boleh diisi secara mass-assignment
    protected $fillable = [
        'nik',
        'nama_lengkap',
        'jenis_kelamin',
        'tanggal_lahir',
        'nomor_paspor',
        'nomor_hp',
        'alamat',
    ];

    // Casting tipe data untuk kolom tertentu
    protected function casts(): array
    {
        return [
            'tanggal_lahir' => 'date',
        ];
    }

    // Relasi: Satu jamaah bisa punya banyak pendaftaran paket
    public function jamaahPakets(): HasMany
    {
        return $this->hasMany(JamaahPaket::class);
    }
}
```

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;

class JamaahPaket extends Model
{
    use HasFactory;

    // Kolom yang boleh diisi secara mass-assignment
    protected $fillable = [
        'jamaah_id',
        'paket_haji_id',
        'nomor_pendaftaran',
        'tanggal_daftar',
        'total_tagihan',
        'sisa_tagihan',
        'status_pembayaran',
        'status_keberangkatan',
    ];

    // Casting tipe data
    protected function casts(): array
    {
        return [
            'tanggal_daftar' => 'date',
            'total_tagihan' => 'decimal:2',
            'sisa_tagihan' => 'decimal:2',
        ];
    }

    // Relasi: Pendaftaran ini milik satu jamaah
    public function jamaah(): BelongsTo
    {
        return $this->belongsTo(Jamaah::class);
    }

    // Relasi: Pendaftaran ini milik satu paket haji
    public function paketHaji(): BelongsTo
    {
        return $this->belongsTo(PaketHaji::class);
    }

    // Relasi: Satu pendaftaran bisa punya banyak pembayaran
    public function pembayarans(): HasMany
    {
        return $this->hasMany(Pembayaran::class);
    }

    // Relasi: Satu pendaftaran bisa punya banyak dokumen
    public function dokumenJamaahs(): HasMany
    {
        return $this->hasMany(DokumenJamaah::class);
    }
}
```

#### 2.2 Controller — `PascalCase` + Postfix `Controller`

| Controller | File |
| :--- | :--- |
| `DashboardController` | `app/Http/Controllers/DashboardController.php` |
| `JamaahController` | `app/Http/Controllers/JamaahController.php` |
| `PaketHajiController` | `app/Http/Controllers/PaketHajiController.php` |
| `PembayaranController` | `app/Http/Controllers/PembayaranController.php` |

**Contoh Controller:**
```php
<?php

namespace App\Http\Controllers;

use App\Models\Jamaah;
use App\Models\PaketHaji;
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\View\View;

class JamaahController extends Controller
{
    // Menampilkan daftar seluruh data jamaah dengan filter & pagination
    public function index(Request $request): View
    {
        // Buat query dasar dengan eager loading untuk hindari N+1
        $query = Jamaah::with('jamaahPakets.paketHaji');

        // Filter berdasarkan pencarian nama atau NIK
        if ($request->filled('search')) {
            $search = $request->input('search');
            $query->where(function ($q) use ($search) {
                $q->where('nama_lengkap', 'ilike', "%{$search}%")
                  ->orWhere('nik', 'ilike', "%{$search}%");
            });
        }

        // Filter berdasarkan jenis kelamin
        if ($request->filled('jenis_kelamin')) {
            $query->where('jenis_kelamin', $request->input('jenis_kelamin'));
        }

        // Ambil data dengan pagination 10 row per halaman
        $jamaahList = $query->orderBy('created_at', 'desc')->paginate(10);

        // Preserve filter saat pindah halaman pagination
        $jamaahList->appends($request->query());

        return view('jamaah.index', compact('jamaahList'));
    }

    // Menampilkan halaman form tambah jamaah baru
    public function create(): View
    {
        // Ambil data paket haji untuk dropdown select
        $paketList = PaketHaji::where('status', 'Active')
            ->orderBy('nama_paket')
            ->get();

        return view('jamaah.create', compact('paketList'));
    }

    // Menyimpan data jamaah baru ke database
    public function store(Request $request): RedirectResponse
    {
        // Validasi semua input — pesan error dalam bahasa Indonesia
        $validated = $request->validate([
            'nik'           => 'required|string|size:16|unique:jamaahs,nik',
            'nama_lengkap'  => 'required|string|max:255',
            'jenis_kelamin' => 'required|in:Laki-laki,Perempuan',
            'tanggal_lahir' => 'required|date|before:today',
            'nomor_paspor'  => 'nullable|string|max:50',
            'nomor_hp'      => 'required|string|max:20',
            'alamat'        => 'nullable|string|max:500',
        ], [
            'nik.required'          => 'NIK wajib diisi.',
            'nik.size'              => 'NIK harus terdiri dari 16 digit.',
            'nik.unique'            => 'NIK sudah terdaftar di sistem.',
            'nama_lengkap.required' => 'Nama lengkap wajib diisi.',
            'jenis_kelamin.required'=> 'Jenis kelamin wajib dipilih.',
            'jenis_kelamin.in'      => 'Jenis kelamin tidak valid.',
            'tanggal_lahir.required'=> 'Tanggal lahir wajib diisi.',
            'tanggal_lahir.before'  => 'Tanggal lahir harus sebelum hari ini.',
            'nomor_hp.required'     => 'Nomor HP wajib diisi.',
        ]);

        // Simpan data jamaah baru ke database
        Jamaah::create($validated);

        // Redirect ke halaman index dengan pesan sukses
        return redirect()->route('jamaah.index')
            ->with('success', 'Data jamaah berhasil ditambahkan.');
    }

    // Menampilkan detail lengkap satu data jamaah
    public function show(int $id): View
    {
        // Ambil data jamaah beserta semua relasi terkait
        $jamaah = Jamaah::with([
            'jamaahPakets.paketHaji',
            'jamaahPakets.pembayarans',
            'jamaahPakets.dokumenJamaahs',
        ])->findOrFail($id);

        return view('jamaah.show', compact('jamaah'));
    }

    // Menampilkan halaman form edit data jamaah
    public function edit(int $id): View
    {
        // Ambil data jamaah yang akan diedit
        $jamaah = Jamaah::findOrFail($id);

        return view('jamaah.edit', compact('jamaah'));
    }

    // Memperbarui data jamaah yang sudah ada
    public function update(Request $request, int $id): RedirectResponse
    {
        // Ambil data jamaah yang akan diupdate
        $jamaah = Jamaah::findOrFail($id);

        // Validasi input — NIK unik kecuali milik jamaah ini sendiri
        $validated = $request->validate([
            'nik'           => 'required|string|size:16|unique:jamaahs,nik,' . $jamaah->id,
            'nama_lengkap'  => 'required|string|max:255',
            'jenis_kelamin' => 'required|in:Laki-laki,Perempuan',
            'tanggal_lahir' => 'required|date|before:today',
            'nomor_paspor'  => 'nullable|string|max:50',
            'nomor_hp'      => 'required|string|max:20',
            'alamat'        => 'nullable|string|max:500',
        ], [
            'nik.required'          => 'NIK wajib diisi.',
            'nik.size'              => 'NIK harus terdiri dari 16 digit.',
            'nik.unique'            => 'NIK sudah digunakan oleh jamaah lain.',
            'nama_lengkap.required' => 'Nama lengkap wajib diisi.',
            'jenis_kelamin.required'=> 'Jenis kelamin wajib dipilih.',
            'tanggal_lahir.required'=> 'Tanggal lahir wajib diisi.',
            'nomor_hp.required'     => 'Nomor HP wajib diisi.',
        ]);

        // Update data jamaah di database
        $jamaah->update($validated);

        // Redirect ke halaman index dengan pesan sukses
        return redirect()->route('jamaah.index')
            ->with('success', 'Data jamaah berhasil diperbarui.');
    }

    // Menghapus data jamaah dari database
    public function destroy(int $id): RedirectResponse
    {
        // Ambil data jamaah yang akan dihapus
        $jamaah = Jamaah::findOrFail($id);

        // Hapus data jamaah
        $jamaah->delete();

        // Redirect ke halaman index dengan pesan sukses
        return redirect()->route('jamaah.index')
            ->with('success', 'Data jamaah berhasil dihapus.');
    }
}
```

#### 2.3 Migration — `snake_case` dengan Timestamp Laravel

```
database/migrations/2024_01_15_100000_create_jamaahs_table.php
database/migrations/2024_01_15_100001_create_paket_hajis_table.php
```

#### 2.4 Blade View — `snake_case` di dalam Folder Modul

```
resources/views/
├── jamaah/
│   ├── index.blade.php       ← Halaman list/tabel data
│   ├── create.blade.php      ← Halaman form tambah data
│   ├── edit.blade.php        ← Halaman form edit data
│   └── show.blade.php        ← Halaman detail data (jika diperlukan)
├── paket-haji/
│   ├── index.blade.php
│   ├── create.blade.php
│   ├── edit.blade.php
│   └── show.blade.php
├── pembayaran/
│   ├── index.blade.php
│   ├── create.blade.php
│   └── edit.blade.php
```

---

### 3. Aturan Controller & Logika Bisnis

#### 3.1 Semua Logika di Controller

- **Semua proses logika bisnis ditulis langsung di Controller** — tidak di Service class, tidak di Action class.
- **DILARANG membuat Service class** kecuali diminta secara eksplisit oleh user.
- **DILARANG membuat file Form Request** (seperti `StoreJamaahRequest.php`) — semua validasi di-handle langsung di method Controller menggunakan `$request->validate()`.
- **Relasi database** tetap didefinisikan di Model (bukan di Controller).

#### 3.2 Validasi — Langsung di Controller

```php
// ✅ BENAR — Validasi langsung di controller
public function store(Request $request): RedirectResponse
{
    $validated = $request->validate([
        'nama_lengkap' => 'required|string|max:255',
        'nik'          => 'required|string|size:16|unique:jamaahs,nik',
    ], [
        'nama_lengkap.required' => 'Nama lengkap wajib diisi.',
        'nik.required'          => 'NIK wajib diisi.',
        'nik.size'              => 'NIK harus 16 digit.',
        'nik.unique'            => 'NIK sudah terdaftar.',
    ]);

    Jamaah::create($validated);

    return redirect()->route('jamaah.index')
        ->with('success', 'Data jamaah berhasil ditambahkan.');
}
```

```php
// ❌ SALAH — Jangan membuat file Form Request terpisah
// JANGAN buat file: app/Http/Requests/StoreJamaahRequest.php
```

#### 3.3 Pesan Validasi — Bahasa Indonesia, Singkat & Jelas

Setiap rule validasi **WAJIB** memiliki custom message dalam **bahasa Indonesia** yang singkat dan mudah dipahami:

```php
$validated = $request->validate([...], [
    'nama.required'      => 'Nama wajib diisi.',
    'email.email'        => 'Format email tidak valid.',
    'tanggal.before'     => 'Tanggal harus sebelum hari ini.',
    'file.max'           => 'Ukuran file maksimal 2MB.',
    'nomor_hp.required'  => 'Nomor HP wajib diisi.',
    'jenis.in'           => 'Pilihan jenis tidak valid.',
]);
```

---

### 4. Standar Routing — Eksplisit, Tidak Resource

#### 4.1 Aturan Utama Routing

- **DILARANG menggunakan `Route::resource()`** — semua route ditulis secara **eksplisit** per method (GET, POST, PUT, DELETE).
- **WAJIB menggunakan Named Routes** — setiap route memiliki nama (`->name('...')`).
- **WAJIB menggunakan HTTP method aslinya** — `Route::get()`, `Route::post()`, `Route::put()`, `Route::delete()`.
- **WAJIB melakukan grouping** pada setiap kelompok route berdasarkan modul/fitur.
- **Penamaan route harus mudah dibaca** — tidak terlalu modular atau terlalu panjang.

#### 4.2 Contoh Route yang Benar

```php
use App\Http\Controllers\DashboardController;
use App\Http\Controllers\JamaahController;
use App\Http\Controllers\PaketHajiController;
use App\Http\Controllers\PembayaranController;
use Illuminate\Support\Facades\Route;

// ======================================================
// Dashboard
// ======================================================
Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');

// ======================================================
// Manajemen Jamaah
// ======================================================
Route::prefix('jamaah')->name('jamaah.')->group(function () {
    Route::get('/', [JamaahController::class, 'index'])->name('index');           // Halaman list jamaah
    Route::get('/tambah', [JamaahController::class, 'create'])->name('create');   // Form tambah jamaah
    Route::post('/simpan', [JamaahController::class, 'store'])->name('store');    // Proses simpan jamaah
    Route::get('/{id}/detail', [JamaahController::class, 'show'])->name('show');  // Detail jamaah
    Route::get('/{id}/edit', [JamaahController::class, 'edit'])->name('edit');     // Form edit jamaah
    Route::put('/{id}/update', [JamaahController::class, 'update'])->name('update'); // Proses update jamaah
    Route::delete('/{id}/hapus', [JamaahController::class, 'destroy'])->name('destroy'); // Proses hapus jamaah
    Route::get('/export', [JamaahController::class, 'export'])->name('export');   // Export data jamaah
    Route::post('/import', [JamaahController::class, 'import'])->name('import');  // Import data jamaah
});

// ======================================================
// Manajemen Paket Haji & Umrah
// ======================================================
Route::prefix('paket-haji')->name('paket-haji.')->group(function () {
    Route::get('/', [PaketHajiController::class, 'index'])->name('index');
    Route::get('/tambah', [PaketHajiController::class, 'create'])->name('create');
    Route::post('/simpan', [PaketHajiController::class, 'store'])->name('store');
    Route::get('/{id}/detail', [PaketHajiController::class, 'show'])->name('show');
    Route::get('/{id}/edit', [PaketHajiController::class, 'edit'])->name('edit');
    Route::put('/{id}/update', [PaketHajiController::class, 'update'])->name('update');
    Route::delete('/{id}/hapus', [PaketHajiController::class, 'destroy'])->name('destroy');
    Route::get('/export', [PaketHajiController::class, 'export'])->name('export');
    Route::post('/import', [PaketHajiController::class, 'import'])->name('import');
});

// ======================================================
// Manajemen Pembayaran
// ======================================================
Route::prefix('pembayaran')->name('pembayaran.')->group(function () {
    Route::get('/', [PembayaranController::class, 'index'])->name('index');
    Route::get('/tambah', [PembayaranController::class, 'create'])->name('create');
    Route::post('/simpan', [PembayaranController::class, 'store'])->name('store');
    Route::get('/{id}/detail', [PembayaranController::class, 'show'])->name('show');
    Route::get('/{id}/edit', [PembayaranController::class, 'edit'])->name('edit');
    Route::put('/{id}/update', [PembayaranController::class, 'update'])->name('update');
    Route::delete('/{id}/hapus', [PembayaranController::class, 'destroy'])->name('destroy');
    Route::post('/{id}/verifikasi', [PembayaranController::class, 'verify'])->name('verify'); // Aksi khusus
});
```

```php
// ❌ SALAH — Jangan pakai resource route
// Route::resource('jamaah', JamaahController::class);
```

---

### 5. Pengelolaan Database & Model

#### 5.1 Database PostgreSQL

- Aplikasi ini menggunakan **PostgreSQL** sebagai database utama.
- Gunakan `ilike` (bukan `like`) untuk pencarian case-insensitive di PostgreSQL.
- Perhatikan perbedaan tipe data PostgreSQL saat menulis migration (misalnya: `jsonb` bukan `json`).

#### 5.2 Migration — Wajib via Artisan

- Semua perubahan skema database **WAJIB** melalui Laravel Migration.
- Jangan manipulasi database secara langsung (raw SQL di tinker, dsb).
- Setiap kali ada migration baru, **WAJIB update** [docs/ERD.md](file:///Users/tnoerman/Herd/hajitrackers/docs/ERD.md).

#### 5.3 Model — Relationship Eksplisit

- Semua **relationship** antar model **WAJIB** didefinisikan secara eksplisit di Model menggunakan return type declaration.
- Gunakan `HasMany`, `BelongsTo`, `HasOne`, `BelongsToMany` secara eksplisit.
- Sertakan **komentar bahasa Indonesia** di setiap relationship method.

#### 5.4 Hindari N+1 Query

- **WAJIB** menggunakan Eager Loading (`with()`) di Controller saat mengambil data yang memiliki relasi.
- Gunakan `->with(['relasi1', 'relasi2.subRelasi'])` di query Controller.

```php
// ✅ BENAR — Eager loading untuk hindari N+1
$jamaahList = Jamaah::with('jamaahPakets.paketHaji')->paginate(10);

// ❌ SALAH — N+1 query (lazy loading di loop)
$jamaahList = Jamaah::paginate(10);
// Lalu di Blade: $jamaah->jamaahPakets (ini query per row!)
```

#### 5.5 Factory & Seeder

- Setiap Model yang dibuat **WAJIB** disertai Factory dan Seeder untuk mendukung testing.

---

### 6. Helper Function — Satu File untuk Fungsi Singkat

Sediakan **1 file helper** (`app/Helpers/helpers.php`) yang berisi fungsi-fungsi singkat & umum. File ini di-load otomatis via `composer.json`.

#### 6.1 Registrasi di `composer.json`

```json
"autoload": {
    "psr-4": {
        "App\\": "app/"
    },
    "files": [
        "app/Helpers/helpers.php"
    ]
}
```

> Setelah menambahkan, jalankan: `composer dump-autoload`

#### 6.2 Contoh Isi Helper

```php
<?php

// =============================================================
// File: app/Helpers/helpers.php
// Kumpulan fungsi singkat yang digunakan di seluruh aplikasi.
// =============================================================

if (!function_exists('formatTanggal')) {
    // Format tanggal ke format Indonesia (contoh: 15 Januari 2025)
    function formatTanggal(?string $tanggal, string $format = 'd F Y'): string
    {
        if (empty($tanggal)) {
            return '-';
        }

        // Parse tanggal dan format sesuai parameter
        return \Carbon\Carbon::parse($tanggal)
            ->locale('id')
            ->translatedFormat($format);
    }
}

if (!function_exists('formatTanggalSingkat')) {
    // Format tanggal singkat (contoh: 15 Jan 2025)
    function formatTanggalSingkat(?string $tanggal): string
    {
        if (empty($tanggal)) {
            return '-';
        }

        return \Carbon\Carbon::parse($tanggal)
            ->locale('id')
            ->translatedFormat('d M Y');
    }
}

if (!function_exists('formatRupiah')) {
    // Format angka ke format Rupiah (contoh: Rp 15.000.000)
    function formatRupiah(float|int|null $nominal): string
    {
        if (is_null($nominal)) {
            return 'Rp 0';
        }

        return 'Rp ' . number_format($nominal, 0, ',', '.');
    }
}

if (!function_exists('formatNomor')) {
    // Format angka dengan pemisah ribuan (contoh: 1.500.000)
    function formatNomor(float|int|null $angka, int $desimal = 0): string
    {
        if (is_null($angka)) {
            return '0';
        }

        return number_format($angka, $desimal, ',', '.');
    }
}

if (!function_exists('potongTeks')) {
    // Potong teks panjang dan tambahkan "..." di akhir
    function potongTeks(?string $teks, int $panjang = 50): string
    {
        if (empty($teks)) {
            return '-';
        }

        return \Illuminate\Support\Str::limit($teks, $panjang);
    }
}

if (!function_exists('formatNomorUrut')) {
    // Format nomor urut dengan leading zero (contoh: 001, 012)
    function formatNomorUrut(int $nomor, int $digit = 3): string
    {
        return str_pad($nomor, $digit, '0', STR_PAD_LEFT);
    }
}

if (!function_exists('hitungUmur')) {
    // Hitung umur berdasarkan tanggal lahir
    function hitungUmur(?string $tanggalLahir): string
    {
        if (empty($tanggalLahir)) {
            return '-';
        }

        return \Carbon\Carbon::parse($tanggalLahir)->age . ' tahun';
    }
}

if (!function_exists('formatFileSize')) {
    // Format ukuran file ke format yang mudah dibaca (KB, MB)
    function formatFileSize(int $bytes): string
    {
        if ($bytes >= 1048576) {
            return number_format($bytes / 1048576, 1) . ' MB';
        }

        return number_format($bytes / 1024, 1) . ' KB';
    }
}
```

---

### 7. Pagination — Bawaan Laravel, Default 10 Row

- Gunakan **pagination bawaan Laravel** (`->paginate(10)`).
- Default data yang ditampilkan: **10 row per halaman**.
- Gunakan `->appends($request->query())` untuk mempertahankan parameter filter saat berpindah halaman.
- Render pagination di Blade menggunakan: `{{ $data->links() }}`

```php
// Di Controller
$jamaahList = Jamaah::with('jamaahPakets')
    ->orderBy('created_at', 'desc')
    ->paginate(10);

// Preserve filter saat pindah halaman
$jamaahList->appends($request->query());
```

```blade
{{-- Di View --}}
<div class="px-4 py-3 border-t border-slate-200/80">
    {{ $jamaahList->links() }}
</div>
```

---

### 8. Struktur View & Standar Desain

#### 8.1 Struktur Folder View

Setiap modul **WAJIB** memiliki folder sendiri di `resources/views/` dengan file-file berikut:

```
resources/views/[modul]/
├── index.blade.php       ← Halaman list/tabel data (WAJIB)
├── create.blade.php      ← Halaman form tambah data (WAJIB)
├── edit.blade.php        ← Halaman form edit data (WAJIB)
└── show.blade.php        ← Halaman detail data (jika diperlukan)
```

- Jika ada fungsi tambahan yang sederhana (misalnya: verifikasi pembayaran, upload dokumen), **prioritaskan penggunaan Modal** di halaman index.
- Jika fungsinya terlalu kompleks untuk modal, buatkan **view terpisah** di folder modul yang sama.

#### 8.2 Standar Desain Wajib

Setiap view **HARUS** mengikuti standar [docs/DESAIN.md](file:///Users/tnoerman/Herd/hajitrackers/docs/DESAIN.md), termasuk:

- **Breadcrumb** — selalu ada di setiap halaman (lihat DESAIN.md Bagian 5)
- **View Header** — judul halaman + deskripsi + action buttons (lihat DESAIN.md Bagian 6)
- **Flash Message / Alert** — menggunakan pola alert standar (lihat DESAIN.md Bagian 7)
- **Filter Section** — filter bar dengan Select2 + pencarian (lihat DESAIN.md Bagian 9)
- **Data Table** — tabel dengan toolbar search, checkbox, pagination (lihat DESAIN.md Bagian 10)
- **Button Standards** — mengikuti standar tombol (lihat DESAIN.md Bagian 16)
- **Modal** — untuk konfirmasi hapus dan form sederhana (lihat DESAIN.md Bagian 13)

#### 8.3 Row Action — Minimal 3 Fungsi

Setiap row data di tabel **WAJIB** memiliki minimal 3 action button:

| Aksi | Warna | Keterangan |
| :--- | :--- | :--- |
| **Detail** | Blue | Lihat detail data (link ke halaman `show`) |
| **Edit** | Amber | Edit data (link ke halaman `edit`) |
| **Hapus** | Rose | Konfirmasi hapus data (SweetAlert2) |

Jika ada fungsi tambahan yang berkaitan dengan 1 row data (misalnya: verifikasi, cetak, upload), masukkan ke **tombol "More" (titik tiga)** sebagai dropdown menu. Referensi: [DESAIN.md Bagian 10.6](file:///Users/tnoerman/Herd/hajitrackers/docs/DESAIN.md).

---

### 9. Filter & Pencarian — Default Wajib

#### 9.1 Aturan Filter

- Secara default, **setiap halaman index WAJIB memiliki fitur pencarian** (search by keyword).
- Setiap data yang **berelasi dengan tabel lain** harus ada filternya. Tentukan filter berdasarkan relasi yang ada di [docs/ERD.md](file:///Users/tnoerman/Herd/hajitrackers/docs/ERD.md).
- Contoh penerapan:

| Halaman | Filter Wajib (berdasarkan relasi) |
| :--- | :--- |
| **Data Jamaah** | Pencarian (nama/NIK), Jenis Kelamin |
| **Pendaftaran (JamaahPaket)** | Pencarian, Paket Haji, Status Pembayaran, Status Keberangkatan |
| **Pembayaran** | Pencarian, Status Verifikasi, Metode Pembayaran, Rentang Tanggal |
| **Dokumen Jamaah** | Pencarian, Jenis Dokumen, Status Dokumen |
| **Paket Haji** | Pencarian, Jenis Paket, Status |

#### 9.2 Implementasi Filter di Controller

```php
// Contoh filter lengkap di JamaahPaketController::index
public function index(Request $request): View
{
    // Buat query dasar dengan eager loading
    $query = JamaahPaket::with(['jamaah', 'paketHaji']);

    // Filter: Pencarian berdasarkan nomor pendaftaran atau nama jamaah
    if ($request->filled('search')) {
        $search = $request->input('search');
        $query->where(function ($q) use ($search) {
            $q->where('nomor_pendaftaran', 'ilike', "%{$search}%")
              ->orWhereHas('jamaah', function ($subQ) use ($search) {
                  $subQ->where('nama_lengkap', 'ilike', "%{$search}%");
              });
        });
    }

    // Filter: Berdasarkan paket haji
    if ($request->filled('paket_haji_id')) {
        $query->where('paket_haji_id', $request->input('paket_haji_id'));
    }

    // Filter: Berdasarkan status pembayaran
    if ($request->filled('status_pembayaran')) {
        $query->where('status_pembayaran', $request->input('status_pembayaran'));
    }

    // Filter: Berdasarkan status keberangkatan
    if ($request->filled('status_keberangkatan')) {
        $query->where('status_keberangkatan', $request->input('status_keberangkatan'));
    }

    // Ambil data dengan pagination 10 row per halaman
    $pendaftaranList = $query->orderBy('created_at', 'desc')->paginate(10);

    // Preserve filter saat pindah halaman
    $pendaftaranList->appends($request->query());

    // Ambil data untuk dropdown filter
    $paketList = PaketHaji::orderBy('nama_paket')->get();

    return view('jamaah-paket.index', compact('pendaftaranList', 'paketList'));
}
```

---

### 10. Import & Export — PhpSpreadsheet di Controller

#### 10.1 Aturan Utama

- Secara default, setiap halaman index yang menampilkan data **WAJIB** menyediakan fungsi **Import** dan **Export**.
- Gunakan library **PhpSpreadsheet** (`phpoffice/phpspreadsheet`).
- Fungsi import/export ditulis **langsung di Controller** — **DILARANG** membuat file Service/Export class terpisah.

#### 10.2 Contoh Export

```php
use PhpOffice\PhpSpreadsheet\Spreadsheet;
use PhpOffice\PhpSpreadsheet\Writer\Xlsx;

// Export data jamaah ke file Excel
public function export(Request $request)
{
    // Buat spreadsheet baru
    $spreadsheet = new Spreadsheet();
    $sheet = $spreadsheet->getActiveSheet();

    // Set judul kolom header
    $sheet->setCellValue('A1', 'No');
    $sheet->setCellValue('B1', 'NIK');
    $sheet->setCellValue('C1', 'Nama Lengkap');
    $sheet->setCellValue('D1', 'Jenis Kelamin');
    $sheet->setCellValue('E1', 'Tanggal Lahir');
    $sheet->setCellValue('F1', 'Nomor HP');

    // Ambil semua data jamaah (dengan filter jika ada)
    $query = Jamaah::orderBy('nama_lengkap');

    // Terapkan filter yang sama seperti di halaman index
    if ($request->filled('jenis_kelamin')) {
        $query->where('jenis_kelamin', $request->input('jenis_kelamin'));
    }

    $jamaahList = $query->get();

    // Isi data ke dalam spreadsheet
    $row = 2;
    foreach ($jamaahList as $index => $jamaah) {
        $sheet->setCellValue('A' . $row, $index + 1);
        $sheet->setCellValue('B' . $row, $jamaah->nik);
        $sheet->setCellValue('C' . $row, $jamaah->nama_lengkap);
        $sheet->setCellValue('D' . $row, $jamaah->jenis_kelamin);
        $sheet->setCellValue('E' . $row, formatTanggal($jamaah->tanggal_lahir));
        $sheet->setCellValue('F' . $row, $jamaah->nomor_hp);
        $row++;
    }

    // Generate file dan kirim sebagai download
    $fileName = 'data-jamaah-' . date('Y-m-d-His') . '.xlsx';
    $writer = new Xlsx($spreadsheet);

    // Set header untuk download
    return response()->streamDownload(function () use ($writer) {
        $writer->save('php://output');
    }, $fileName, [
        'Content-Type' => 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
    ]);
}
```

#### 10.3 Contoh Import

```php
use PhpOffice\PhpSpreadsheet\IOFactory;

// Import data jamaah dari file Excel
public function import(Request $request): RedirectResponse
{
    // Validasi file yang diupload
    $request->validate([
        'file' => 'required|mimes:xlsx,xls,csv|max:5120',
    ], [
        'file.required' => 'File wajib dipilih.',
        'file.mimes'    => 'Format file harus xlsx, xls, atau csv.',
        'file.max'      => 'Ukuran file maksimal 5MB.',
    ]);

    // Baca file spreadsheet yang diupload
    $spreadsheet = IOFactory::load($request->file('file')->getPathname());
    $sheet = $spreadsheet->getActiveSheet();
    $rows = $sheet->toArray();

    // Hapus baris header (baris pertama)
    array_shift($rows);

    // Counter untuk tracking hasil import
    $berhasil = 0;
    $gagal = 0;

    // Proses setiap baris data
    foreach ($rows as $row) {
        // Skip baris kosong
        if (empty($row[0]) && empty($row[1])) {
            continue;
        }

        // Cek apakah NIK sudah ada di database
        $existingJamaah = Jamaah::where('nik', $row[0])->first();
        if ($existingJamaah) {
            $gagal++;
            continue;
        }

        // Simpan data jamaah baru
        Jamaah::create([
            'nik'           => $row[0],
            'nama_lengkap'  => $row[1],
            'jenis_kelamin' => $row[2],
            'tanggal_lahir' => $row[3],
            'nomor_hp'      => $row[4],
        ]);

        $berhasil++;
    }

    // Redirect dengan pesan hasil import
    return redirect()->route('jamaah.index')
        ->with('success', "Import selesai: {$berhasil} data berhasil, {$gagal} data gagal/duplikat.");
}
```

---

### 11. Larangan & Hal yang Harus Dihindari

| No | Larangan | Keterangan |
| :--- | :--- | :--- |
| 1 | **DILARANG** membuat Form Request class | Validasi hanya di Controller |
| 2 | **DILARANG** membuat Service class | Kecuali diminta eksplisit oleh user |
| 3 | **DILARANG** menggunakan `Route::resource()` | Route harus eksplisit per method |
| 4 | **DILARANG** menggunakan magic function / implicit binding | Gunakan `->findOrFail($id)` eksplisit |
| 5 | **DILARANG** menggunakan Livewire / Inertia / Vue / React | Hanya Blade + jQuery + Alpine.js (minimal) |
| 6 | **DILARANG** lazy loading di loop | Selalu eager load dengan `->with()` |
| 7 | **DILARANG** menulis code tanpa komentar | Setiap baris proses harus ada komentar bahasa Indonesia |
| 8 | **DILARANG** menggunakan Alpine.js jika bisa pakai jQuery | Alpine.js hanya untuk hal yang tidak bisa diatasi jQuery |
| 9 | **DILARANG** membuat documentation file | Kecuali diminta eksplisit oleh user |

---

## 📝 PROTOKOL EKSEKUSI AI AGENT (STEP-BY-STEP)

Setiap tugas yang diproses oleh AI Agent harus mengikuti alur kerja ini:

1. **Fase 1 - Pembacaan Dokumen (Reading Context)**
   - Cek `docs/PRD.md` & `docs/STRUKTUR_MENU.md` untuk memahami lingkup & perizinan role.
   - Cek `docs/ERD.md` untuk struktur tabel dan relasinya.
   - Cek `docs/DESAIN.md` untuk memastikan tampilan sesuai standar UI/UX.
   - Cek `GEMINI.md` ini untuk memastikan kepatuhan standar teknis.

2. **Fase 2 - Eksekusi Code (Development)**
   - Buat migration, model, controller, dan view sesuai standar di atas.
   - Validasi langsung di Controller — **jangan buat Form Request terpisah**.
   - Route eksplisit — **jangan pakai Route::resource()**.
   - Semua logika di Controller — **jangan buat Service class**.
   - Komentar bahasa Indonesia di setiap baris proses.
   - Eager loading untuk semua query yang memiliki relasi.
   - Filter & pencarian di setiap halaman index.
   - Tombol import/export di setiap halaman index.

3. **Fase 3 - Pengujian (Testing)**
   - Buat/jalankan test Pest yang relevan (`php artisan test --compact`).

4. **Fase 4 - Pembaruan Dokumentasi (Updating Docs)**
   - Update `docs/ERD.md` jika ada perubahan DB.
   - Update `docs/FITUR_APLIKASI.md` (Diagram ASCII, alur proses, status data).
   - Update `docs/STRUKTUR_MENU.md` jika ada penambahan menu/role.
   - Update `docs/DOKUMENTASI_PENGGUNAAN.md` (Manual book bahasa awam).
   - Update `docs/UAT.md` (Skenario pengujian).
   - Catat semua perubahan di `docs/TASK_DONE.md`.

---

## 🗓️ FASE PENGEMBANGAN (Timeline AI Agent)

> **Panduan bagi AI Agent**: Section ini adalah **acuan utama urutan pengerjaan**. AI Agent **WAJIB** mengerjakan sesuai urutan fase dan modul yang tertera. Setiap item yang selesai harus dicatat di `docs/TASK_DONE.md`.

> **Referensi Arsitektur**: Hirarki 4-Tier (Tahun → Event → Program → Paket + Add-on). Detail lengkap di [docs/FITUR_APLIKASI.md](file:///Users/tnoerman/Herd/hajitrackers/docs/FITUR_APLIKASI.md) dan [docs/ERD.md](file:///Users/tnoerman/Herd/hajitrackers/docs/ERD.md).

---

### FASE 1 — FOUNDATION (Harus Ada Dulu)

> Seluruh fondasi sistem: autentikasi, master data, hierarki produk, data jamaah, dan pendaftaran.

#### Modul 1: Autentikasi & RBAC

**1.1 Database & Model**
- Migration `create_roles_table` — tabel role (`super-admin`, `admin-operasional`, `verifikator-keuangan`, `marketing`, `tour-leader`, `pimpinan`, `jamaah`)
- Migration `create_role_user_table` — tabel pivot many-to-many user ↔ role
- Migration `create_login_logs_table` — audit trail login (IP, user_agent, method, status)
- Migration `update_users_table` — tambah field `user_type` (staff/jamaah), `is_active`, `last_login_at`, `username`
- Model `Role` — relasi `belongsToMany(User::class)`
- Model `LoginLog` — relasi `belongsTo(User::class)`
- Update Model `User` — relasi `belongsToMany(Role::class)`, method `hasRole()`, `hasAnyRole()`
- Seeder `RoleSeeder` — seed 7 role default
- Seeder `UserSeeder` — seed 1 Super Admin default
- Factory `UserFactory` — support `user_type` staff/jamaah

**1.2 Middleware & Auth**
- Middleware `RoleMiddleware` — cek role user sebelum akses route
- Middleware `EnsureUserIsActive` — cek `is_active` user
- Konfigurasi Laravel Auth — login staff (username/email + password)
- Login Jamaah — NIK/Kode Jamaah/Nomor HP + PIN (DDMMYYYY)
- Pencatatan otomatis ke `login_logs` setiap login (berhasil/gagal)

**1.3 Controller & View**
- `AuthController` — login, logout, redirect per role
- `UserManagementController` — CRUD user staff (Super Admin only)
  - Index (list + filter + pagination)
  - Create/Store (form + validasi)
  - Edit/Update
  - Delete (soft delete)
  - Toggle aktif/nonaktif
  - Assign/remove role
- View Blade: `auth/login.blade.php`, `users/index.blade.php`, `users/create.blade.php`, `users/edit.blade.php`
- Layout Blade: `layouts/app.blade.php` — sidebar navigasi per role
- SweetAlert2 untuk notifikasi sukses/error
- Login As (Impersonation) — fitur Super Admin masuk ke sesi user lain + banner sticky

**1.4 Testing**
- Test login staff (berhasil/gagal)
- Test login jamaah via NIK + PIN
- Test middleware role
- Test CRUD user management
- Test impersonation

---

#### Modul 2: Master Data (Tier 1 — Library Reusable)

**2.1 Master Rangkaian Kegiatan**
- Migration `create_master_rangkaians_table` — kode, nama, kategori (manasik/mcu/lainnya), urutan_default, status
- Model `MasterRangkaian` — fillable, casts, relasi `hasMany(EventRangkaian::class)`
- Factory + Seeder — 6 data seed (Pra Manasik 1-4, Manasik Akbar, MCU)
- `MasterRangkaianController` — CRUD lengkap
  - Index (list + filter kategori + pencarian nama + pagination)
  - Create/Store (validasi: kode unik, nama wajib)
  - Edit/Update
  - Delete (cek apakah sudah dipakai event → jika ya, hanya nonaktifkan)
- View Blade: `master-rangkaian/index.blade.php`, `create.blade.php`, `edit.blade.php`
- Test CRUD + test proteksi hapus jika sudah dipakai

**2.2 Master Syarat Dokumen**
- Migration `create_master_dokumens_table` — kode, nama, kategori, default_wajib, format_file_allowed, max_size_mb, status
- Model `MasterDokumen` — fillable, casts, relasi `hasMany(EventDokumen::class)`
- Factory + Seeder — 9 data seed (KTP, KK, Paspor, Vaksin, Foto, BPIH, Biometrik, Visa, Buku Nikah)
- `MasterDokumenController` — CRUD lengkap
  - Index (list + filter kategori + pencarian + pagination)
  - Create/Store (validasi: kode unik)
  - Edit/Update
  - Delete (proteksi jika sudah dipakai)
- View Blade: `master-dokumen/index.blade.php`, `create.blade.php`, `edit.blade.php`
- Test CRUD

**2.3 Master Perlengkapan**
- Migration `create_master_perlengkapans_table` — kode, nama, perlu_ukuran (boolean), opsi_ukuran (jsonb), status
- Model `MasterPerlengkapan` — fillable, casts (opsi_ukuran → array), relasi `hasMany(EventPerlengkapan::class)`
- Factory + Seeder — 16 data seed (Jaket, Gamis, Koko, dst.)
- `MasterPerlengkapanController` — CRUD lengkap
  - Index (list + filter perlu_ukuran + pencarian + pagination)
  - Create/Store (validasi: kode unik, opsi_ukuran wajib jika perlu_ukuran=true)
  - Edit/Update
  - Delete (proteksi jika sudah dipakai)
- View Blade: `master-perlengkapan/index.blade.php`, `create.blade.php`, `edit.blade.php`
- Test CRUD

**2.4 Master Hotel**
- Migration `create_master_hotels_table` — kode, nama_hotel, kota (enum), bintang (1-5), alamat, jarak_ke_masjid, status
- Model `MasterHotel` — fillable, casts, relasi `hasMany(EventHotel::class)`
- Factory + Seeder — data hotel contoh (Pullman ZamZam ★5, Grand Mercure ★4, dll)
- `MasterHotelController` — CRUD lengkap
  - Index (list + filter kota + filter bintang + pencarian + pagination)
  - Create/Store (validasi: kode unik, bintang 1-5)
  - Edit/Update
  - Delete (proteksi jika sudah dipakai)
- View Blade: `master-hotel/index.blade.php`, `create.blade.php`, `edit.blade.php`
- Test CRUD

---

#### Modul 2: Tahun, Event, Program, Paket (Tier 0-3 — Hierarki Produk)

**2.5 Tahun Keberangkatan (Tier 0)**
- Migration `create_tahun_keberangkatans_table` — tahun_masehi (unique), tahun_hijriah (string), label, status
- Model `TahunKeberangkatan` — fillable, relasi `hasMany(EventKeberangkatan::class)`
- Factory + Seeder — seed tahun 2026 & 2027
- `TahunKeberangkatanController` — CRUD
  - Index (list + filter status + pagination)
  - Create/Store (validasi: tahun_masehi unik)
  - Edit/Update
  - Delete (cek apakah punya event → jika ya, tolak)
- View Blade: `tahun-keberangkatan/index.blade.php`, `create.blade.php`, `edit.blade.php`
- Test CRUD

**2.6 Event Keberangkatan (Tier 1)**
- Migration `create_event_keberangkatans_table` — tahun_keberangkatan_id (FK), kode_event (UK), nama_event, jenis_event (haji/umrah), tanggal_berangkat, tanggal_pulang, program_tarwiyah_tersedia, status, catatan
- Model `EventKeberangkatan` — fillable, casts, relasi:
  - `belongsTo(TahunKeberangkatan::class)`
  - `hasMany(EventProgram::class)`
  - `hasMany(Paket::class)`
  - `hasMany(EventRangkaian::class)`, `hasMany(EventDokumen::class)`, `hasMany(EventPerlengkapan::class)`, `hasMany(EventHotel::class)`
- Factory + Seeder
- `EventKeberangkatanController` — CRUD + Publish
  - Index (list per tahun + filter jenis/status + pencarian + pagination)
  - Create/Store (validasi: tanggal_pulang > tanggal_berangkat, kode auto-generate)
  - Show/Detail (tab-based: info umum, program, paket, setting)
  - Edit/Update
  - Delete (cek ada jamaah → jika ya, hanya cancel)
  - Publish (Draft → Active) — validasi checklist (min 1 paket, rangkaian, dokumen, perlengkapan, hotel)
- View Blade: `event-keberangkatan/index.blade.php`, `create.blade.php`, `show.blade.php` (tab-based), `edit.blade.php`
- Test CRUD + test publish checklist

**2.7 Event Program (Tier 2 — Opsional, Khusus Haji)**
- Migration `create_event_programs_table` — event_keberangkatan_id (FK), nama_program, deskripsi, urutan, status
- Model `EventProgram` — fillable, relasi `belongsTo(EventKeberangkatan::class)`, `hasMany(Paket::class)`
- `EventProgramController` — CRUD (inline di halaman show event)
  - Store/Update/Delete program di bawah event
  - Validasi: hanya event jenis_event=haji yang bisa punya program
- View: komponen partial di `event-keberangkatan/show.blade.php` tab "Program"
- Test CRUD

**2.8 Paket (Tier 3)**
- Migration `create_pakets_table` — event_keberangkatan_id (FK), event_program_id (FK, nullable), kode_paket (UK), nama_paket, jenis (enum 6 nilai), is_arbain (boolean), kategori_hotel (integer), harga_dasar (decimal), kuota, kuota_terisi (default 0), status, catatan
- Model `Paket` — fillable, casts, relasi:
  - `belongsTo(EventKeberangkatan::class)`
  - `belongsTo(EventProgram::class)` (nullable)
  - `hasMany(PaketAddon::class)`
  - `hasMany(JamaahEvent::class)`
- Factory + Seeder
- `PaketController` — CRUD
  - Index per event/program
  - Create/Store (validasi: jenis enum, kategori_hotel 1-5, harga_dasar > 0, kuota > 0, kode auto-generate)
  - Edit/Update
  - Delete (cek ada jamaah → jika ya, hanya cancel)
- View: komponen partial di `event-keberangkatan/show.blade.php` tab "Paket"
- Test CRUD + test kuota update

**2.9 Paket Add-on (Surcharge Pricing)**
- Migration `create_paket_addons_table` — paket_id (FK), kategori (enum: upgrade_seat/upgrade_kamar/upgrade_hotel_transit), nama_addon, harga_tambahan (decimal), kuota (nullable), kuota_terisi (default 0), status
- Model `PaketAddon` — fillable, casts, relasi `belongsTo(Paket::class)`, `hasMany(JamaahEventAddon::class)`
- `PaketAddonController` — CRUD inline di halaman paket detail
  - Store/Update/Delete add-on per paket
  - Validasi: harga_tambahan >= 0, kategori valid
- View: komponen partial di detail paket
- Test CRUD

**2.10 Setting Event (4 Kategori)**
- Migration `create_event_rangkaians_table` — event_keberangkatan_id, master_rangkaian_id, urutan, tanggal_pelaksanaan, lokasi, is_wajib
- Migration `create_event_dokumens_table` — event_keberangkatan_id, master_dokumen_id, is_wajib, catatan
- Migration `create_event_perlengkapans_table` — event_keberangkatan_id, master_perlengkapan_id, is_wajib, catatan
- Migration `create_event_hotels_table` — event_keberangkatan_id, master_hotel_id, catatan
- Migration `create_event_hotel_kamars_table` — event_hotel_id, tipe_kamar (enum), kuota, kuota_terisi, harga_tambahan, catatan
- Model: `EventRangkaian`, `EventDokumen`, `EventPerlengkapan`, `EventHotel`, `EventHotelKamar` — masing-masing dengan relasi FK
- Controller per setting — CRUD inline (add/remove/edit di halaman show event, tab-based)
  - `EventRangkaianController` — pick dari master, atur urutan + tanggal + lokasi
  - `EventDokumenController` — pick dari master, override wajib/opsional
  - `EventPerlengkapanController` — pick dari master, atur wajib/opsional
  - `EventHotelController` — pick dari master + atur tipe kamar per hotel
- View: tab-based di `event-keberangkatan/show.blade.php`
  - Tab "Rangkaian Kegiatan"
  - Tab "Syarat Dokumen"
  - Tab "Perlengkapan"
  - Tab "Hotel & Kamar"
- Test CRUD per setting + test kuota hotel kamar

---

#### Modul 3: Manajemen Jamaah (Biodata)

**3.1 Database & Model**
- Migration `create_jamaahs_table` — semua field biodata (NIK encrypted, nama, gender, TTL, paspor, kontak, TC/PIC, BPJS, kode_etis, dll)
- Model `Jamaah` — fillable, casts (tanggal_lahir → date), relasi:
  - `belongsTo(User::class, 'user_id')` (nullable)
  - `belongsTo(User::class, 'created_by')`
  - `hasMany(JamaahEvent::class)`
- Factory `JamaahFactory` — faker data realistis
- Seeder `JamaahSeeder` — seed 20-50 jamaah contoh

**3.2 Controller & View**
- `JamaahController` — CRUD lengkap
  - Index (list + search nama/NIK ilike + filter gender/TC/BPJS/paket + pagination 10/page)
  - Create/Store (validasi: NIK unik 16 digit, TTL < hari ini, pesan error bahasa Indonesia)
  - Show/Detail (biodata + tab pendaftaran paket terkait)
  - Edit/Update (NIK unique except self)
  - Delete (cek ada pendaftaran → jika ya, tolak / soft delete)
  - Export Excel (PhpSpreadsheet)
  - Import Excel (PhpSpreadsheet + validasi per row)
- View Blade: `jamaah/index.blade.php`, `create.blade.php`, `show.blade.php`, `edit.blade.php`
- NIK masking di UI (`3275************`) + tombol 👁️ untuk role berwenang
- Warning masa berlaku paspor < 6 bulan

**3.3 Testing**
- Test CRUD jamaah
- Test NIK unique validation
- Test import/export Excel
- Test NIK masking display

---

#### Modul 4: Pendaftaran Jamaah ke Paket Event

**4.1 Database & Model**
- Migration `create_jamaah_events_table` — jamaah_id (FK), paket_id (FK), nomor_pendaftaran (UK), tanggal_daftar, nomor_porsi, harga_dasar_snapshot, total_addon, total_tagihan, total_terbayar, sisa_tagihan, status_pembayaran (enum), status_keberangkatan (enum), program_tarwiyah, + semua field operasional (visa, tiket, bus, kamar, dll)
- Migration `create_jamaah_event_addons_table` — jamaah_event_id (FK), paket_addon_id (FK), harga_saat_dipilih, tanggal_dipilih, dipilih_oleh (FK users)
- Model `JamaahEvent` — fillable, casts, relasi:
  - `belongsTo(Jamaah::class)`
  - `belongsTo(Paket::class)`
  - `hasMany(JamaahEventAddon::class)`
  - `hasMany(Pembayaran::class)`
  - `hasMany(JamaahEventDokumen::class)`, `hasMany(JamaahEventKehadiran::class)`, `hasMany(JamaahEventPerlengkapan::class)`, `hasMany(JamaahEventHotelPilihan::class)`
- Model `JamaahEventAddon` — fillable, relasi `belongsTo(JamaahEvent::class)`, `belongsTo(PaketAddon::class)`
- Factory + Seeder

**4.2 Controller & View**
- `PendaftaranController` — CRUD
  - Index (list pendaftaran + filter event/tahun/status + pencarian jamaah + pagination)
  - Create (step: pilih jamaah → pilih tahun → pilih event → pilih program (jika haji) → pilih paket → opsional add-on)
  - Store (validasi: jamaah belum daftar di event sama, kuota paket tersedia, event Active)
    - Auto-generate nomor pendaftaran `REG-{TAHUN}-{5 DIGIT}`
    - Snapshot harga_dasar dari paket
    - Kuota paket++ (`kuota_terisi`)
    - Auto-create: `jamaah_event_dokumens`, `jamaah_event_kehadirans`, `jamaah_event_perlengkapans` berdasarkan setting event
  - Show/Detail (tab: info pendaftaran, pembayaran, dokumen, rangkaian, perlengkapan, preferensi, operasional)
  - Edit/Update (update status keberangkatan, tambah/hapus add-on → recalculate tagihan)
  - Cancel (batalkan pendaftaran → kuota paket--, status → batal)
- View Blade: `pendaftaran/index.blade.php`, `create.blade.php`, `show.blade.php` (tab-based), `edit.blade.php`

**4.3 Testing**
- Test pendaftaran sukses + auto-create records
- Test duplikat pendaftaran ditolak
- Test kuota paket update
- Test pembatalan + kuota berkurang
- Test harga_dasar_snapshot
- Test add-on → recalculate total_tagihan

---

### FASE 2 — TRANSAKSI & DOKUMEN

> Pembayaran, verifikasi keuangan, upload dokumen, dan dashboard overview.

#### Modul 5: Pembayaran & Verifikasi

**5.1 Database & Model**
- Migration `create_pembayarans_table` — jamaah_event_id (FK), kode_transaksi (UK), jenis_bayar (enum: dp/cicilan/pelunasan), jumlah_bayar, tanggal_bayar, metode_pembayaran, bukti_pembayaran (file), status_verifikasi (enum), verified_by (FK), verified_at, catatan_verifikasi
- Model `Pembayaran` — fillable, casts, relasi `belongsTo(JamaahEvent::class)`, `belongsTo(User::class, 'verified_by')`
- Factory + Seeder

**5.2 Controller & View**
- `PembayaranController` — CRUD + Verifikasi
  - Index (list pembayaran + filter event/status_verifikasi/periode + pencarian jamaah + pagination)
  - Create/Store (input: jamaah_event, jenis bayar, nominal, bukti transfer → status: Pending)
    - Auto-generate kode transaksi `TRX-{YYYYMMDD}-{5 DIGIT}`
  - Show/Detail (info pembayaran + bukti + riwayat verifikasi)
  - Verify (Verified/Rejected oleh Verifikator Keuangan)
    - Jika Verified → update `total_terbayar`, `sisa_tagihan`, `status_pembayaran` di `jamaah_events`
    - Jika Rejected → catatan penolakan
  - Delete (hanya Super Admin, hanya status Pending)
- View Blade: `pembayaran/index.blade.php`, `create.blade.php`, `show.blade.php`, `verify.blade.php`

**5.3 Testing**
- Test create pembayaran + kode transaksi auto
- Test verifikasi → update total_terbayar
- Test status_pembayaran auto-update (belum_bayar → dp → cicilan → lunas)
- Test reject + catatan

---

#### Modul 6: Dokumen Jamaah

**6.1 Database & Model**
- Migration `create_jamaah_event_dokumens_table` — jamaah_event_id (FK), event_dokumen_id (FK), file_path, status (enum: belum_upload/pending/valid/invalid), catatan, verified_by, uploaded_at, verified_at
- Model `JamaahEventDokumen` — fillable, casts, relasi `belongsTo(JamaahEvent::class)`, `belongsTo(EventDokumen::class)`, `belongsTo(User::class, 'verified_by')`

**6.2 Controller & View**
- `DokumenJamaahController` — Upload + Verifikasi
  - Index per jamaah_event (list dokumen + status per jenis)
  - Upload (admin/jamaah upload file → status: Pending)
    - Validasi: maks 2MB, format jpg/jpeg/png/pdf
    - Upload baru mengganti file lama
  - Verify (Valid/Invalid oleh admin/verifikator)
    - Invalid → catatan feedback penolakan
  - Progress kelengkapan: `(Valid) / (Wajib) × 100%`
- View: komponen partial di `pendaftaran/show.blade.php` tab "Dokumen"

**6.3 Testing**
- Test upload file + validasi ukuran/format
- Test verifikasi valid/invalid
- Test progress calculation
- Test upload ulang (replace lama)

---

#### Modul 7: Dashboard

**7.1 Controller & View**
- `DashboardController` — view per role
  - Dashboard Super Admin / Pimpinan:
    - Stat cards: total jamaah aktif, total paket aktif, pendapatan bulan ini, outstanding keuangan
    - Chart: pendapatan vs target (12 bulan), statistik per jenis paket (pie/donut)
    - Progress kelengkapan dokumen per event
    - Tabel jamaah mendekati keberangkatan (T-30)
    - Tabel pembayaran menunggu verifikasi
  - Dashboard Admin Operasional:
    - Jamaah terbaru hari ini
    - Tugas verifikasi dokumen (Pending)
    - Pembayaran baru
    - Paket mendekati keberangkatan
  - Dashboard Verifikator Keuangan:
    - Count & list pembayaran Pending
    - Total verifikasi hari ini
    - Outstanding per event
    - Pembayaran ditolak (perlu follow up)
  - Dashboard Jamaah (Portal):
    - Paket saya, status pembayaran (progress bar), dokumen (checklist), tracking timeline
- View Blade: `dashboard/index.blade.php` (conditional per role)
- Chart.js atau library chart ringan

**7.2 Testing**
- Test dashboard load per role
- Test statistik calculation benar

---

### FASE 3 — OPERASIONAL PERJALANAN

> Kegiatan pra-keberangkatan, perlengkapan, preferensi, dan data operasional lapangan.

#### Modul 8: Rangkaian Pra Jamaah

**8.1 Database & Model**
- Migration `create_jamaah_event_kehadirans_table` — jamaah_event_id (FK), event_rangkaian_id (FK), status_kehadiran (enum: belum/hadir/tidak_hadir/izin), catatan, updated_by
- Model `JamaahEventKehadiran` — fillable, casts, relasi

**8.2 Controller & View**
- `RangkaianPraJamaahController`
  - Index per event (list kegiatan + jumlah hadir per kegiatan)
  - Detail per kegiatan (list jamaah + status kehadiran masing-masing)
  - Update kehadiran (individual per jamaah)
  - Bulk update kehadiran (update banyak jamaah sekaligus untuk satu kegiatan)
  - Progress checklist per jamaah
- View: komponen di `pendaftaran/show.blade.php` tab "Rangkaian" + halaman dedicated per event

**8.3 Testing**
- Test update kehadiran individual
- Test bulk update kehadiran
- Test progress calculation

---

#### Modul 9: Perlengkapan Jamaah

**9.1 Database & Model**
- Migration `create_jamaah_event_perlengkapans_table` — jamaah_event_id (FK), event_perlengkapan_id (FK), ukuran (nullable), sudah_diterima (boolean default false), tanggal_diterima (nullable), catatan
- Model `JamaahEventPerlengkapan` — fillable, casts, relasi

**9.2 Controller & View**
- `PerlengkapanJamaahController`
  - Index per event (list item + jumlah diterima per item)
  - Detail per item (list jamaah + ukuran + status diterima)
  - Input ukuran per jamaah (jika perlu_ukuran = true)
  - Update penerimaan (centang diterima + tanggal)
  - Bulk update penerimaan (update banyak jamaah sekaligus)
  - Progress penerimaan per jamaah
  - Filter gender (item relevan per gender)
- View: komponen di `pendaftaran/show.blade.php` tab "Perlengkapan" + halaman dedicated per event

**9.3 Testing**
- Test input ukuran
- Test update penerimaan
- Test bulk update
- Test gender filter

---

#### Modul 10: Preferensi Jamaah

**10.1 Database & Model**
- Migration `create_jamaah_event_hotel_pilihans_table` — jamaah_event_id (FK), event_hotel_kamar_id (FK)
- Model `JamaahEventHotelPilihan` — fillable, relasi

**10.2 Controller & View**
- `PreferensiJamaahController`
  - Form edit preferensi per jamaah_event:
    - Pilihan kamar per hotel (dropdown, terikat kuota)
    - Pilihan add-on upgrade (seat, kamar, transit) — dari `paket_addons`
    - Program Tarwiyah (toggle ya/tidak)
    - Request note (free text)
  - Store/Update preferensi
    - Recalculate `total_tagihan` saat add-on berubah
    - Update kuota kamar hotel + kuota add-on
    - Simpan riwayat upgrade di `jamaah_event_addons`
- View: komponen di `pendaftaran/show.blade.php` tab "Preferensi"

**10.3 Testing**
- Test pilih kamar + kuota update
- Test add-on + recalculate tagihan
- Test paket asal tidak berubah setelah upgrade

---

#### Modul 11: Data Operasional & Manifest

**11.1 Controller & View**
- `OperasionalController`
  - Form edit data operasional per jamaah_event:
    - Visa: nomor + status (belum/proses/issued)
    - Tiket pesawat: nomor + status (belum/booked/issued)
    - Flight: departure + arrival
    - Kereta Haramain: nomor
    - Bus: nomor
    - Kamar/Tower: nomor
    - Paspor fisik: tanggal penyerahan
  - Bulk update (bus, kamar — banyak jamaah sekaligus)
- `ManifestController`
  - Generate manifest per event
  - Filter: per paket, per bus, per status keberangkatan
  - Export Excel (PhpSpreadsheet)
  - Export PDF
- View: `operasional/edit.blade.php`, `manifest/index.blade.php`

**11.2 Testing**
- Test update data operasional
- Test bulk update bus/kamar
- Test manifest generate + filter
- Test export Excel/PDF

---

### FASE 4 — PORTAL & LAPORAN

> Portal mandiri jamaah (ramah lansia) dan sistem laporan/export.

#### Modul 12: Self Login Jamaah (Portal Mandiri)

**12.1 Auth & Routing**
- Route group khusus portal jamaah (`/portal`)
- Login portal: Nomor HP + PIN 6 digit
- Middleware: hanya role `jamaah`, hanya data sendiri
- Session long-lived (30 hari)
- Layout khusus: font besar, tombol besar, navigasi minimal (max 5 menu)

**12.2 Controller & View**
- `PortalController`
  - Beranda Saya — ringkasan: paket, status bayar (progress bar besar), dokumen (checklist), countdown keberangkatan
  - Tagihan & Bayar — total tagihan, sisa bayar, riwayat pembayaran, upload bukti bayar
  - Dokumen Saya — checklist status, upload foto dokumen per item
  - Status Perjalanan — timeline tracking 7 tahap (otomatis berdasarkan data modul terkait)
  - Profil — data pribadi (read-only) + ganti PIN
- View Blade: `portal/beranda.blade.php`, `portal/tagihan.blade.php`, `portal/dokumen.blade.php`, `portal/tracking.blade.php`, `portal/profil.blade.php`
- UI/UX: font min 16px, tombol min 48px, icon + teks, bahasa sederhana ("Kirim Foto" bukan "Upload")

**12.3 Admin: Pembuatan Akun Jamaah**
- Fitur di halaman jamaah: "Buatkan Akun Login"
  - Generate PIN 6 digit random
  - Link user ke jamaah (`user_id`)
  - Admin kirim kredensial via WA/SMS

**12.4 Testing**
- Test login portal HP + PIN
- Test akses hanya data sendiri
- Test upload bukti bayar dari portal
- Test upload dokumen dari portal
- Test tracking timeline otomatis

---

#### Modul 13: Laporan & Export

**13.1 Controller & View**
- `LaporanController`
  - Rekap Jamaah per Event (Excel, PDF) — biodata + paket + status
  - Manifest Keberangkatan (Excel, PDF) — paspor + visa + bus + kamar + flight
  - Laporan Keuangan (Excel, PDF) — pendapatan, outstanding, per periode
  - Outstanding per Jamaah (Excel) — sisa tagihan per jamaah per event
  - Status Dokumen (Excel) — kelengkapan per jamaah per event
  - Rekap Perlengkapan (Excel) — ukuran + status penerimaan
  - Rekap Kehadiran Manasik (Excel) — kehadiran pra manasik 1-4 + akbar
  - Rekap MCU (Excel) — status MCU + istithaah
  - Statistik per Periode (Excel, PDF) — total jamaah, pendapatan per bulan/tahun
- Filter: per event, per paket, per periode, per status pembayaran/keberangkatan, per TC/PIC
- Export engine: PhpSpreadsheet (Excel) + library PDF
- Nama file: `{nama_laporan}_{tanggal_export}.xlsx`
- RBAC: laporan keuangan hanya Verifikator + Pimpinan

**13.2 Testing**
- Test generate + download per laporan
- Test filter applied correctly
- Test RBAC akses laporan

---

### FASE 5 — ADVANCED (Dibahas Nanti)

> Modul-modul advanced yang akan dibahas di iterasi berikutnya setelah Fase 1-4 stabil.

#### Modul 14: QR Code Tracking
- Scope: Scan QR untuk kehadiran manasik, MCU, distribusi perlengkapan, check-in keberangkatan
- Belum dikerjakan — menunggu keputusan: 1 QR per jamaah vs per event, mekanisme scan

#### Modul 15: Follow Up Sales
- Scope: CRM follow up calon jamaah oleh tim sales/marketing
- Belum dikerjakan — menunggu keputusan: CRM sederhana vs pencatatan history lengkap

---

### Catatan Penting untuk AI Agent

1. **Urutan pengerjaan WAJIB sesuai fase** — Fase 1 harus selesai sebelum lanjut ke Fase 2.
2. **Dalam satu fase, urutan modul fleksibel** — tapi perhatikan dependency (Modul 2 harus sebelum Modul 4).
3. **Setiap modul selesai** → catat di `docs/TASK_DONE.md`, update `docs/ERD.md` jika ada migration baru.
4. **Testing wajib per modul** — jalankan `php artisan test --compact` setelah setiap modul.
5. **Format code** — jalankan `vendor/bin/pint --dirty --format agent` setelah setiap perubahan PHP.
6. **Referensi utama**: [docs/FITUR_APLIKASI.md](file:///Users/tnoerman/Herd/hajitrackers/docs/FITUR_APLIKASI.md) untuk alur bisnis, [docs/ERD.md](file:///Users/tnoerman/Herd/hajitrackers/docs/ERD.md) untuk skema database, [docs/DESAIN.md](file:///Users/tnoerman/Herd/hajitrackers/docs/DESAIN.md) untuk standar UI/UX.

---
> Source: [aliwafa16/safarstrack](https://github.com/aliwafa16/safarstrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
