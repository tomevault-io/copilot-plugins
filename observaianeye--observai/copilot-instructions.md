## observai-context

> ObservAI projesi için AI bağlamı - her dosya açıldığında otomatik yükle


# 🤖 ObservAI — AI Asistan Bağlamı

Sen bu projede **otomatik geliştirme sisteminin** bir parçasısın.
`HANDOFF.md` dosyasını oku ve kaldığın yerden devam et.

## Proje Yapısı
- **Frontend**: `frontend/` — React 18 + TypeScript + Vite (port 5173)
- **Node Backend**: `backend/` — Express + TypeScript + Supabase (port 3001)
- **Python Backend**: `packages/camera-analytics/` — FastAPI + WebSocket + YOLO11 + InsightFace (port 5001)

## Otomatik Geliştirme Sistemi
- **Görev takibi**: `auto_dev_progress.json` — tüm tasklar burada
- **AI Handoff**: `HANDOFF.md` — önceki AI'ın bıraktığı yer, sıradaki görev
- **Otomasyon scripti**: `observai_auto_dev.py` — scheduled task olarak çalışır
- **Bildirim köprüsü**: `ntfy_bridge.py` — Windows'ta çalışır, telefona ntfy.sh bildirimi gönderir

## ⚡ İlk Yapman Gereken
1. `HANDOFF.md` dosyasını oku
2. Sıradaki görev: **5.0.0** — Sistem Saglık Kontrolü (Test)
3. Görevi uygula → `auto_dev_progress.json` güncelle
4. `python ai_handoff.py` çalıştır

## 📊 İlerleme (2026-03-08 15:33)
- 15/16 görev tamamlandı (%93)

## 📊 İlerleme (2026-03-04 03:05)
- 15/15 görev tamamlandı (%100)

## 📊 İlerleme (2026-03-04 00:35)
- 14/15 görev tamamlandı (%93)

## 📊 İlerleme (2026-03-03 23:05)
- 13/15 görev tamamlandı (%86)

## 📊 İlerleme (2026-03-03 22:36)
- 12/15 görev tamamlandı (%80)

## 📊 İlerleme (2026-03-03 22:04)
- 11/15 görev tamamlandı (%73)

## 📊 İlerleme (2026-03-03 21:39)
- 10/15 görev tamamlandı (%66)

## 📊 İlerleme (2026-03-03 19:13)
- 9/15 görev tamamlandı (%60)

## 📊 İlerleme (2026-03-03 18:05)
- 8/15 görev tamamlandı (%53)

## 📊 İlerleme (2026-03-03 17:37)
- 7/15 görev tamamlandı (%46)

## 📊 İlerleme (2026-03-03 16:44)
- 6/15 görev tamamlandı (%40)

## 📊 İlerleme (2026-03-03 16:32)
- 5/15 görev tamamlandı (%33)

## 📊 İlerleme (2026-03-03 15:59)
- 4/15 görev tamamlandı (%26)

## Kodlama Kuralları
- TypeScript strict mode aktif, `any` kullanma
- React hooks kurallarına uy
- Python: async/await kullan, type hints ekle
- Her değişiklikten sonra TypeScript build kontrol et: `cd frontend && npx tsc --noEmit`
- Commit mesajı: `feat(task-X.Y.Z): kısa açıklama`

---
> Source: [observaianeye/ObservAI](https://github.com/observaianeye/ObservAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
