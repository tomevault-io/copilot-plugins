## observai

> ObservAI, gerçek zamanlı müşteri analizi yapan bir AI platformudur.

# ObservAI — Cursor AI Kuralları
# Bu dosya Cursor'ın tüm AI asistanları (GPT-4, Claude, Gemini) için geçerlidir.

## 🎯 Proje Amacı
ObservAI, gerçek zamanlı müşteri analizi yapan bir AI platformudur.
- Kamera feed'inden YOLO11 ile insan tespiti
- InsightFace/MiVOLO ile demografik analiz (yaş, cinsiyet)
- React dashboard ile canlı görselleştirme

## 📋 Otomatik Geliştirme Sistemi — MUTLAKA OKU

Bu projede **AI-to-AI otomatik geliştirme sistemi** çalışmaktadır:

1. **`HANDOFF.md`** — Önceki AI'ın bıraktığı yer. Her yeni AI oturumu BURADAN başlamalıdır.
2. **`auto_dev_progress.json`** — Tüm görevlerin durumu (pending/in_progress/completed/failed)
3. **`observai_auto_dev.py`** — Scheduled task scripti, otomatik görev yürütür

### Görev Akışı
```
HANDOFF.md oku → sıradaki pending task'ı al → uygula → progress.json güncelle → HANDOFF.md yenile
```

## 🏗️ Teknik Stack

### Frontend (port 5173)
- React 18, TypeScript, Vite
- Tailwind CSS, Recharts, Lucide icons
- `frontend/src/components/camera/CameraFeed.tsx` — Ana kamera bileşeni (1350+ satır)
- `frontend/src/services/cameraBackendService.ts` — Backend sağlık kontrolü

### Node Backend (port 3001)
- Express, TypeScript, Supabase
- `backend/src/routes/python-backend.ts` — Python backend proxy

### Python Backend (port 5001)
- FastAPI + WebSocket
- YOLO11s model (ultralytics)
- InsightFace + MiVOLO (demografik analiz)
- `packages/camera-analytics/camera_analytics/run_with_websocket.py` — Ana WebSocket server
- `packages/camera-analytics/camera_analytics/analytics.py` — AI analiz motoru

## 🔗 Bağlantı State Machine
```typescript
type ConnectionState = 'DISCONNECTED' | 'CONNECTING' | 'WAITING_FOR_BACKEND' | 'CONNECTED' | 'STREAMING' | 'FAILED'
```
- Exponential backoff: `Math.min(30000, Math.pow(2, attempt) * 1000)`
- MJPEG max retry: 8 deneme
- WebSocket health poll: 3 saniyede bir

## 📁 Önemli Dosyalar
```
ObservAI/
├── HANDOFF.md                    ← Her AI oturumu bunu okur
├── auto_dev_progress.json        ← Görev durumları
├── ai_handoff.py                 ← Handoff belgesi üretici
├── observai_auto_dev.py          ← Otomasyon scripti
├── ntfy_bridge.py                ← Windows ntfy.sh köprüsü
├── frontend/src/
│   └── components/camera/CameraFeed.tsx
└── packages/camera-analytics/camera_analytics/
    ├── run_with_websocket.py
    └── analytics.py
```

## ⚠️ Dikkat Edilecekler
- TypeScript strict: `any` kullanma, proper typing yap
- Python: async/await ile yazılmış, sync fonksiyon ekleme
- Her edit sonrası: `cd frontend && npx tsc --noEmit` ile kontrol et
- `auto_dev_progress.json` dosyasını doğrudan düzenle (id, status, completed_at alanları)

## 📊 Güncel İlerleme (2026-03-08 15:33)
- **Tamamlanan:** 15/16 görev (%93)
- **Sıradaki Görev:** **5.0.0** — Sistem Saglık Kontrolü (Test)
- **Son Not:** Scheduled task calisti (2026-03-08). observai_auto_dev.py --once komutu VM sandbox'ta internet erisimi olmadigi icin erken cikti (beklenen davranis). Tüm 15 gelistirme gorevi (Faz 1-4) tamamlanmis dur

## ⚡ Hemen Şimdi Yap
1. `HANDOFF.md` dosyasını oku (tüm bağlam burada)
2. `auto_dev_progress.json` içindeki ilk `pending` görevi seç
3. Görevi uygula
4. `auto_dev_progress.json` içinde status'ü `"completed"` yap ve `completed_at` ekle
5. `python ai_handoff.py` çalıştır (handoff belgesini güncelle)

---
> Source: [observaianeye/ObservAI](https://github.com/observaianeye/ObservAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
