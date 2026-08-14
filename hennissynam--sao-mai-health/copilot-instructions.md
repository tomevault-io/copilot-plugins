## sao-mai-health

> Đọc file này TRƯỚC KHI làm bất cứ điều gì. Nó chứa mọi ngữ cảnh cần thiết để làm việc hiệu quả trên codebase này.

# CLAUDE.md — Sao Mai Health

Đọc file này TRƯỚC KHI làm bất cứ điều gì. Nó chứa mọi ngữ cảnh cần thiết để làm việc hiệu quả trên codebase này.

---

## Sản phẩm là gì

**Sao Mai Health** là nền tảng giám sát dịch tễ học và sức khỏe cộng đồng.

**Định vị thương mại đã chốt:** B2G platform bán cho CDC/Sở Y tế cấp tỉnh — KHÔNG phải consumer health app.

**Ba trụ cột tạo ra doanh thu:**
1. **Disease Surveillance Intelligence** — bản đồ ca bệnh + hotspot detection + báo cáo mẫu Bộ Y tế → bán cho Sở Y tế
2. **Fast Case Intake (30 giây)** — nhập ca bệnh offline-first → adoption driver cho nhân viên y tế tuyến cơ sở
3. **Environmental Stroke Risk** — AQI + GPS + thời tiết → rủi ro đột quỵ thời gian thực → differentiator với bảo hiểm và bệnh viện tư

**Flywheel logic:** FIELD (case intake miễn phí) → tạo data → RADAR (Sở Y tế trả tiền) → justify CLINIC (phòng khám trả tiền).

---

## Stack kỹ thuật

```
Frontend:  React 18 + TypeScript + Vite + Tailwind + shadcn/ui + Zustand
Backend:   Supabase (PostgreSQL + Auth + Realtime + Edge Functions)
Maps:      Mapbox GL + Leaflet + react-leaflet-cluster
Charts:    Recharts
3D:        Three.js + React Three Fiber
Offline:   Dexie (IndexedDB)
i18n:      i18next (vi + en)
```

**Dev commands:**
```bash
npm install    # cài dependencies
npm run dev    # dev server tại localhost:8080
npm run build  # production build
npm run lint   # ESLint check
```

**Supabase client:** `src/integrations/supabase/client.ts` — dùng `import { supabase } from "@/integrations/supabase/client"`.

---

## Cấu trúc thư mục quan trọng

```
src/
├── pages/              # Route pages — mỗi file = 1 trang
├── components/
│   ├── ui/            # shadcn/ui primitives — KHÔNG sửa trực tiếp
│   ├── biovault/      # Digital Twin components
│   ├── stroke/        # Stroke risk components
│   ├── surveillance/  # Map + hotspot components
│   └── dashboard/     # Dashboard widgets
├── hooks/             # 50+ custom hooks — logic nghiệp vụ ở đây
├── services/
│   ├── healthAPI.ts   # API wrapper + type definitions
│   └── dbService.ts   # DB-first service layer — dùng khi record user events
├── integrations/supabase/
│   ├── client.ts      # Supabase client (anon key)
│   └── types.ts       # Auto-generated DB types — KHÔNG sửa tay
└── i18n/locales/      # vi.json + en.json — thêm keys vào đây khi cần text mới
supabase/functions/    # 30+ Edge Functions (Deno/TypeScript)
```

---

## Patterns bắt buộc dùng

### 1. Supabase calls
```typescript
// Luôn import từ đây
import { supabase } from "@/integrations/supabase/client"

// RPC call
const { data, error } = await supabase.rpc('function_name', { param: value })
if (error) throw error

// Table insert
const { error } = await supabase.from('table_name').insert({ ... })
if (error) throw error

// Lấy user hiện tại
const { data: { user } } = await supabase.auth.getUser()
```

### 2. User event tracking (audit trail)
Dùng `dbService.ts` khi ghi user action — KHÔNG dùng console.log:
```typescript
import { recordEvent } from '@/services/dbService'
await recordEvent({ actionType: 'feature_use', payload: { feature: 'case_intake' } })
```

### 3. Offline-first pattern
```typescript
const { queueRecord, isOnline } = useOfflineStorage()
if (isOnline) {
  await supabase.rpc(...)  // online: lưu thẳng
} else {
  await queueRecord('table', 'insert', data)  // offline: queue
}
```

### 4. Toast notifications
```typescript
import { toast } from "sonner"
toast.success("Thành công")
toast.error("Lỗi: " + error.message)
```

### 5. Form validation
```typescript
import { z } from "zod"
import { useForm } from "react-hook-form"
import { zodResolver } from "@hookform/resolvers/zod"
// Định nghĩa schema → useForm với zodResolver → Form component từ shadcn
```

### 6. i18n
```typescript
import { useTranslation } from 'react-i18next'
const { t } = useTranslation()
// Dùng: t('nav.dashboard')
// Thêm key vào: src/i18n/locales/vi.json VÀ src/i18n/locales/en.json
```

---

## Database — bảng quan trọng nhất

| Bảng | Mục đích |
|------|---------|
| `case_events` | Ca bệnh báo cáo — source of truth cho surveillance |
| `daily_counts` | Thống kê ca bệnh theo ngày/quận/loại |
| `disease_hotspots` | Cụm bệnh được phát hiện bởi AI |
| `alerts` | Cảnh báo ngưỡng ca bệnh |
| `health_facilities` | Cơ sở y tế |
| `biovault_metrics` | Chỉ số sức khỏe cá nhân (lab results, sensors) |
| `user_events` | Audit trail user actions — `{ user_id, action_type, payload }` |
| `health_records` | Hồ sơ sức khỏe |
| `campaigns` | Chiến dịch tiêm chủng/vật tư |

**RPCs quan trọng:**
- `intake_case_fast(p_full_name, p_gender, p_birth_year, p_disease_code, p_facility_id, p_ward_id, p_district_id, p_onset_date, p_report_date, p_status, p_symptoms, p_lat, p_lng, p_mpi_hash, p_address_hash, p_phone_hash)` → ghi ca bệnh nhanh
- `metrics_json(in_date, in_metric)` → KPI metrics
- `get_points_available(p_user)` — điểm loyalty (ít dùng)

**Hashing PII (CCCD, phone, address):**
```typescript
btoa(unescape(encodeURIComponent(value))).slice(0, 32)
```

---

## Supabase Edge Functions — khi nào dùng cái nào

| Function | Dùng khi |
|---------|---------|
| `fetch-hcmc-data` | Lấy dữ liệu ca bệnh HCMC |
| `health-data-synthesis` | Tổng hợp KPI dashboard |
| `health-kpi-intelligence` | Tính toán các chỉ số KPI |
| `disease-hotspot-detector` | Phát hiện cụm bệnh mới |
| `check-stroke-risk` | Tính risk score đột quỵ |
| `fetch-environment-data` | AQI + thời tiết theo tọa độ |
| `classify-alert` | Phân loại và tạo alert |
| `analyze-health-document` | OCR + extract từ file lab |
| `fetch-disease-news` | Tin tức dịch bệnh |
| `geocode-location` | Tọa độ → địa chỉ |
| `health-strategy-engine` | Khuyến nghị chiến lược AI |

**Gọi edge function:**
```typescript
const { data, error } = await supabase.functions.invoke('function-name', {
  body: { param1: value1 }
})
```

---

## KHÔNG làm những điều này

**Features đã quyết định CẮT — đừng khôi phục:**
- Shopify/cart integration (`useCartSync`, `cartStore.ts`) — đã xóa khỏi App.tsx
- Face health scanner dùng cho chẩn đoán — liability pháp lý
- ElevenLabs voice alerts — chi phí cao, dùng push notification thay
- Device sensor tremor/fall detection standalone — nếu cần thì tích hợp vào Stroke Risk

**Coding conventions:**
- Không comment giải thích WHAT code làm — chỉ comment WHY nếu non-obvious
- Không thêm error handling cho cases không thể xảy ra
- Không tạo abstraction khi chưa cần (rule of 3)
- Không mock Supabase trong tests — dùng real DB
- Không hardcode text — dùng i18n keys
- Không sửa `src/components/ui/` (shadcn primitives) — extend bằng wrapper

---

## Lộ trình tính năng (ưu tiên thương mại)

### Đã làm (Sprint 1)
- [x] Fix CaseIntake → gọi RPC `intake_case_fast` thực tế
- [x] Fix LabImport → lưu vào `biovault_metrics`
- [x] Fix useConsentTracker → ghi `user_events`
- [x] Medical disclaimer trên StrokeRisk
- [x] Xóa CartSync/Shopify

### Cần làm tiếp (Sprint 2 — tạo ra deal)
- [ ] Export báo cáo PDF theo mẫu A1/A2 Bộ Y tế — **DEALMAKER số 1 với Sở Y tế**
- [ ] Multi-tenant: org_id isolation (Sở Y tế A không thấy data Sở Y tế B)
- [ ] Role-based access: `admin` | `doctor` | `field_worker` | `viewer`
- [ ] Onboarding flow 3 bước cho admin mới
- [ ] Demo mode với synthetic data HCMC thực tế (không cần login)

### Sprint 3 — Scale
- [ ] FHIR/HL7 resource endpoint (bảng `fhir_resources` đã có)
- [ ] Stroke Risk API endpoint để bán cho bảo hiểm
- [ ] Tích hợp BHYT API (bảng `bhyt_records` đã có schema)
- [ ] Báo cáo trend 30/90/365 ngày per bệnh per địa bàn

---

## Quy tắc bảo mật và pháp lý

1. **PII của bệnh nhân PHẢI hash** trước khi lưu DB: CCCD, phone, address → `btoa(unescape(encodeURIComponent(value))).slice(0, 32)`
2. **Mọi feature AI/ML hiển thị cho người dùng** PHẢI có disclaimer: *"Công cụ hỗ trợ tham khảo. Không thay thế tư vấn và chẩn đoán của bác sĩ."*
3. **Consent trước khi thu thập** sensor data (accelerometer, gyro, camera)
4. **User events audit trail** bắt buộc cho mọi action quan trọng (dùng `user_events` table hoặc `dbService.recordEvent`)
5. **RLS (Row Level Security)** phải được test sau mỗi thay đổi schema

---

## Performance guidelines

- **Memoize expensive computations:** `useMemo` cho risk score calculations, `useCallback` cho event handlers truyền xuống children
- **Virtualise long lists:** dùng `@tanstack/react-virtual` (đã có trong package.json) cho danh sách >100 items
- **Lazy load heavy pages:** `React.lazy()` + `Suspense` cho BioVault, StrokeRisk (heavy 3D/map)
- **Debounce search inputs:** dùng `useDebounce` hook có sẵn tại `src/hooks/useDebounce.ts`
- **Realtime subscriptions:** cleanup trong `useEffect` return — tránh memory leak
- **Map performance:** cluster markers khi >50 points, dùng `react-leaflet-cluster` đã có

---

## Cách test một feature mới

1. Chạy `npm run dev` → mở `localhost:8080`
2. Test golden path: nhập đúng → submit → kiểm tra Supabase dashboard xem data thật chưa
3. Test offline: tắt network trong DevTools → kiểm tra Dexie queue
4. Test error state: truyền data sai → toast error hiện ra không
5. Test i18n: chuyển ngôn ngữ → text có dịch không
6. Kiểm tra console: không có warning React, không có unhandled promise rejection

---

## Context thương mại quan trọng

- **ICP (Ideal Customer Profile):** CDC/Sở Y tế cấp tỉnh — có ngân sách Đề án 06 chuyển đổi số
- **Key pain point:** báo cáo dịch tễ hàng tuần đang làm thủ công bằng Excel → đây là entry point
- **Pricing:** RADAR tier ~500M-2B VND/năm/tỉnh (B2G contract); CLINIC tier ~5-15M VND/tháng/cơ sở
- **Pilot target:** 1 CDC quận/huyện tại HCMC (Thủ Đức hoặc Bình Dương)
- **USP duy nhất không ai có:** Environmental Stroke Risk correlating AQI + thời tiết + GPS realtime

---
> Source: [HENNISSYNAM/sao-mai-health](https://github.com/HENNISSYNAM/sao-mai-health) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
