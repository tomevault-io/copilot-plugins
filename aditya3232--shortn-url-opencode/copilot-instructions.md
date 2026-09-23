## shortn-url-opencode

> Dokumen ini berisi aturan yang wajib dipatuhi oleh AI Coding Agent ketika mengembangkan project **shortnurl-service**.

# AGENTS.md

# Shortnurl Service - AI Coding Agent Guidelines

Dokumen ini berisi aturan yang wajib dipatuhi oleh AI Coding Agent ketika mengembangkan project **shortnurl-service**.

---

# 1. Project Context & Environment

## Tech Stack

- Language: Go (Golang)
- Framework: Fiber v2
- ORM: GORM
- Database: PostgreSQL
- Cache: Redis
- Logger: Zerolog
- Containerization: Docker & Docker Compose

---

## Architecture

Project menggunakan pendekatan Clean Architecture yang disederhanakan.

```
Handler
    ↓
Service
    ↓
Repository
    ↓
Database
```

Setiap layer memiliki tanggung jawab masing-masing.

### Handler

Bertanggung jawab untuk:

- parsing request
- validasi request
- memanggil service
- mengembalikan response

Handler **tidak boleh** berisi business logic maupun query database.

---

### Service

Berisi seluruh business logic.

Contohnya:

- validasi URL
- generate short code
- rate limiting
- validasi expiration
- validasi ownership
- pencatatan analytics
- dan seluruh aturan bisnis lainnya.

---

### Repository

Repository hanya bertanggung jawab terhadap akses database.

Repository tidak boleh berisi business logic.

---

## Folder Structure

AI wajib mengikuti struktur project berikut.

```
cmd/
config/
database/
internal/
    adapter/
    app/
    core/
    middleware/
utils/
```

Jangan membuat struktur baru apabila masih dapat menggunakan struktur yang sudah ada.

---

## Coding Style

Ikuti standar Go.

- gunakan `gofmt`
- gunakan penamaan idiomatik Go
- receiver menggunakan nama pendek
- hindari package yang terlalu besar
- hindari fungsi yang terlalu panjang

Usahakan satu fungsi hanya memiliki satu tanggung jawab (Single Responsibility Principle).

---

## Dependency Injection

Seluruh dependency dibuat melalui constructor.

Contoh:

```go
func NewURLService(repo repository.URLRepository) URLService
```

Jangan membuat dependency menggunakan global variable apabila tidak benar-benar diperlukan.

---

## Error Handling

Gunakan error wrapping.

```go
return fmt.Errorf("create url: %w", err)
```

Jangan mengabaikan error.

Jangan menggunakan panic untuk flow normal aplikasi.

---

## Logging

Gunakan Zerolog.

Logging digunakan untuk:

- error
- warning
- informasi penting

Jangan melakukan logging terhadap:

- password
- token
- authorization header
- secret key
- database credential

---

## Database

Gunakan GORM.

Gunakan transaction hanya ketika memang dibutuhkan.

Selalu gunakan Context.

Contoh:

```go
db.WithContext(ctx)
```

---

## API Response

Gunakan response yang konsisten.

Contoh:

```json
{
    "success": true,
    "message": "Success",
    "data": {}
}
```

Error response juga harus menggunakan format yang sama.

---

## Validation

Seluruh validasi request dilakukan di layer Handler.

Business validation dilakukan di layer Service.

---

## Context

Seluruh function yang melakukan IO wajib menerima context.Context.

Contoh:

```go
func (s *urlService) Create(ctx context.Context, req entity.URLEntity) error
```

---

# 2. Code Quality & Security

## Security

Selalu asumsikan seluruh input berasal dari user yang tidak dipercaya.

Lakukan validasi terhadap:

- URL
- query parameter
- JSON body
- path parameter
- header

---

## SQL Injection

Selalu gunakan parameter binding dari GORM.

Jangan membuat query menggunakan string concatenation.

Benar:

```go
db.Where("id = ?", id)
```

Salah:

```go
db.Where("id = " + id)
```

---

## XSS

Jangan mengembalikan HTML dari input user tanpa sanitasi.

---

## URL Validation

Pastikan URL:

- memiliki skema http atau https
- valid
- bukan localhost
- bukan private IP (kecuali memang diperbolehkan)

---

## Secret

Jangan pernah hardcode:

- password
- API key
- JWT secret
- Redis password
- database password

Gunakan environment variable.

---

## Redis

Gunakan Redis untuk:

- rate limiting
- cache
- session sementara

Jangan menyimpan data permanen di Redis.

---

## Concurrency

Gunakan goroutine hanya jika benar-benar diperlukan.

Pastikan tidak menyebabkan:

- race condition
- deadlock
- goroutine leak

---

## Clean Code

Prioritaskan:

- fungsi kecil
- nama yang jelas
- early return
- minim nested if

Contoh:

Benar:

```go
if err != nil {
    return err
}
```

Hindari:

```go
if err == nil {
    ...
} else {
    ...
}
```

---

## Testing

Apabila membuat business logic baru:

- tambahkan unit test apabila memungkinkan
- jangan merusak test yang sudah ada

---

## Performance

Hindari:

- query N+1
- SELECT *
- loading data yang tidak diperlukan

Gunakan pagination apabila query berpotensi menghasilkan data besar.

---

# 3. Restrictions

AI Coding Agent **WAJIB** mematuhi aturan berikut.

## Git

❌ Dilarang melakukan `git push` apabila masih terdapat error, test gagal, atau kode belum siap dipublikasikan.

---

## Destructive Command

❌ Dilarang menjalankan command yang bersifat destruktif tanpa persetujuan pengguna.

Contoh:

```
rm -rf
DROP DATABASE
DROP TABLE
TRUNCATE
docker system prune -a
docker volume prune
docker network prune
git reset --hard
git clean -fd
```

---

## Folder Structure

❌ Dilarang mengubah struktur folder utama aplikasi tanpa instruksi spesifik dari pengguna.

---

## Dependency

❌ Dilarang menambahkan dependency/library baru apabila masih dapat menggunakan library yang sudah tersedia.

Selalu prioritaskan standard library Go.

---

## Refactoring

❌ Jangan melakukan refactor besar di luar scope task yang sedang dikerjakan.

---

## Database Migration

❌ Jangan mengubah migration yang sudah pernah digunakan.

Apabila terdapat perubahan schema, buat migration baru.

---

## Breaking Changes

❌ Jangan mengubah signature function public tanpa alasan yang jelas.

---

## Backward Compatibility

Usahakan perubahan tetap kompatibel dengan API yang sudah ada kecuali pengguna meminta sebaliknya.

---

## Scope

Kerjakan hanya sesuai permintaan pengguna.

Jangan menambahkan fitur baru tanpa diminta.

---

## Output Quality

Sebelum menyelesaikan task, pastikan:

- code berhasil di-compile
- tidak ada unused import
- tidak ada unused variable
- mengikuti gofmt
- mengikuti struktur project
- mengikuti guideline pada dokumen ini

---
> Source: [aditya3232/shortn-url-opencode](https://github.com/aditya3232/shortn-url-opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
