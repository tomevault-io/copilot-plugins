## greenpool-erp

> > Auto-updated: 2026-06-11 | Người dùng: Nguyễn Văn Hướng (QUẢN TRỊ VIÊN HỆ THỐNG)

# CLAUDE.md — Green Pool ERP Working Guide
> Auto-updated: 2026-06-11 | Người dùng: Nguyễn Văn Hướng (QUẢN TRỊ VIÊN HỆ THỐNG)
> Repo: https://github.com/trihuong815-oss/greenpool-erp
> Production: https://greenpool-erp.vercel.app (Firebase App Hosting — asia-southeast1)

---

## 1. THÔNG TIN DỰ ÁN

### Tổng quan
- **Tên**: Green Pool ERP — Hệ thống quản lý vận hành 5 cơ sở bể bơi
- **Stack**: Next.js 16 + React 18 + TypeScript + Tailwind CSS + Firebase (Auth + Firestore + Storage + FCM)
- **Deploy**: Firebase App Hosting (apphosting.yaml) — NOT Vercel deploy, chỉ domain vercel.app
- **Font**: Inter (Google Fonts) + system font fallback — import trong globals.css

### 5 Cơ sở (BranchId)
| ID | Tên đầy đủ | Màu |
|----|-----------|-----|
| HM | Green Pool Hoàng Mai | #10b981 |
| TK | Green Pool 20 Thuỷ Khuê | #06b6d4 |
| CTT | Green Pool Cung Thể Thao Mã | #8b5cf6 |
| 24 | Green Pool 24 Nguyễn Công Trứ | #f59e0b |
| TT | Green Pool Thanh Trì | #ef4444 |

### Cấu trúc tổ chức (Roles)
Các role function check: `isTopAdmin`, `isCEO`, `isAdminSystem`, `isGD`, `isTP`, `isQLCS`, `isWriteAdmin`, `canSeeAllFacilities`, `hasRole`
(source: lib/auth/roles.ts — SINGLE SOURCE OF TRUTH)

---

## 2. KIẾN TRÚC CODE

### Cấu trúc thư mục chính
```
app/
  (app)/              # protected routes — yêu cầu login
    dashboard/        # DashboardContent.tsx, KTDashboardSection.tsx
    giao-viec/        # GiaoViecClient.tsx, TaskCreateModal.tsx, TaskDetailModal.tsx
    doanh-so/         # NhapClient.tsx, PackagesClient.tsx, ManageSalesModal.tsx
    ky-thuat/         # GiaoViecClient.tsx (KT), HoaChatClient.tsx, MayClient.tsx
    checklist-v2/     # ChecklistV2Client.tsx, SupervisorView.tsx, ChecklistHeatmap.tsx
    cong-viec-ca-nhan/ # PersonalWorkClient.tsx + AIPanel, GoalsPanel, HabitsPanel, JournalPanel
    tin-nhan/         # TinNhanClient.tsx + ChatAttachments, MessageThread, Modals
    sodo/             # OrgChartClient.tsx, FlowView.tsx, OrgTreeView.tsx
    bao-cao/          # page.tsx (báo cáo tự động)
    bao-mat/          # SecurityClient.tsx (bảo mật & thông báo)
    users/            # UsersClient.tsx, PermissionGrantPanel.tsx
    quan-ly-sale/     # QuanLySaleClient.tsx
    settings-packages/ # page.tsx
  api/
    tasks/            # 8 route handlers (CRUD + comments + attachments + approval)
    sales/            # 5 route handlers
    ky-thuat/         # 5 route handlers
    chat/             # 8 route handlers
    personal/         # 19 route handlers
    cron/             # 5 scheduled jobs
    admin/            # 5 admin routes
  globals.css         # @import Inter, @tailwind base/components/utilities
  layout.tsx          # root layout — body: antialiased, bg-slate-50 text-slate-800
components/
  AppShell.tsx        # layout shell: sidebar + topbar + main
  Sidebar.tsx         # navigation sidebar
  AppTopBar.tsx       # top navigation bar
  ui/                 # Button, Card, Input, Badge, Toast, EmptyState, Skeleton...
lib/
  types/
    index.ts          # BARREL — import tất cả types từ đây
    tasks.ts          # re-export từ services/tasks/api-client.ts
    users.ts          # UserDoc, UserPublic, FcmDevice, CallerProfile
    branches.ts       # BranchId, BranchMeta, BRANCHES, BRANCH_BY_ID
  auth/
    roles.ts          # SINGLE SOURCE — isTopAdmin, isCEO, isGD, isTP, isQLCS...
    can.ts            # permission helpers
  firebase/
    collections.ts    # SINGLE SOURCE — tên collection Firestore
    admin.ts          # Firebase Admin SDK
    client.ts         # Firebase Client SDK
  services/
    tasks/api-client.ts      # Task CRUD, TaskCreate, Task interface
    sales/api-client.ts      # Sales CRUD
    ky-thuat/work-api-client.ts  # KT work tasks
    chat/api-client.ts       # Chat/messaging
  permissions.ts      # permission logic
  branches.ts         # branch data
  navigation/routes.ts # ALL_ROUTES definition
```

---

## 3. MENU / NAVIGATION

### Sections và routes (theo thứ tự sidebar)
**Tổng quan**: dashboard | tin-nhan | cong-viec-ca-nhan
**Vận hành**: doanh-so | doanh-so/nhap | ky-thuat | checklist-v2 | quy-trinh | giao-viec
**Nhân sự**: sodo | luong
**Báo cáo**: bao-cao | daotao | mkt
**Cài đặt**: bao-mat | doanh-so/packages | users | doi-mat-khau

---

## 4. MODULE ĐIỀU PHỐI CÔNG VIỆC (/giao-viec) — TRUNG TÂM HỆ THỐNG

### Files
- `app/(app)/giao-viec/GiaoViecClient.tsx` — main client component
- `app/(app)/giao-viec/TaskCreateModal.tsx` — form tạo task/proposal
- `app/(app)/giao-viec/TaskDetailModal.tsx` — chi tiết + timeline + actions
- `app/(app)/giao-viec/page.tsx` — server component wrapper

### Tabs
- **Tôi phụ trách** (my-tasks): tasks giao cho mình
- **Tôi giao** (assigned-by-me): tasks mình tạo
- **Liên khối** (cross-block): chỉ CEO/Admin
- **Chờ phản hồi** (pending-response): cần action
- **Quá hạn** (overdue): deadline đã qua

### KPI Header (5 cards)
Đang xử lý | Chờ phản hồi | Chờ duyệt | Quá hạn | Hoàn thành

### 3 Panels (header section)
Công việc theo khối | Tắc nghẽn hiện tại | Công việc quá hạn

### Table View (9 cột)
\# | Công việc | Loại | Khối chủ trì | Phối hợp | Trạng thái | Tiến độ | Đang chờ | Deadline

### Task Interface (Task type — lib/services/tasks/api-client.ts)
```typescript
interface Task {
  id: string;
  kind: 'assignment' | 'proposal';
  title: string;
  description: string;
  createdBy, createdByName, createdByRole, createdByBlock, createdAt
  assigneeBlock: Block;
  assigneeDeptId: string | null;
  assigneeFacilityId: string | null;
  assigneeUserIds: string[];        // multi-performer
  collaboratorDeptIds?: string[];   // đơn vị phối hợp — phòng ban
  collaboratorFacilityIds?: string[]; // đơn vị phối hợp — cơ sở
  goal?: string | null;             // mục tiêu công việc
  crossBlock: boolean;
  status: TaskStatus;
  priority: TaskPriority;
  dueDate: string | null;
  progressPct: number;
  approvalChain?: string[];         // chuỗi role cần duyệt
  currentApprover?: string | null;
  proposalType?: ProposalType | null;
  financialGroup?: FinancialGroup | null;
  estimatedCost?: number | null;
}
```

### TaskStatus values
`pending_approval` | `pending` | `in_progress` | `requested_revision` | `done` | `rejected` | `cancelled`

---

## 5. FIRESTORE COLLECTIONS (lib/firebase/collections.ts)

```
BRANCHES          = 'branches'
USERS             = 'users'
TASKS             = 'tasks'           # tasks + subcoll: comments, attachments
CHECKLISTS        = 'checklists'      # subcoll: items, evidenceFiles
TEMPLATES         = 'templates'       # subcoll: items
AUDIT_LOGS        = 'auditLogs'
SALES             = 'sales'
LEADS             = 'leads'
LEAD_ACTIVITIES   = 'leadActivities'
SALES_ENTRIES     = 'salesEntries'
PACKAGE_GROUPS    = 'packageGroups'
PACKAGES          = 'packages'
PACKAGE_SALES     = 'packageSales'
PACKAGE_QUANTITIES = 'packageQuantities'
DISCREPANCIES     = 'discrepancies'
CHEMICAL_ENTRIES  = 'chemicalEntries' # KT: clo/axit xử lý nước
MACHINES          = 'machines'        # KT: máy lọc/nhiệt
MACHINE_RUNS      = 'machineRuns'     # KT: giờ chạy máy
TECH_WORK         = 'techWork'        # KT: tasks + reports + proposals
SALES_TARGETS     = 'salesTargets'
DASHBOARD_SNAPSHOTS = 'dashboardSnapshots'
DEPARTMENTS       = 'departments'
ROLES             = 'roles'
PERSONAL_TASKS    = 'personalTasks'   # owner-only
PERSONAL_JOURNAL  = 'personalJournal' # owner-only
PERSONAL_HABITS   = 'personalHabits'  # owner-only
PERSONAL_GOALS    = 'personalGoals'   # owner-only
CHECKLIST_RUNS_V2 = 'checklistRunsV2'
```

---

## 6. DESIGN SYSTEM

### Font
- **Body font**: Inter (Google Fonts) — được import trong `app/globals.css`
- **Fallback chain**: Inter, -apple-system, "system-ui", "Segoe UI", Helvetica Neue, Arial
- **font-family: inherit** trên input/select/textarea/button (defined in globals.css)
- **font-mono**: ui-monospace, SFMono-Regular, "SF Mono", Menlo, "Cascadia Code", "JetBrains Mono", "Noto Sans Mono" (KHÔNG dùng Consolas/Courier New — thiếu glyph tiếng Việt trên Windows/Android)

### Tailwind Font Scale — CHUẨN BẮT BUỘC
| Class | Size | Dùng cho |
|-------|------|---------|
| text-xs | 12px | labels, badges, chips, meta info, table secondary |
| text-sm | 14px | table body, form labels, secondary text |
| text-base | 16px | body text, form inputs |
| text-lg | 18px | subheadings |
| text-xl | 20px | card titles |
| text-2xl | 24px | KPI values |
| text-3xl | 30px | page headers |

⚠️ **TUYỆT ĐỐI KHÔNG DÙNG** arbitrary pixel sizes: `text-[9px]`, `text-[10px]`, `text-[11px]`
→ Thay bằng `text-xs` (12px minimum readable)

### Icon sizes — CHUẨN
- Minimum: size={12}
- Inline text icon: size={12}–{14}
- Button icon: size={14}–{16}
- Section icon: size={16}–{18}
- ⚠️ **KHÔNG DÙNG** size={9}, size={10}, size={11}

### Custom CSS classes (globals.css)
```css
.card        { bg-white rounded-xl p-5 shadow-sm border border-slate-200 }
.card-title  { text-sm font-bold text-slate-800 mb-3 pb-2 border-b-2 border-slate-100 flex items-center gap-2 }
```

### Color palette (Tailwind)
- Brand: emerald-600 (primary), teal-600 (secondary)
- Status: sky=đang làm, amber=chờ phản hồi, orange=chờ duyệt, rose=quá hạn, emerald=hoàn thành, slate=default
- Custom: lavi (50/100/500/600/900/950) — xanh navy lavi

### Spacing pattern
- Card padding: p-4 hoặc p-5
- Section gap: space-y-5
- Grid gap: gap-3 hoặc gap-4
- Button padding: px-4 py-2 (large), px-3 py-1.5 (medium), px-2 py-1 (small)

---

## 7. DASHBOARD (/dashboard)

### Thứ tự sections (BẮT BUỘC)
1. Header KPI (5 cards) + link "Điều phối công việc →"
2. Cơ sở (FacilitySection)
3. Doanh số (RevenueSection)
4. Kỹ thuật (KTDashboardSection) — **GIỮ metric Axit (pH)**
5. Công việc chi tiết

### KPI Interface (TaskCounts)
```typescript
{
  myInProgress: number;
  approvalNeeded: number;   // chờ phản hồi
  pendingApproval?: number; // chờ duyệt
  overdue?: number;
  myDone: number;
  todo?: number;
  checklistSent?: number;
  checklistUnread?: number;
}
```

### RevenueSummary Interface
```typescript
{ monthPct?: number; yearPct?: number; ... }
```

### KTDashboardSection
- KHÔNG XÓA metric Axit (pH) — đây là yêu cầu cứng
- Hiển thị: system.cloTotal, system.locCapTotal, system.nhietCapTotal

---

## 8. QUI TẮC LÀM VIỆC VỚI CODE

### Workflow commit
1. Fetch SHA hiện tại của file trước khi PUT
2. Đọc nội dung file (btoa decode)
3. Patch nội dung
4. Encode lại: `btoa(unescape(encodeURIComponent(content)))`
5. PUT với SHA chính xác
6. Verify build: GET /commits/main/check-runs

### Build checks
- TypeScript Check ✅
- Next.js Build ✅
- Vitest Unit Tests ✅
- Firestore Rules Compile ✅
- App Hosting Rollout ✅ (mất 3–8 phút)

### Token restore sau navigate/timeout
```javascript
window._ghToken = 'ghp_XXXX...XXXX (lấy từ GitHub Settings → PATs)';
```
Token: greenpool-claude-sprint2, expires 2026-06-18, scope: repo

---

## 9. ⚠️ ENCODING — NGUYÊN NHÂN GỐC RỄ LỖI "CHỮ KHÔNG ĐỌC ĐƯỢC"

### Root cause: MOJIBAKE (double-encoded UTF-8)
Khi push từ máy Windows/editor sai encoding:
- UTF-8 bytes bị đọc như CP1252, rồi save lại → double-encoded
- Kết quả trong file: `'ChÃ¡Â»Â duyÃ¡Â»Ât'` thay vì `'Chờ duyệt'`
- Browser render thành ký tự lạ / ô vuông

### Cách nhận biết
- Source code có các ký tự: `Ã`, `Â`, `á»`, `á»`, `áº`
- Xem file trên GitHub Raw — nếu thấy các ký tự trên = mojibake

### Fix khi phát hiện
1. Dùng `ftfy` (Python: `ftfy.fix_text()`) để auto-decode
2. Hoặc: save file với encoding UTF-8 (không phải UTF-8-BOM)
3. Push lại

### Phòng tái lỗi
- VSCode setting: `"files.encoding": "utf8"`
- Không dùng Notepad Windows để edit file .tsx/.ts
- Khi thấy lỗi "chữ không đọc được" → CHECK MOJIBAKE TRƯỚC khi fix font

### Quy trình khi Claude gặp source có mojibake
1. DỪNG ngay — không sửa font/layout
2. Kiểm tra: `content.indexOf('Ã') > -1 || content.indexOf('Â') > -1`
3. Báo user: "File bị mojibake — cần fix encoding trước"
4. Đề xuất: decode lại bằng ftfy hoặc sửa tay

---

## 10. RULES CỨNG (KHÔNG VI PHẠM)

- ✅ PHẢI verify SHA hiện tại trước mỗi PUT commit
- ✅ PHẢI restore `window._ghToken` sau mỗi navigate
- ✅ KHÔNG xóa metric Axit (pH) khỏi KTDashboardSection
- ✅ KHÔNG thêm menu sidebar mới không cần thiết
- ✅ KHÔNG phá vỡ TypeScript types
- ✅ "Đề xuất" đã ẩn khỏi Điều phối sidebar — KHÔNG khôi phục
- ✅ KHÔNG embed actual GitHub token vào file content (bị Secret Scanning block)
- ✅ CEO không cần thấy số liệu thô kỹ thuật
- ✅ Dashboard thứ tự: KPI → Cơ sở → Doanh số → Kỹ thuật → Công việc
- ✅ Font size tối thiểu: text-xs (12px) — KHÔNG dùng text-[9px]/[10px]/[11px]
- ✅ Icon size tối thiểu: 12 — KHÔNG dùng size={9}/{10}/{11}
- ✅ KHÔNG dùng font-mono cho text tiếng Việt

---

## 11. DEPENDENCIES CHÍNH

```json
{
  "next": "^16.2.6",
  "react": "^18.3.1",
  "firebase": "^12.13.0",
  "firebase-admin": "^13.10.0",
  "lucide-react": "^0.408.0",
  "tailwind-merge": "^2.5.2",
  "recharts": "^2.12.7",
  "clsx": "^2.1.1",
  "@sentry/nextjs": "^10.56.0"
}
```

---

## 12. PHASE HISTORY (tham khảo)

| Phase | Nội dung |
|-------|---------|
| 1–2 | MVP auth + checklist cơ bản |
| 3 | Dashboard snapshots |
| 4 | Users → profiles migration |
| 5 | Push notifications FCM |
| 6 | Sales module (leads, packages, targets, entries, discrepancies) |
| 7 | KT module (chemical, machines, tech-work) + Tasks/GiaoViec |
| 8 | System errors + admin tools |
| 9 | Personal workspace (tasks, journal, habits, goals, AI coach) |
| 10 | Checklist v2 (spec 2026-05-28) |
| 11 | Tin nhắn / Chat |
| 12 | Sales pipeline v2 |
| 13 | Session management, PWA, mobile UX |
| B.x | Type consolidation (lib/types barrel), role helpers, branch SSOT |

---

## 13. API ROUTES TASKS (/api/tasks)

```
GET    /api/tasks                # list tasks (mode + filters)
POST   /api/tasks                # create task
GET    /api/tasks/[id]           # get single task
PATCH  /api/tasks/[id]           # update task
GET    /api/tasks/[id]/comments  # list comments/timeline
POST   /api/tasks/[id]/comments  # add comment
POST   /api/tasks/[id]/attachments # upload attachment
POST   /api/tasks/[id]/approve   # approve/reject
```

---

## 14. COMMIT PATTERNS CHUẨN

```
feat: <mô tả tính năng mới>
fix: <mô tả bug fix>
style: <CSS/layout changes>
refactor: <code restructure>
chore: <config, deps, tooling>
```

---

## 15. NOTIFICATION SYSTEM V6.5 (Phase A+B+C đã hoàn tất 2026-06-15)

### Kiến trúc (BẮT BUỘC theo)
1. **Notification DB = nguồn dữ liệu gốc** (Firestore collection `notifications`).
   Mỗi noti có schema: userId/module/entityId/title/message/type/priority/
   isRead/isActionRequired/actionStatus/pushStatus/pushError/sentAt/retryCount/
   nextRetryAt/pushPayloadSnapshot.
2. **Push FCM = kênh nhắc** (không phải source). Push fail → noti vẫn còn trong DB.
3. **Badge sidebar/chuông đọc từ DB** qua `use-noti-counts` hook + Firestore realtime listener.
4. **Engine duy nhất**: `sendNotificationEvent` (`lib/firebase/noti-engine.ts`). MỌI noti
   từ task/proposal/dispatch/chat/kt PHẢI đi qua engine này — KHÔNG gọi `pushToUsers` thuần.

### 4 module noti
- `proposal`  → đề xuất duyệt
- `dispatch`  → điều phối công việc
- `chat`      → tin nhắn (Phase A.1 noti audit 2026-06-15)
- `kt`        → kỹ thuật vận hành (Phase C.1 noti audit 2026-06-15)

### iOS PWA notification — limitation
- iOS Safari KHÔNG deliver web push notification nếu chưa "Add to Home Screen" (PWA installed).
- Component `IOSInstallPwaBanner` (components/) tự detect + show banner hướng dẫn.
- iOS PWA deeplink có delay 5-10s (Apple throttle FCM web push).

### FCM token lifecycle
- Storage: `users.fcmDevices[]` với schema { token, enabled, lastSeen, userAgent, label, disabledReason?, disabledAt? }
- Heartbeat: cron + client mỗi 6h update `lastSeen`. Nếu device.enabled=false → server trả 410 (Phase A.3 audit).
- Cleanup stale token: cron daily 10:00 VN xoá fcmDevices có lastSeen >7 ngày.
- Invalid token (FCM trả `registration-token-not-registered`): SOFT DELETE (enabled=false + disabledReason='invalid') — Phase B.5 audit.
- Retry queue: 5p / 15p / 30p, max 3 lần (cron retry-failed-push every 5 min).

### Cron jobs noti
- `*/5 * * * *` retry-failed-push (FCM push fail)
- `0 3 * * *`   cleanup-stale-fcm (xoá token >7d)
- `15 * * * *`  proposal-overdue (SLA reminder)
- `30 * * * *`  dispatch-overdue (escalation GĐ khối >24h)
- `45 * * * *`  action-required-stuck (resend >24h chưa xử lý)
- `50 * * * *`  proposal-stale-recipient (cancel proposal+dispatch khi approver disabled)

### Channel settings per user
- Settings page: `/bao-mat` → component `NotiChannelsSettings`
- API: `GET/PUT /api/personal/noti-channels`
- 3 module × 3 channel: inApp/push/email — inApp luôn ON (source of truth)
- Engine read user override khi gửi: nếu user tắt push/email → skip kênh đó (vẫn persist inApp doc)

### Email backup (Gmail SMTP)
- Wrapper: `lib/email/gmail-smtp-client.ts` (Nodemailer)
- Env vars: `GMAIL_SMTP_USER` + `GMAIL_SMTP_PASS` (App Password)
- Default policy: chỉ gửi email cho `ACTION_REQUIRED_TYPES` (proposal cần duyệt, KT proposal pending, ...)
- KHÔNG gửi cho chat_message (spam quá)

### Multi-tab sync
- BroadcastChannel `gp-noti-sync`: tab A mark read → tab B refetch counters (Phase C.4 audit)
- Realtime onSnapshot `notifications where userId+isActionRequired+actionStatus=pending` → debounce 150ms → fetchBiz

### Deep link
- `/de-xuat?proposalId=X[&action=approve|reject|revision]` — proposal drawer auto-open + focus action button
- `/dieu-phoi?taskId=X[&createFromProposal=Y]` — dispatch drawer auto-open
- `/tin-nhan?cid=X` — chat conversation
- `/ky-thuat/giao-viec?id=X` — KT work item

---

*Claude: Đọc file này TRƯỚC KHI làm bất kỳ thay đổi nào. Kiểm tra encoding trước khi sửa font/layout.*

---
> Source: [trihuong815-oss/greenpool-erp](https://github.com/trihuong815-oss/greenpool-erp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
