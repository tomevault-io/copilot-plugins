## fiziktedaviuyg

> Web tabanlı, uygulama indirmeye gerek duymayan, **tamamen istemci tarafında (Edge AI)** çalışan fizik tedavi egzersiz takip platformu. Video verisi **asla sunucuya gönderilmez**; cihazda işlenerek yalnızca biyomekanik JSON metrikleri iletilir.

# CLAUDE.md — Edge AI Fizik Tedavi Asistanı

## Proje Özeti

Web tabanlı, uygulama indirmeye gerek duymayan, **tamamen istemci tarafında (Edge AI)** çalışan fizik tedavi egzersiz takip platformu. Video verisi **asla sunucuya gönderilmez**; cihazda işlenerek yalnızca biyomekanik JSON metrikleri iletilir.

---

## Teknoloji Stack'i

### Frontend / Edge AI
- **Pose Estimation:** `@mediapipe/tasks-vision` → BlazePose (WorldLandmarks, 3D)
- **GPU Hızlandırma:** WebGPU (birincil) → WebGL 2.0 (fallback)
- **Threading:** Web Workers (inference main thread'den ayrı çalışır)
- **Wasm:** SIMD + Pthread optimizasyonları aktif
- **Biyomekanik:** Kosinüs teoremi ile ROM hesabı, DTW ile jitter giderme
- **Framework:** React + TypeScript + Vite

### Backend
- **Runtime:** Node.js (TypeScript) veya Python (FastAPI) — karar aşamasında
- **Realtime:** WebSocket (eş zamanlı izleme senaryoları için)
- **REST:** HTTP/2 + TLS 1.3
- **Standart:** HL7 FHIR R4 (Observation, Procedure resource modelleri)

### Veritabanı
- **İlişkisel:** PostgreSQL (hasta, klinisyen, seans kayıtları)
- **Zaman serisi:** TimescaleDB veya InfluxDB (ROM trend verileri)
- **Şifreleme:** AES-256 (data at rest)

### Güvenlik & Uyumluluk
- HIPAA + KVKK (Privacy by Design)
- TLS 1.3 (data in transit)
- Video frame'ler RAM'de işlendikten milisaniyeler içinde imha edilir
- Sunucuya **asla** piksel/video/fotoğraf gönderilmez

---

## Klasör Yapısı

```
/
├── CLAUDE.md                        ← Bu dosya
├── packages/
│   ├── client/                      ← React + TypeScript frontend
│   │   ├── public/
│   │   ├── src/
│   │   │   ├── workers/
│   │   │   │   └── pose.worker.ts   ← MediaPipe inference (Web Worker)
│   │   │   ├── engine/
│   │   │   │   ├── poseEngine.ts    ← WebGPU/WebGL delegate yönetimi
│   │   │   │   ├── biomechanics.ts  ← ROM hesabı, kosinüs teoremi
│   │   │   │   ├── occlusion.ts     ← Visibility skoru & fallback mantığı
│   │   │   │   └── repCounter.ts    ← DTW tabanlı tekrar sayımı
│   │   │   ├── components/
│   │   │   │   ├── Camera/          ← Video akışı & canvas overlay
│   │   │   │   ├── ExerciseView/    ← Egzersiz UI
│   │   │   │   └── Dashboard/       ← Hasta özet ekranı
│   │   │   ├── hooks/
│   │   │   │   ├── usePoseDetection.ts
│   │   │   │   └── useWebSocket.ts
│   │   │   ├── transport/
│   │   │   │   └── wsClient.ts      ← JSON metrik gönderimi (WS/WebRTC)
│   │   │   └── types/
│   │   │       ├── biomechanics.ts
│   │   │       └── fhir.ts
│   │   └── vite.config.ts
│   │
│   ├── server/                      ← Backend API
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   │   ├── sessions.ts      ← Seans CRUD
│   │   │   │   ├── metrics.ts       ← Biyomekanik veri alımı
│   │   │   │   ├── fhir.ts          ← FHIR Observation/Procedure endpoint'leri
│   │   │   │   └── clinician.ts     ← Klinisyen dashboard API
│   │   │   ├── ws/
│   │   │   │   └── metricsSocket.ts ← WebSocket sunucu tarafı
│   │   │   ├── db/
│   │   │   │   ├── schema.sql        ← PostgreSQL şeması
│   │   │   │   └── migrations/
│   │   │   ├── fhir/
│   │   │   │   ├── observation.ts   ← ROM → FHIR Observation builder
│   │   │   │   └── procedure.ts     ← Egzersiz → FHIR Procedure builder
│   │   │   └── security/
│   │   │       ├── encryption.ts    ← AES-256 yardımcıları
│   │   │       └── tls.ts
│   │   └── tsconfig.json
│   │
│   └── shared/                      ← Ortak tipler (client + server)
│       └── src/
│           ├── MetricPayload.ts     ← WebSocket/REST JSON şeması
│           └── FHIRTypes.ts
│
├── docs/
│   ├── architecture/
│   │   ├── system-diagram.md        ← Deliverable 1: Sistem mimarisi
│   │   └── sequence-diagrams.md     ← Deliverable 2: Sekans diyagramları
│   ├── database/
│   │   └── schema-design.md         ← Deliverable 3: DB şeması & FHIR modeli
│   └── edge-ai/
│       └── webgpu-workers-guide.md  ← Deliverable 4: WebGPU/Workers/Wasm kılavuzu
│
└── docker-compose.yml
```

---

## Kritik Mimari Kararlar (Değiştirilmez)

### 1. Video Asla Sunucuya Gönderilmez
```
Kamera → RAM (frame) → MediaPipe (inference) → JSON metrikler → Backend
                ↑
        Frame burada imha edilir
```
Sunucuya giden tek veri: `{ patientId, timestamp, metrics: { knee_flexion_deg, ... } }`

### 2. Web Worker Zorunluluğu
MediaPipe inference **mutlaka** `pose.worker.ts` içinde çalışmalıdır.
Main thread'de inference yapılması UI freeze'e neden olur → **kesinlikle yasaktır**.

### 3. WebGPU → WebGL Fallback Sırası
```typescript
// poseEngine.ts içinde bu sıra korunmalıdır
const delegate = await detectDelegate(); // 'GPU' | 'WEBGL' | 'CPU'
```
WebGPU yoksa WebGL 2.0, o da yoksa CPU fallback devreye girer.

### 4. Oklüzyon Güvenlik Sınırı
`visibility < 0.5` olan eklem noktaları için açı hesabı **durdurulur**, önceki geçerli değer korunur.
Halüsinasyon (hatalı açı) üretimi engellenmiş olur.

### 5. FHIR Uyumu
ROM verileri `FHIR Observation` kaynağına, egzersiz seansları `FHIR Procedure` kaynağına map'lenmelidir.
USBS / e-Nabız entegrasyonu bu standart üzerinden sağlanır.

---

## Veri Akışı (Özet)

```
[Kamera]
   │  video frame (RAM only)
   ▼
[Web Worker: pose.worker.ts]
   │  MediaPipe BlazePose (WorldLandmarks, 3D)
   │  WebGPU / WebGL delegate
   ▼
[biomechanics.ts]
   │  ROM açıları (kosinüs teoremi)
   │  Visibility kontrolü (oklüzyon fallback)
   │  Tekrar sayımı (DTW smoothing)
   ▼
[wsClient.ts]
   │  JSON payload — TLS 1.3
   │  { patientId, timestamp, jointAngles: {...} }
   ▼
[Backend: metricsSocket.ts]
   │  AES-256 şifreli saklama
   ▼
[PostgreSQL / TimescaleDB]
   │
   ▼
[Clinician Dashboard]  ←→  [FHIR API]  ←→  [e-Nabız / USBS / EMR]
```

---

## Beklenen Çıktılar (Deliverables) — Durum Takibi

| # | Çıktı | Dosya | Durum |
|---|-------|-------|-------|
| 1 | Sistem Mimarisi Şeması | `docs/architecture/system-diagram.md` | ⬜ Bekliyor |
| 2 | Sekans Diyagramları | `docs/architecture/sequence-diagrams.md` | ⬜ Bekliyor |
| 3 | DB Şeması & FHIR Modeli | `docs/database/schema-design.md` | ⬜ Bekliyor |
| 4 | WebGPU/Workers/Wasm Kılavuzu | `docs/edge-ai/webgpu-workers-guide.md` | ⬜ Bekliyor |
| 5 | Çalışan Frontend (Edge AI) | `packages/client/` | ⬜ Bekliyor |
| 6 | Çalışan Backend API | `packages/server/` | ⬜ Bekliyor |

---

## Kodlama Kuralları

- **Dil:** TypeScript (strict mode) her yerde
- **Linter:** ESLint + Prettier
- **Test:** Vitest (unit), Playwright (e2e)
- **Commit:** Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`)
- **Güvenlik:** Hiçbir zaman `console.log` ile hasta verisi yazdırılmaz
- **Yorum dili:** Türkçe (iş mantığı), İngilizce (teknik/API)

---

## Sık Karşılaşılan Sorunlar & Çözümler

| Sorun | Çözüm |
|-------|-------|
| MediaPipe WASM yüklenmiyor | `vite.config.ts`'te `optimizeDeps.exclude: ['@mediapipe/tasks-vision']` ekle |
| Web Worker'da SharedArrayBuffer hatası | Sunucu response header'larına `COOP` + `COEP` ekle |
| WebGPU `undefined` geliyor | Chrome 113+ gerekli; fallback algılaması `poseEngine.ts`'te yapılır |
| Visibility skoru 0.5 sınırında titriyor | Hysteresis band uygula: aç için 0.55, kapat için 0.45 |
| FHIR endpoint'i e-Nabız'a bağlanmıyor | USBS sandbox ortamında FHIR R4 base URL'ini `server/.env`'e ekle |

---

## Ortam Değişkenleri (`.env` şablonu)

```bash
# Server
DATABASE_URL=postgresql://user:pass@localhost:5432/physio_db
JWT_SECRET=<üretilecek>
AES_256_KEY=<üretilecek>
FHIR_BASE_URL=https://fhir.e-nabiz.gov.tr/r4   # sandbox

# Client (Vite)
VITE_WS_URL=wss://api.yourdomain.com/ws
VITE_API_URL=https://api.yourdomain.com
```

---

## Referanslar

- [MediaPipe Tasks Vision](https://developers.google.com/mediapipe/solutions/vision/pose_landmarker)
- [WebGPU Spec](https://gpuweb.github.io/gpuweb/)
- [HL7 FHIR R4 Observation](https://www.hl7.org/fhir/observation.html)
- [HL7 FHIR R4 Procedure](https://www.hl7.org/fhir/procedure.html)
- [KVKK Rehberi](https://www.kvkk.gov.tr)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fevzisuhansahin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
