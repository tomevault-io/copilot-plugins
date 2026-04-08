## sipos

> > File ini adalah sumber kebenaran tunggal (_single source of truth_) untuk AI agent

# CLAUDE.md — AI Agent Context: SIPOS

> File ini adalah sumber kebenaran tunggal (_single source of truth_) untuk AI agent
> yang bekerja pada proyek **SIPOS (Sistem Informasi Posyandu)**. Baca seluruh file
> ini sebelum melakukan perubahan apapun pada codebase.

---

## Daftar Isi

1. [Identitas Proyek](#1-identitas-proyek)
2. [Domain & Konteks Bisnis](#2-domain--konteks-bisnis)
3. [Stack Teknologi](#3-stack-teknologi)
4. [Arsitektur & Pola Desain](#4-arsitektur--pola-desain)
5. [Struktur Direktori](#5-struktur-direktori)
6. [Domain Model & Database](#6-domain-model--database)
7. [Aktor & Permission](#7-aktor--permission)
8. [Konvensi Kode](#8-konvensi-kode)
9. [Alur Kerja Development](#9-alur-kerja-development)
10. [Perintah Penting](#10-perintah-penting)
11. [Testing](#11-testing)
12. [Aturan Wajib untuk AI Agent](#12-aturan-wajib-untuk-ai-agent)
13. [Kesalahan Umum yang Harus Dihindari](#13-kesalahan-umum-yang-harus-dihindari)
14. [Referensi Cepat](#14-referensi-cepat)

---

## 1. Identitas Proyek

```
Nama Aplikasi : SIPOS (Sistem Informasi Posyandu)
Nama Singkat  : sipos
Versi         : 1.0.0
Lokasi        : Posyandu Desa Belumbang, Kec. Kerambitan, Kab. Tabanan, Bali
Tujuan        : Digitalisasi pencatatan & pelaporan kesehatan posyandu
Repository    : github.com/[org]/sipos
Branch Utama  : main (production) | develop (staging) | feature/* (development)
```

---

## 2. Domain & Konteks Bisnis

### 2.1 Apa itu Posyandu?

**Posyandu (Pos Pelayanan Terpadu)** adalah layanan kesehatan masyarakat berbasis komunitas yang dikelola oleh kader (sukarelawan) dan diawasi bidan. Di Desa Belumbang menggunakan model **ILP-HIU** (Integrasi Layanan Primer — Holistik Integrasi dan Unggul).

### 2.2 Skala Operasional

| Entitas         | Jumlah | Keterangan                   |
| --------------- | ------ | ---------------------------- |
| Posyandu        | 8      | Masing-masing 1 per dusun    |
| Bidan           | 2      | Tenaga kesehatan profesional |
| Kader           | 56     | 7 per posyandu               |
| Ibu Hamil Aktif | ~7     | Per periode                  |
| Balita Aktif    | ~100   | Per periode                  |
| Lansia Aktif    | ~74    | Per periode                  |

### 2.3 Alur Bisnis Utama

```
1. PENDAFTARAN
   Peserta datang → Kader cek (baru/lama)
   Baru  → Kader input identitas peserta ke sistem
   Lama  → Kader cari data peserta di sistem

2. PEMERIKSAAN
   Kader input: BB, TB, lingkar kepala/perut/lengan, tensi
   Bidan input: edukasi, resep obat, catatan medis

3. PELAPORAN
   Sistem generate laporan bulanan → Dikirim ke Puskesmas
   Peserta menerima KMS (Kartu Menuju Sehat) digital

4. PENJADWALAN
   Kader usulkan jadwal → Bidan validasi → Notifikasi ke peserta
```

### 2.4 Masalah yang Dipecahkan

- ❌ **Sebelum:** Pencatatan manual di buku → data hilang/rusak, laporan lambat
- ✅ **Sesudah:** Sistem digital → data tersimpan aman, laporan instan, notifikasi otomatis

### 2.5 Istilah Domain Penting

| Istilah           | Makna dalam Kode                                                   |
| ----------------- | ------------------------------------------------------------------ |
| `ibu_hamil`       | Peserta hamil yang terdaftar di posyandu                           |
| `balita`          | Anak usia 0–5 tahun terdaftar                                      |
| `lansia`          | Warga usia lanjut (≥60 tahun) terdaftar                            |
| `kader`           | Operator posyandu (bukan tenaga medis)                             |
| `bidan`           | Tenaga medis yang memvalidasi & mencatat hasil medis               |
| `peserta`         | Role user untuk ibu hamil / orang tua balita / keluarga lansia     |
| `posyandu`        | Unit pelayanan (1 dusun = 1 posyandu)                              |
| `jadwal_posyandu` | Sesi kegiatan posyandu (per event, bukan entitas posyandu)         |
| `pemeriksaan`     | Record hasil pemeriksaan kesehatan per kunjungan                   |
| `KMS`             | Kartu Menuju Sehat — rekam tumbuh-kembang digital                  |
| `LILA`            | Lingkar Lengan Atas — indikator gizi ibu hamil (KEK jika < 23.5cm) |
| `KEK`             | Kekurangan Energi Kronik                                           |
| `TTD`             | Tablet Tambah Darah                                                |
| `NIK`             | Nomor Induk Kependudukan (16 digit, ID nasional)                   |

---

## 3. Stack Teknologi

### Backend

```
PHP              8.3+
Laravel          11.x (dengan konvensi terbaru: bootstrap/app.php)
Laravel Breeze   2.x  (Inertia stack — Vue)
Inertia Laravel  2.x
MariaDB          11.x
Redis            7.x  (cache, queue, session)
Pest PHP         3.x  (testing framework)
Laravel Pint     1.x  (code style — PSR-12 + Laravel preset)
PHPStan/Larastan 2.x  (static analysis, level 8)
```

### Frontend

```
Vue.js           3.x  (Composition API + <script setup> + TypeScript)
Inertia.js       2.x  (SSR adapter untuk Vue)
Vite             6.x
Tailwind CSS     4.x
shadcn-vue       latest (komponen UI utama)
Pinia            3.x  (state management)
VeeValidate      4.x  + Zod (form validation)
Chart.js         4.x  + vue-chartjs (grafik pertumbuhan)
Lucide Vue Next  latest (icons)
@vueuse/core     12.x (composable utilities)
TypeScript       5.x
```

### Infrastructure

```
Docker           + Docker Compose
Nginx            1.25-alpine (web server)
Supervisor       (queue worker + scheduler)
```

### Package PHP Penting

```
spatie/laravel-permission    — RBAC
spatie/laravel-query-builder — filter/sort API
spatie/laravel-activitylog   — audit trail
barryvdh/laravel-dompdf      — export PDF
maatwebsite/excel            — export Excel
```

---

## 4. Arsitektur & Pola Desain

### 4.1 Layered Architecture

```
HTTP Request
    ↓
[Middleware] → rate limit, auth, role check
    ↓
[Controller] → validasi input, panggil service, return Inertia response
    ↓
[Service] → business logic, orchestration, event dispatch
    ↓
[Repository] → data access abstraction (interface + eloquent implementation)
    ↓
[Model] → Eloquent ORM, relations, scopes, casts
    ↓
[Database] → MariaDB
```

### 4.2 Pola Wajib

| Pola                   | Di mana                         | Tujuan                                       |
| ---------------------- | ------------------------------- | -------------------------------------------- |
| **Repository Pattern** | `app/Repositories/`             | Abstraksi akses data, mudah di-mock          |
| **Service Layer**      | `app/Services/`                 | Isolasi business logic dari controller       |
| **DTO**                | `app/DTOs/`                     | Transfer data antar layer dengan type-safety |
| **Form Request**       | `app/Http/Requests/`            | Validasi + otorisasi di layer HTTP           |
| **API Resource**       | `app/Http/Resources/`           | Transformasi data untuk response Inertia     |
| **Policy**             | `app/Policies/`                 | Otorisasi per-resource berbasis role         |
| **Event/Listener**     | `app/Events/`, `app/Listeners/` | Decoupling side effects                      |
| **Observer**           | `app/Observers/`                | Audit log otomatis                           |
| **Enum**               | `app/Enums/`                    | Nilai tetap (role, status, tipe)             |

### 4.3 Aturan Dependency Flow

```
Controller → Service → Repository → Model
Controller TIDAK BOLEH langsung query Model
Service TIDAK BOLEH tahu tentang HTTP Request
Repository TIDAK BOLEH berisi business logic
```

### 4.4 Inertia.js Flow

```
Laravel Controller
  → Inertia::render('PageName/Component', ['prop' => $data])
  → Diterima Vue component sebagai defineProps<Props>()
  → Router menggunakan router.visit() / <Link> / router.post()
  → Tidak ada fetch() manual ke Laravel endpoint
```

---

## 5. Struktur Direktori

```
sipos/
├── app/
│   ├── Console/Commands/          # Artisan commands (e.g. SendJadwalNotification)
│   ├── DTOs/                      # Data Transfer Objects (readonly class)
│   ├── Enums/                     # PHP 8.1+ backed enums
│   │   ├── UserRole.php           # admin|bidan|kader|peserta
│   │   ├── PesertaType.php        # ibu_hamil|balita|lansia
│   │   └── JadwalStatus.php       # draft|validated|rejected|completed
│   ├── Events/                    # Domain events
│   ├── Exceptions/
│   ├── Http/
│   │   ├── Controllers/           # Thin controllers — hanya HTTP concerns
│   │   ├── Middleware/
│   │   ├── Requests/              # Form Requests (validasi + otorisasi)
│   │   └── Resources/             # API Resources (Inertia response transformation)
│   ├── Listeners/                 # Event listeners
│   ├── Models/                    # Eloquent models
│   ├── Notifications/             # Laravel Notifications
│   ├── Observers/                 # Model observers (audit)
│   ├── Policies/                  # Authorization policies
│   ├── Providers/
│   │   ├── AppServiceProvider.php
│   │   └── RepositoryServiceProvider.php  # Bind interface → implementation
│   ├── Repositories/
│   │   ├── Interfaces/            # Contracts (interface)
│   │   └── Eloquent/              # Implementasi Eloquent
│   │       └── BaseRepository.php # Shared CRUD logic
│   └── Services/                  # Business logic
│
├── database/
│   ├── factories/                 # Model factories (Pest testing)
│   ├── migrations/                # Database migrations (chronological)
│   └── seeders/
│       ├── DatabaseSeeder.php
│       ├── UserSeeder.php         # Default users per role
│       ├── PosyanduSeeder.php     # 8 posyandu Desa Belumbang
│       └── DemoDataSeeder.php     # Data demo (dev only)
│
├── docker/
│   ├── dev/                       # Dev Dockerfile + configs
│   ├── prod/                      # Prod Dockerfile + configs
│   ├── mariadb/init.sql
│   └── secrets/                   # ⚠️ GITIGNORED — credential files
│
├── resources/js/
│   ├── Components/
│   │   ├── ui/                    # shadcn-vue components (auto-generated)
│   │   ├── Charts/                # Chart.js wrapper components
│   │   └── Shared/                # Reusable app components
│   ├── Composables/               # Vue composables (useConfirm, useToast, dll)
│   ├── Layouts/
│   │   ├── AppLayout.vue          # Layout utama (authenticated)
│   │   └── AuthLayout.vue         # Layout login
│   ├── Pages/                     # Inertia pages (1:1 dengan route)
│   │   ├── Auth/
│   │   ├── Dashboard.vue
│   │   ├── IbuHamil/              # Index, Create, Edit, Show
│   │   ├── Balita/
│   │   ├── Lansia/
│   │   ├── Pemeriksaan/
│   │   ├── Jadwal/
│   │   ├── Laporan/
│   │   └── Users/
│   ├── stores/                    # Pinia stores
│   └── types/
│       └── index.d.ts             # TypeScript type definitions
│
├── routes/
│   ├── web.php                    # Semua Inertia routes
│   └── auth.php                   # Auth routes (via Breeze)
│
├── tests/
│   ├── Feature/                   # HTTP + DB tests
│   └── Unit/                      # Isolated class tests
│
├── docker-compose.dev.yml
├── docker-compose.prod.yml
├── CLAUDE.md                      # ← FILE INI
└── SRS_SIPOS_Posyandu_Belumbang.md
```

---

## 6. Domain Model & Database

### 6.1 Primary Key Convention

**Semua tabel menggunakan ULID** (bukan auto-increment integer atau UUID standar).

```php
// Di setiap Model:
use Illuminate\Database\Eloquent\Concerns\HasUlids;

class IbuHamil extends Model
{
    use HasUlids;
    // ...
}
```

Alasan: ULID sortable secara leksikal, URL-safe, dan tidak mengekspos jumlah record.

### 6.2 Soft Delete

Semua tabel entitas utama menggunakan `SoftDeletes`. Data tidak pernah dihapus permanen kecuali via command eksplisit admin.

### 6.3 Tabel & Relasi Utama

```
users           → kader (1:1), bidan (1:1)
users           → ibu_hamil (1:1, nullable), balita (1:1, nullable), lansia (1:1, nullable)
posyandu        → jadwal_posyandu (1:M)
jadwal_posyandu → pemeriksaan (1:M)
kader           → jadwal_posyandu (M:M via posyandu_kader pivot)
pemeriksaan     → ibu_hamil / balita / lansia (POLYMORPHIC via peserta_type + peserta_id)
users           → notifications (polymorphic, Laravel default)
```

### 6.4 Polymorphic Pemeriksaan

Tabel `pemeriksaan` melayani 3 jenis peserta menggunakan polymorphic relation:

```php
// Model Pemeriksaan
public function peserta(): MorphTo
{
    return $this->morphTo();
}

// Model IbuHamil
public function pemeriksaan(): MorphMany
{
    return $this->morphMany(Pemeriksaan::class, 'peserta');
}
```

Kolom `peserta_type` berisi nilai: `'ibu_hamil'`, `'balita'`, atau `'lansia'`  
Kolom `peserta_id` berisi ULID dari record yang bersangkutan.

### 6.5 Enum Values

```php
// app/Enums/UserRole.php
enum UserRole: string {
    case Admin   = 'admin';
    case Bidan   = 'bidan';
    case Kader   = 'kader';
    case Peserta = 'peserta';
}

// app/Enums/JadwalStatus.php
enum JadwalStatus: string {
    case Draft     = 'draft';
    case Validated = 'validated';
    case Rejected  = 'rejected';
    case Completed = 'completed';
}

// app/Enums/PesertaType.php
enum PesertaType: string {
    case IbuHamil = 'ibu_hamil';
    case Balita   = 'balita';
    case Lansia   = 'lansia';
}
```

### 6.6 Casts Model Wajib

```php
// Gunakan selalu di model yang relevan:
protected $casts = [
    'tgl_lahir'         => 'date',
    'tgl_pemeriksaan'   => 'date',
    'is_active'         => 'boolean',
    'obat_cacing'       => 'boolean',
    'role'              => UserRole::class,
    'status'            => JadwalStatus::class,
    'peserta_type'      => PesertaType::class,
    'deleted_at'        => 'datetime',
];
```

---

## 7. Aktor & Permission

### 7.1 Role Hierarchy

```
admin
  └── Full access ke seluruh sistem + user management
bidan
  └── Input pemeriksaan, validasi jadwal, cetak laporan, lihat semua data
kader
  └── CRUD peserta, input pemeriksaan, usul jadwal, cetak laporan
peserta
  └── Lihat data kesehatan sendiri + KMS digital + jadwal (read-only)
```

### 7.2 Permission Matrix (Ringkas)

| Resource           | admin | bidan | kader | peserta |
| ------------------ | :---: | :---: | :---: | :-----: |
| users CRUD         |  ✅   |  ❌   |  ❌   |   ❌    |
| kader/bidan CRUD   |  ✅   |  ❌   |  ❌   |   ❌    |
| posyandu CRUD      |  ✅   |  ❌   |  ❌   |   ❌    |
| ibu_hamil CRUD     |  ✅   |  ✅   |  ✅   |   ❌    |
| balita CRUD        |  ✅   |  ✅   |  ✅   |   ❌    |
| lansia CRUD        |  ✅   |  ✅   |  ✅   |   ❌    |
| pemeriksaan CRUD   |  ✅   |  ✅   |  ✅   |   ❌    |
| jadwal — buat/edit |  ✅   |  ✅   |  ✅   |   ❌    |
| jadwal — validasi  |  ✅   |  ✅   |  ❌   |   ❌    |
| laporan — cetak    |  ✅   |  ✅   |  ✅   |   ❌    |
| data diri sendiri  |  ✅   |  ✅   |  ✅   |   ✅    |
| KMS digital        |  ✅   |  ✅   |  ✅   |   ✅    |
| audit log          |  ✅   |  ❌   |  ❌   |   ❌    |

### 7.3 Implementasi Policy

Setiap resource **wajib** memiliki Policy. Gunakan `authorize()` di controller:

```php
// Di controller
public function store(StoreIbuHamilRequest $request): RedirectResponse
{
    // Otorisasi sudah ada di StoreIbuHamilRequest::authorize()
    // atau eksplisit:
    $this->authorize('create', IbuHamil::class);
}
```

---

## 8. Konvensi Kode

### 8.1 PHP / Laravel

#### Naming

```
Model            → PascalCase singular    : IbuHamil, JadwalPosyandu
Controller       → PascalCase + Controller: IbuHamilController
Service          → PascalCase + Service   : IbuHamilService
Repository       → PascalCase + Repository: IbuHamilRepository
Interface        → PascalCase + ...Interface: IbuHamilRepositoryInterface
Form Request     → Verb + Resource + Request: StoreIbuHamilRequest
DTO              → Verb + Resource + DTO  : CreateIbuHamilDTO
Policy           → Resource + Policy      : IbuHamilPolicy
Event            → PascalCase past tense  : IbuHamilCreated
Listener         → PascalCase action      : SendWelcomeNotification
Enum             → PascalCase             : UserRole
Migration        → timestamp_action_table : 2024_01_01_000001_create_users_table
```

#### Method Naming di Service

```php
public function paginate(array $filters): LengthAwarePaginator  // untuk list
public function findById(string $id): Model                     // single record
public function create(CreateXxxDTO $dto): Model                // buat baru
public function update(string $id, UpdateXxxDTO $dto): Model    // update
public function delete(string $id): bool                        // hapus (soft)
public function restore(string $id): bool                       // restore soft-deleted
```

#### Controller harus tipis (thin)

```php
// ✅ BENAR — Controller hanya koordinasi
public function store(StoreIbuHamilRequest $request): RedirectResponse
{
    $dto = CreateIbuHamilDTO::fromRequest($request);
    $this->ibuHamilService->create($dto);
    return redirect()->route('ibu-hamil.index')->with('success', 'Data berhasil disimpan.');
}

// ❌ SALAH — Logic bisnis di controller
public function store(Request $request): RedirectResponse
{
    $data = $request->validate([...]);
    if ($data['usia_kehamilan'] > 42) { ... }  // ← ini harusnya di Service
    IbuHamil::create($data);                   // ← ini harusnya di Repository
}
```

#### DTO harus `readonly`

```php
// ✅ BENAR
readonly class CreateIbuHamilDTO
{
    public function __construct(
        public string $nik,
        public string $nama,
        // ...
    ) {}

    public static function fromRequest(StoreIbuHamilRequest $request): self
    {
        return new self(nik: $request->validated('nik'), ...);
    }
}
```

#### Gunakan Enum, bukan magic string

```php
// ✅ BENAR
$user->role === UserRole::Kader
$jadwal->status === JadwalStatus::Validated

// ❌ SALAH
$user->role === 'kader'
$jadwal->status === 'validated'
```

#### Eager Loading — hindari N+1

```php
// ✅ BENAR
IbuHamil::with(['pemeriksaan' => fn($q) => $q->latest()->limit(1)])->paginate(15);

// ❌ SALAH
$ibuHamil = IbuHamil::paginate(15);
foreach ($ibuHamil as $item) {
    $item->pemeriksaan->last(); // N+1!
}
```

### 8.2 Vue.js / TypeScript

#### Selalu gunakan `<script setup lang="ts">`

```vue
<script setup lang="ts">
import { ref, computed } from "vue";
import { router } from "@inertiajs/vue3";

interface Props {
  ibuHamil: PaginatedResponse<IbuHamil>;
  filters: { search?: string };
}

const props = defineProps<Props>();
</script>
```

#### Type definitions di `resources/js/types/index.d.ts`

```typescript
// Semua tipe domain didefinisikan di sini
export interface IbuHamil {
  id: string;
  nik: string;
  nama: string;
  tglLahir: string;
  usiaKehamilan: number;
  noTelp: string;
  alamat: string;
  isActive: boolean;
  createdAt: string;
}

export interface PaginatedResponse<T> {
  data: T[];
  meta: PaginationMeta;
  links: PaginationLinks;
}
```

#### Navigasi menggunakan Inertia router

```typescript
// ✅ BENAR
import { router, Link } from '@inertiajs/vue3'
router.post(route('ibu-hamil.store'), formData)
router.delete(route('ibu-hamil.destroy', id))

// ❌ SALAH — jangan gunakan fetch/axios langsung
fetch('/ibu-hamil', { method: 'POST', body: ... })
```

#### Gunakan `route()` helper (ziggy)

```typescript
// ✅ BENAR
router.visit(route("ibu-hamil.show", { id: ibuHamil.id }));

// ❌ SALAH — hardcode URL
router.visit(`/ibu-hamil/${ibuHamil.id}`);
```

### 8.3 Database / Migration

```php
// ✅ BENAR — selalu gunakan ULID, soft delete, timestamps
Schema::create('ibu_hamil', function (Blueprint $table) {
    $table->ulid('id')->primary();
    $table->foreignUlid('user_id')->nullable()->constrained()->nullOnDelete();
    $table->string('nik', 16)->unique();
    $table->string('nama', 100);
    // ... kolom domain ...
    $table->boolean('is_active')->default(true);
    $table->timestamps();
    $table->softDeletes();
});

// Selalu tambahkan index pada kolom yang sering di-query
$table->index('nik');
$table->index('is_active');
```

### 8.4 Testing (Pest PHP)

```php
// ✅ BENAR — deskriptif, coverage bisnis
it('kader dapat mendaftarkan ibu hamil baru', function () {
    $kader = User::factory()->kader()->create();

    actingAs($kader)
        ->post(route('ibu-hamil.store'), IbuHamilFactory::validData())
        ->assertRedirect(route('ibu-hamil.index'))
        ->assertSessionHas('success');

    assertDatabaseHas('ibu_hamil', ['nik' => '5101234567890001']);
});

it('peserta tidak dapat mendaftarkan ibu hamil', function () {
    $peserta = User::factory()->peserta()->create();

    actingAs($peserta)
        ->post(route('ibu-hamil.store'), IbuHamilFactory::validData())
        ->assertForbidden();
});

// Gunakan dataset untuk multiple scenarios
it('memvalidasi NIK', function (string $nik, bool $valid) {
    $kader = User::factory()->kader()->create();
    $response = actingAs($kader)->post(route('ibu-hamil.store'), ['nik' => $nik, ...]);
    $valid
        ? $response->assertSessionMissing('errors.nik')
        : $response->assertSessionHasErrors('nik');
})->with([
    'valid NIK'        => ['5101234567890001', true],
    'NIK terlalu pendek' => ['123', false],
    'NIK bukan angka'  => ['ABCDEFGHIJ123456', false],
]);
```

---

## 9. Alur Kerja Development

### 9.1 Menambah Fitur Baru

Ikuti urutan ini **tanpa pengecualian**:

```
1. Migration    → buat/update skema database
2. Model        → tambah casts, relations, scopes
3. DTO          → buat CreateXxxDTO dan UpdateXxxDTO
4. Repository   → interface + eloquent implementation
5. Service      → business logic
6. Form Request → validasi + otorisasi
7. Policy       → otorisasi per-resource
8. Controller   → thin, koordinasi saja
9. Resource     → transformasi response
10. Route       → tambahkan di routes/web.php
11. Vue Pages   → Pages/Xxx/Index.vue, Create.vue, Edit.vue, Show.vue
12. Tests       → Feature test + Unit test
```

### 9.2 Bind Repository di Service Provider

Setiap interface **wajib** di-bind di `RepositoryServiceProvider`:

```php
// app/Providers/RepositoryServiceProvider.php
public function register(): void
{
    $this->app->bind(
        IbuHamilRepositoryInterface::class,
        IbuHamilRepository::class
    );
    // tambahkan binding baru di sini
}
```

### 9.3 Register Event & Listener

```php
// app/Providers/AppServiceProvider.php
protected $listen = [
    IbuHamilCreated::class => [LogAuditTrail::class],
    PemeriksaanRecorded::class => [UpdateKMSCache::class],
    JadwalValidated::class => [SendJadwalNotification::class],
];
```

### 9.4 Checklist Sebelum Commit

- [ ] `php artisan test` — semua test hijau
- [ ] `./vendor/bin/pint` — code style bersih
- [ ] `./vendor/bin/phpstan analyse` — tidak ada error level 8
- [ ] Tidak ada `dd()`, `dump()`, `var_dump()`, `console.log()` tersisa
- [ ] Tidak ada hardcoded string yang seharusnya Enum
- [ ] Eager loading sudah diterapkan (tidak ada N+1)
- [ ] Migration reversible (ada `down()` method)
- [ ] Pesan flash dalam Bahasa Indonesia

---

## 10. Perintah Penting

### Development (via Docker)

```bash
# Start environment
docker compose -f docker-compose.dev.yml up -d

# Akses shell container
docker exec -it sipos_app_dev sh

# Jalankan migration
docker exec sipos_app_dev php artisan migrate

# Fresh migration + seed (dev only)
docker exec sipos_app_dev php artisan migrate:fresh --seed

# Buat component shadcn-vue baru
docker exec sipos_app_dev npx shadcn-vue@latest add button

# Jalankan Vite dev server (hot reload)
docker exec sipos_app_dev npm run dev

# Artisan tinker
docker exec -it sipos_app_dev php artisan tinker

# Lihat semua routes
docker exec sipos_app_dev php artisan route:list

# Clear semua cache
docker exec sipos_app_dev php artisan optimize:clear
```

### Generate File Baru

```bash
# Model + migration + factory + seeder
php artisan make:model NamaModel -mfs

# Controller resource
php artisan make:controller NamaController --resource --model=NamaModel

# Form Request
php artisan make:request StoreNamaRequest
php artisan make:request UpdateNamaRequest

# Policy
php artisan make:policy NamaPolicy --model=NamaModel

# Event + Listener
php artisan make:event NamaEvent
php artisan make:listener NamaListener --event=NamaEvent

# Job (queue)
php artisan make:job NamaJob

# Notification
php artisan make:notification NamaNotification

# Resource
php artisan make:resource NamaResource
```

### Testing

```bash
# Semua test
php artisan test

# Dengan coverage HTML
php artisan test --coverage-html=coverage/

# Test spesifik
php artisan test tests/Feature/IbuHamil/
php artisan test --filter="kader dapat mendaftarkan"

# Parallel (lebih cepat)
php artisan test --parallel
```

### Production

```bash
# Deploy
docker compose -f docker-compose.prod.yml up -d --build

# Jalankan migration production
docker exec sipos_app php artisan migrate --force

# Optimize (cache config, route, view)
docker exec sipos_app php artisan optimize

# Restart queue worker
docker exec sipos_app php artisan queue:restart
```

---

## 11. Testing

### 11.1 Struktur Test

```
tests/
├── Feature/                     # HTTP tests (dengan DB)
│   ├── Auth/
│   │   ├── LoginTest.php
│   │   └── LogoutTest.php
│   ├── IbuHamil/
│   │   ├── CreateIbuHamilTest.php
│   │   ├── UpdateIbuHamilTest.php
│   │   ├── DeleteIbuHamilTest.php
│   │   └── ListIbuHamilTest.php
│   ├── Balita/
│   ├── Lansia/
│   ├── Pemeriksaan/
│   ├── Jadwal/
│   └── Laporan/
└── Unit/                        # Isolated tests (tanpa DB)
    ├── Services/
    │   ├── PemeriksaanServiceTest.php
    │   └── LaporanServiceTest.php
    ├── DTOs/
    └── Helpers/
```

### 11.2 Test Environment

File `.env.testing`:

```dotenv
APP_ENV=testing
DB_CONNECTION=mariadb
DB_DATABASE=sipos_test   # DB terpisah dari dev
CACHE_STORE=array
QUEUE_CONNECTION=sync
SESSION_DRIVER=array
MAIL_MAILER=array
```

### 11.3 Factories — State yang Tersedia

```php
// User factory states
User::factory()->admin()->create()
User::factory()->bidan()->create()
User::factory()->kader()->create()
User::factory()->peserta()->create()

// Dengan relasi
User::factory()->kader()->has(Kader::factory())->create()
```

### 11.4 Target Coverage

| Layer        | Target    |
| ------------ | --------- |
| Services     | ≥ 90%     |
| Repositories | ≥ 85%     |
| Controllers  | ≥ 80%     |
| Models       | ≥ 75%     |
| **Overall**  | **≥ 80%** |

---

## 12. Aturan Wajib untuk AI Agent

> Baca bagian ini dengan sangat teliti. Ini adalah batasan keras (_hard rules_)
> yang tidak boleh dilanggar dalam kondisi apapun.

### WAJIB DILAKUKAN ✅

1. **Baca file ini terlebih dahulu** sebelum menulis kode apapun di sesi baru.

2. **Ikuti urutan layered architecture** — jangan skip layer. Jika diminta tambah fitur, selalu mulai dari migration → model → DTO → repository → service → controller → vue page.

3. **Gunakan ULID** sebagai primary key di semua tabel baru. Gunakan trait `HasUlids` di model.

4. **Gunakan Enum** untuk semua nilai tetap (role, status, tipe). Jangan hardcode string.

5. **Tulis test** untuk setiap fitur baru. Minimum: 1 happy path + 1 unauthorized access test per endpoint.

6. **Bahasa Indonesia** untuk: pesan flash (`with('success', '...')`), pesan validasi override, label form di Vue.

7. **Eager loading** di setiap query yang mengakses relasi. Selalu review apakah ada potensi N+1.

8. **Gunakan Form Request** untuk semua input HTTP — jangan `$request->validate()` inline di controller.

9. **Bind interface** di `RepositoryServiceProvider` untuk setiap repository baru.

10. **Soft delete** di semua entitas domain utama (ibu_hamil, balita, lansia, users, kader, bidan, posyandu).

11. **Konsultasi SRS** (`SRS_SIPOS_Posyandu_Belumbang.md`) jika ada keraguan tentang field database, alur bisnis, atau permission.

### DILARANG KERAS ❌

1. **Jangan query Model langsung dari Controller** — selalu lewat Service.

2. **Jangan taruh business logic di Controller atau Model** — taruh di Service.

3. **Jangan gunakan `$request->all()` atau `$request->input()` tanpa validasi** — selalu gunakan `$request->validated()`.

4. **Jangan hardcode ID, URL, atau string domain** — gunakan Enum, route helper, dan config.

5. **Jangan buat migrasi yang merusak data** tanpa backup strategy dan `--force` guard.

6. **Jangan expose port database dan Redis ke host di production** (docker-compose.prod.yml).

7. **Jangan simpan kredensial di kode** — semua sensitive config via `.env` atau Docker secrets.

8. **Jangan hapus data permanen** via UI — hanya soft delete. Hard delete hanya boleh via Artisan command dengan konfirmasi eksplisit.

9. **Jangan gunakan `fetch()` atau `axios` langsung di Vue** — gunakan Inertia `router` atau `useForm`.

10. **Jangan tambah kolom langsung di tabel production** — selalu lewat migration baru.

11. **Jangan lewati Policy** — semua operasi CRUD wajib melalui `authorize()` atau Policy di Form Request.

12. **Jangan commit** jika ada test yang fail.

---

## 13. Kesalahan Umum yang Harus Dihindari

### PHP / Laravel

```php
// ❌ SALAH: Logic di controller
class IbuHamilController extends Controller {
    public function store(Request $request) {
        $data = $request->validate([...]);
        if ($data['usia_kehamilan'] > 42) {
            return back()->withError('Usia kehamilan tidak valid');
        }
        $ibuHamil = IbuHamil::create($data); // langsung query model
        return redirect()->route('ibu-hamil.index');
    }
}

// ✅ BENAR
class IbuHamilController extends Controller {
    public function store(StoreIbuHamilRequest $request): RedirectResponse {
        $dto = CreateIbuHamilDTO::fromRequest($request);
        $this->ibuHamilService->create($dto);
        return redirect()->route('ibu-hamil.index')->with('success', 'Data berhasil disimpan.');
    }
}
```

```php
// ❌ SALAH: Magic string
if ($user->role === 'kader') { ... }

// ✅ BENAR
if ($user->role === UserRole::Kader) { ... }
```

```php
// ❌ SALAH: N+1 query
$records = Pemeriksaan::all();
foreach ($records as $r) {
    echo $r->ibuHamil->nama; // N+1!
}

// ✅ BENAR
$records = Pemeriksaan::with('peserta')->get();
```

### Vue.js

```typescript
// ❌ SALAH: fetch langsung
const response = await fetch("/ibu-hamil", {
  method: "POST",
  body: JSON.stringify(form),
});

// ✅ BENAR: Inertia router
router.post(route("ibu-hamil.store"), form, {
  onSuccess: () => toast.success("Data berhasil disimpan"),
});
```

```typescript
// ❌ SALAH: any type
const ibuHamil: any = props.ibuHamil;

// ✅ BENAR: typed
const props = defineProps<{ ibuHamil: PaginatedResponse<IbuHamil> }>();
```

### Database

```php
// ❌ SALAH: integer primary key
$table->id(); // auto-increment integer

// ✅ BENAR: ULID
$table->ulid('id')->primary();
```

```php
// ❌ SALAH: tanpa soft delete di entitas utama
Schema::create('ibu_hamil', function (Blueprint $table) {
    $table->ulid('id')->primary();
    // ... tanpa softDeletes()
    $table->timestamps();
});

// ✅ BENAR
Schema::create('ibu_hamil', function (Blueprint $table) {
    $table->ulid('id')->primary();
    // ...
    $table->timestamps();
    $table->softDeletes(); // ← wajib
});
```

---

## 14. Referensi Cepat

### Route Names

```
auth.login              → Halaman login
auth.logout             → Logout (POST)
dashboard               → Dashboard utama

ibu-hamil.index         → List ibu hamil
ibu-hamil.create        → Form tambah
ibu-hamil.store         → POST simpan
ibu-hamil.show          → Detail + riwayat
ibu-hamil.edit          → Form edit
ibu-hamil.update        → PUT update
ibu-hamil.destroy       → DELETE

balita.*                → (sama seperti ibu-hamil.*)
lansia.*                → (sama seperti ibu-hamil.*)

pemeriksaan.index
pemeriksaan.create
pemeriksaan.store
pemeriksaan.show
pemeriksaan.edit
pemeriksaan.update
pemeriksaan.destroy

jadwal.index
jadwal.store
jadwal.show
jadwal.update
jadwal.destroy
jadwal.validate         → POST (bidan)
jadwal.reject           → POST (bidan)
jadwal.complete         → POST

laporan.index
laporan.ibu-hamil
laporan.balita
laporan.lansia
laporan.bulanan
laporan.export.pdf      → POST
laporan.export.excel    → POST

users.index
users.create
users.store
users.edit
users.update
users.destroy

notifications.index
notifications.read      → POST (single)
notifications.read-all  → POST
```

### File Penting yang Sering Diedit

```
app/Providers/RepositoryServiceProvider.php   → tambah binding repo baru
app/Providers/AppServiceProvider.php          → tambah event/listener
routes/web.php                                → tambah route baru
resources/js/types/index.d.ts                 → tambah tipe domain baru
database/seeders/DatabaseSeeder.php           → tambah seeder baru
```

### Variabel Shared Inertia (via HandleInertiaRequests)

```php
// app/Http/Middleware/HandleInertiaRequests.php
// Props ini tersedia di SEMUA halaman Vue:
'auth' => [
    'user'          => $request->user(),
    'role'          => $request->user()?->role,
    'permissions'   => [...],
],
'flash' => [
    'success' => session('success'),
    'error'   => session('error'),
    'warning' => session('warning'),
],
'unreadNotifications' => (int) $request->user()?->unreadNotifications()->count(),
```

### Akses di Vue

```typescript
import { usePage } from "@inertiajs/vue3";
const page = usePage();
const user = page.props.auth.user;
const flash = page.props.flash;
```

### Default Credentials (Dev/Seeder)

```
Admin  : username=admin,      password=password
Bidan  : username=bidan1,     password=password
Kader  : username=kader1,     password=password
Peserta: username=peserta1,   password=password
```

> ⚠️ Ubah semua password ini sebelum deploy ke production.

---

_Dokumen ini harus selalu up-to-date dengan kondisi aktual codebase._  
_Setiap perubahan arsitektur, penambahan teknologi, atau perubahan konvensi wajib diperbarui di sini._

**Versi Dokumen:** 1.0.0 | **Diperbarui:** Maret 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stefanfredik)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/stefanfredik)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
