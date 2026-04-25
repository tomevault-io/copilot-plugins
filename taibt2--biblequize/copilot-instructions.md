## biblequize

> - Không bao giờ sửa code module khác khi đang làm 1 module

# Project: BibleQuiz

## Nguyên tắc tuyệt đối
- Không bao giờ sửa code module khác khi đang làm 1 module
- Mỗi thay đổi phải có test pass trước khi commit
- Không tự ý thêm dependency mới — hỏi trước
- Ưu tiên đọc TODO.md trước khi làm bất cứ thứ gì
- LUÔN chia nhỏ task vào TODO.md TRƯỚC khi code — không tự xử lý 1 lần

## Quy trình quản lý Task (BẮT BUỘC)

### Nguyên tắc cốt lõi
> **KHÔNG BAO GIỜ nhận prompt rồi chạy hết 1 lần.**
> Phải chia nhỏ → ghi TODO.md → làm từng task → check ✅ → task kế.

### Quy trình khi nhận prompt/task mới

```
1. ĐỌC TODO.md hiện tại → xem có task đang dở không
2. Nếu có task dở → hoàn thành nó trước (test pass, đánh ✅)
3. PHÂN TÍCH prompt mới → chia thành các task nhỏ (mỗi task 1 commit)
4. GHI tất cả tasks vào TODO.md theo format bên dưới
5. BẮT ĐẦU task đầu tiên
6. Xong task → chạy test → pass → đánh ✅ trong TODO.md → commit
7. Sang task tiếp theo → lặp lại bước 6
8. Hết tasks → chạy full regression → cập nhật TODO.md
```

### Format TODO.md

```markdown
## [Version/Phase] — [Tên nhóm task] [IN PROGRESS/DONE]

### Task [number]: [Tên task ngắn gọn]
- Status: [ ] TODO / [x] DONE / [!] BLOCKED
- File(s): [danh sách files sẽ sửa]
- Test: [test file tương ứng]
- Checklist:
  - [ ] Sub-step 1
  - [ ] Sub-step 2
  - [ ] Unit test pass
  - [ ] Commit: "[type]: [message]"
```

### Ví dụ cụ thể

Khi nhận prompt "Sync Home Dashboard từ Stitch + fix bugs + viết tests":

KHÔNG LÀM: đọc prompt → code hết 1 lần → commit 1 cục

LÀM ĐÚNG: chia thành TODO rồi làm từng task:
```markdown
## v2.5 — Sync Home Dashboard [IN PROGRESS]

### Task 1: Đọc Stitch design qua MCP + diff với code
- Status: [ ] TODO
- File(s): DESIGN_SYNC_AUDIT.md (output)
- Checklist:
  - [ ] MCP query Home screen
  - [ ] Liệt kê diff points
  - [ ] Ghi kết quả

### Task 2: Fix UNDEFINED energy bug
- Status: [ ] TODO
- File(s): GameModeGrid.tsx
- Test: GameModeGrid.test.tsx
- Checklist:
  - [ ] Xác định root cause (field name mismatch)
  - [ ] Fix code
  - [ ] Unit test cho null/undefined/loading cases
  - [ ] Vitest pass
  - [ ] Commit: "fix: energy UNDEFINED in GameModeGrid"

### Task 3: Cập nhật Welcome Bar theo Stitch
- Status: [ ] TODO
- File(s): Home.tsx
- Test: Home.test.tsx
- Checklist:
  - [ ] Compact layout (80px)
  - [ ] Greeting logic (sáng/chiều/tối)
  - [ ] Tier progress bar
  - [ ] Unit test
  - [ ] Vitest pass
  - [ ] Commit: "sync: Home welcome bar from Stitch"

### Task 4: Cập nhật Game Mode Grid theo Stitch
...

### Task 5: Cập nhật Leaderboard Preview
...

### Task 6: Fix warnings (useEffect → useQuery)
...

### Task 7: Full regression
- Status: [ ] TODO
- Checklist:
  - [ ] npx vitest run
  - [ ] npx playwright test
  - [ ] Backend tests
  - [ ] Số test >= baseline
```

### Rules
- Mỗi task phải ĐỦ NHỎ để commit riêng (1 task = 1 commit)
- KHÔNG gộp nhiều thay đổi vào 1 task
- KHÔNG skip task, làm theo thứ tự
- KHÔNG bắt đầu task kế nếu task hiện tại chưa ✅
- Sau mỗi task: cập nhật TODO.md NGAY (đánh ✅, ghi commit hash nếu cần)
- Nếu task bị BLOCKED: ghi lý do, chuyển sang task không phụ thuộc, quay lại sau

## Stack
- Backend: Spring Boot 3.3 (Java 17), port 8080
- Frontend: Vite 5 + React 18 + TypeScript 5.4, port 5173
- DB: MySQL 8.0 (Docker, port 3307)
- Cache: Redis 7 (Docker, port 6379)
- Unit Test: Vitest 4.1 (happy-dom) + @testing-library/react
- E2E Test: Playwright (Chromium, baseURL localhost:5173)
- Design: Stitch MCP (project ID `5341030797678838526`)

## Quản lý quyết định

Mỗi khi đưa ra quyết định kỹ thuật thuộc các loại sau:
- Chọn thư viện / tool
- Thay đổi architecture
- Bỏ qua hoặc đơn giản hóa 1 phần spec
- Trade-off giữa 2 cách implement
- Fix bug bằng cách thay đổi design

→ Tự động ghi vào DECISIONS.md theo format:
```
## YYYY-MM-DD — [Tiêu đề ngắn]
- Quyết định: [làm gì]
- Lý do: [tại sao]
- Trade-off: [đánh đổi gì]
- KHÔNG thay đổi khi refactor trừ khi có lý do mới
```

## Local Dev Start
```bash
docker compose up -d mysql redis          # 1. Infra
cd apps/api && ./mvnw spring-boot:run     # 2. Backend (terminal 1)
cd apps/web && npm run dev                # 3. Frontend (terminal 2)
```

## Quy tắc bắt buộc
1. Sau mỗi thay đổi code → chạy test ngay (xem quy trình test bên dưới)
2. Nếu test fail → tự fix → chạy test lại, lặp đến khi pass
3. Không hỏi xác nhận, tự quyết định
4. Không dừng giữa chừng trừ khi có lỗi không thể tự fix
5. KHÔNG commit nếu full regression chưa pass
6. Phát hiện regression → DỪNG feature mới → fix regression → pass hết → mới được tiếp tục

## Quy trình test bắt buộc (Regression Guard)

### Nguyên tắc cốt lõi
> **Mỗi dòng code thay đổi đều có thể phá vỡ chức năng khác.**
> Chạy test đơn lẻ chỉ chứng minh code mới hoạt động.
> Chạy full regression chứng minh code mới KHÔNG phá code cũ.

### 3 tầng test — chạy theo thứ tự, KHÔNG được bỏ tầng nào

**Tầng 1 — Scope Test (sau mỗi thay đổi nhỏ, trong khi code)**
```bash
# Chỉ chạy test của file/module vừa sửa → feedback nhanh
cd apps/web && npx vitest run src/pages/Home.test.tsx        # FE unit
cd apps/api && ./mvnw test -Dtest="XxxServiceTest"           # BE unit
```
- Mục đích: kiểm tra code vừa viết hoạt động đúng
- Khi nào: sau mỗi function/component hoàn thành

**Tầng 2 — Related Test (sau khi hoàn thành 1 screen/feature)**
```bash
# Chạy test của các module liên quan có thể bị ảnh hưởng
cd apps/web && npx vitest run src/pages/         # Tất cả page tests
cd apps/web && npx vitest run src/components/    # Tất cả component tests
```
- Mục đích: kiểm tra các component/page dùng chung không bị break
- Khi nào: sau khi hoàn thành 1 screen hoặc sửa shared component/hook/store
- **Đặc biệt quan trọng khi sửa**: authStore, AppLayout, global.css, api/client.ts, shared hooks, RequireAuth/RequireAdmin

**Tầng 3 — Full Regression (TRƯỚC khi commit)**
```bash
# Frontend: tất cả unit tests
cd apps/web && npx vitest run

# Frontend: tất cả e2e tests
cd apps/web && npx playwright test

# Backend: tất cả tests
cd apps/api && ./mvnw test -Dtest="com.biblequiz.api.**,com.biblequiz.service.**"
```
- Mục đích: đảm bảo KHÔNG có regression trên toàn bộ hệ thống
- Khi nào: **BẮT BUỘC** trước mỗi commit, không có ngoại lệ
- Kết quả mong đợi: tất cả tests pass, số test >= con số trước khi bắt đầu task

### Quy trình khi phát hiện regression

```
1. DỪNG ngay — không code thêm feature mới
2. Xác định test nào fail → đọc error message
3. Xác định nguyên nhân:
   a. Code mới phá logic cũ → sửa code mới cho compatible
   b. Test cũ outdated (assertion sai do UI thay đổi hợp lệ) → cập nhật test
   c. Shared dependency bị thay đổi → review impact, sửa tất cả chỗ bị ảnh hưởng
4. Sửa xong → chạy lại Full Regression (Tầng 3)
5. ALL PASS → mới được tiếp tục feature hoặc commit
```

### Các file "nhạy cảm" — khi sửa PHẢI chạy Full Regression ngay
| File | Lý do | Impact |
|------|-------|--------|
| `store/authStore.ts` | Global auth state | Mọi page dùng RequireAuth |
| `api/client.ts` | Axios interceptor, token | Mọi API call |
| `api/tokenStore.ts` | Token storage | Auth flow |
| `layouts/AppLayout.tsx` | Sidebar, nav | Mọi page trong AppLayout |
| `contexts/RequireAuth.tsx` | Auth guard | Mọi protected route |
| `contexts/RequireAdmin.tsx` | Admin guard | Mọi admin page |
| `styles/global.css` | Design tokens, utilities | Mọi component dùng glass-card, gold-gradient |
| `hooks/useStomp.ts` | WebSocket | RoomLobby, RoomQuiz |
| `main.tsx` | Routing | Mọi navigation |
| Backend: `SecurityConfig` | JWT, CORS | Mọi API endpoint |
| Backend: `GlobalExceptionHandler` | Error format | Mọi error response |

### Checklist trước commit (tự kiểm)
```
□ Tầng 1 pass — test file vừa sửa
□ Tầng 2 pass — test các module liên quan
□ Tầng 3 pass — full regression (FE unit + FE e2e + BE)
□ Số test KHÔNG giảm so với trước task (hiện tại baseline: 518)
□ Không có test bị skip/disabled mà trước đó đang pass
□ Nếu sửa file "nhạy cảm" → đã chạy full regression ngay sau khi sửa
```

---

## Cấu trúc package backend

```
com.biblequiz/
├── api/                    # REST Controllers + DTOs + WebSocket controllers
│   ├── dto/                # Request/Response DTOs
│   └── websocket/          # STOMP WebSocket controllers
├── infrastructure/         # Cross-cutting concerns (không chứa business logic)
│   ├── audit/              # Audit logging
│   ├── exception/          # GlobalExceptionHandler, custom exceptions
│   ├── security/           # JWT, OAuth2, RateLimiting filters
│   └── service/            # CacheService, monitoring
├── modules/                # Business logic, tổ chức theo domain
│   ├── achievement/        # entity/ + repository/ + service/
│   ├── auth/               # entity/ + repository/ + service/
│   ├── daily/              # service/
│   ├── group/              # entity/ + repository/ + service/  (Church Group)
│   ├── quiz/               # entity/ + repository/ + service/  (Question, Session, Answer)
│   ├── ranked/             # model/ + service/  (ScoringService, RankTier)
│   ├── room/               # entity/ + repository/ + service/  (4 game mode engines)
│   ├── season/             # entity/ + repository/ + service/
│   ├── share/              # entity/ + repository/ + service/  (Share Card)
│   ├── tournament/         # entity/ + repository/ + service/
│   └── user/               # entity/ + repository/ + service/  (User, Streak)
└── shared/                 # Utilities dùng chung giữa nhiều modules
    ├── aspect/             # AOP (performance monitoring)
    └── converter/          # JPA converters (JsonListConverter)
```

### Quy ước đặt file mới
- **Controller mới** → `api/XxxController.java` (không bao giờ đặt trong modules/)
- **Entity/Repository/Service mới** → `modules/{domain}/entity|repository|service/`
- **Module mới** → tạo thư mục `modules/{tên}/` với sub-folders entity/, repository/, service/
- **Filter, Security, Exception** → `infrastructure/{concern}/`
- **Converter, Aspect dùng chung** → `shared/` (chỉ cho utilities không thuộc domain nào)

### Shared/ — dùng cho gì, KHÔNG dùng cho gì
- **DÙNG**: JPA converters, AOP aspects, utility classes dùng bởi >= 2 modules
- **KHÔNG DÙNG**: Business logic, domain entities, DTOs, service classes — những thứ này thuộc modules/

---

## Cấu trúc frontend

```
apps/web/src/
├── api/                    # Axios client (client.ts) + token store (tokenStore.ts)
├── components/             # Shared reusable components + tests
│   └── ui/                 # Button, Card, Input, SearchableSelect
├── contexts/               # ErrorContext, RequireAuth, RequireAdmin
├── hooks/                  # useWebSocket, useStomp, useRankedDataSync
├── layouts/                # AppLayout, AdminLayout
├── pages/                  # 26 user pages + admin/
│   └── admin/              # 7 admin sub-pages
├── store/                  # Zustand (authStore.ts)
├── styles/                 # global.css (Tailwind + Stitch tokens)
└── test/                   # setup.ts (Vitest global setup)

apps/web/tests/             # Playwright e2e tests (*.spec.ts)
```

### Quy ước đặt file mới
- **Page mới** → `pages/XxxPage.tsx`, route thêm trong `main.tsx`
- **Component shared** → `components/XxxComponent.tsx` (hoặc `components/ui/` nếu primitive)
- **Hook mới** → `hooks/useXxx.ts`
- **Unit test** → cạnh source file: `pages/Xxx.test.tsx` hoặc `pages/__tests__/Xxx.test.tsx`
- **E2E test** → `tests/xxx.spec.ts` (Playwright testDir = `./tests`)
- **Admin page** → `pages/admin/XxxPage.tsx`

---

## Design System — "The Sacred Modernist"

### Stitch MCP
- **Server**: `https://stitch.googleapis.com/mcp` (HTTP transport)
- **Project ID**: `5341030797678838526`
- **Auth**: Google OAuth2 Bearer token (refresh: `gcloud auth print-access-token`)
- **Config**: `.mcp.json` at project root

### Design Tokens (bắt buộc tuân theo)
| Token | Value | Usage |
|-------|-------|-------|
| Background | `#11131e` | Page background |
| Surface Container | `#1d1f2a` | Cards, panels |
| Secondary (Gold) | `#e8a832` | CTA buttons, highlights, accents |
| Tertiary | `#e7c268` | Gold gradient end |
| On-Surface | `#e1e1f1` | Primary text |
| Error | Standard red | Error states |
| Font | Be Vietnam Pro | All text |
| Icons | Material Symbols Outlined | All icons |

### CSS Utilities (global.css — không tạo mới, dùng đúng class đã có)
```css
.glass-card     → Cards: rgba(50,52,64,0.6) + backdrop-blur(12px)
.glass-panel    → Panels: rgba(50,52,64,0.6) + backdrop-blur(20px)
.gold-gradient  → CTA buttons: linear-gradient(135deg, #e8a832, #e7c268)
.gold-glow      → Hover effect: box-shadow 0 0 20px rgba(232,168,50,0.2)
.streak-grid    → Heatmap: grid 20 columns
.timer-svg      → Quiz timer: rotate(-90deg)
```

### Quy tắc UI bắt buộc
- Mọi screen mới phải dùng design tokens ở trên — KHÔNG được hardcode màu khác
- Card → dùng `.glass-card`, KHÔNG tự tạo card style mới
- CTA button → dùng `.gold-gradient`, KHÔNG dùng màu khác cho primary action
- Background → luôn `#11131e`, KHÔNG dùng black hay dark gray khác
- Font → Be Vietnam Pro only, import từ Google Fonts
- Khi có Stitch design → phải match pixel-perfect
- Khi không có Stitch design (custom) → follow cùng design tokens + pattern từ screens đã sync
- Responsive: mobile-first, breakpoints theo Tailwind default (sm/md/lg/xl)

---

## Quy ước code bắt buộc

### Backend
- Primary key: UUID v7, không dùng auto-increment
- Mọi Entity phải có: id, createdAt, updatedAt
- Mọi API lỗi trả về: { code, message, requestId, details? }
- Không bao giờ expose stack trace trong response
- Dùng MapStruct để map Entity ↔ DTO
- Mọi thay đổi DB phải có Flyway script: db/migration/V{n}__{description}.sql
- Mọi endpoint /admin/** phải có @PreAuthorize("hasRole('ADMIN')")
- Dùng @Transactional cho mọi thao tác ghi nhiều bảng

### Frontend
- Mọi API call qua TanStack Query — không dùng useEffect + fetch thủ công
- State global dùng Zustand — không dùng Context cho global state
  - Exception: ErrorContext (tree-scoped, render toasts trong React tree) được phép giữ Context
  - Auth state đã migrate sang `src/store/authStore.ts` (Zustand)
- Không hardcode URL — dùng import.meta.env.VITE_API_URL
- Mọi form phải có loading state + error handling
- Component không quá 300 LOC — nếu vượt, tách thành sub-components
- Không inline style — dùng Tailwind classes hoặc global CSS utilities
- Mọi page phải handle 3 states: loading (skeleton), error (message + retry), success (content)

---

## Quy tắc test

### Nguyên tắc chung
- Mỗi screen/component PHẢI có unit test
- Mỗi user flow PHẢI có e2e test
- Test không được mock/hardcode config values khác config thật
- Khi test service cần config value → dùng giá trị giống application-dev.yml
- KHÔNG commit code mà test đang fail

### Unit Test (Vitest)
- **Config**: `vitest.config.ts` (happy-dom, setup `src/test/setup.ts`)
- **Pattern**: `src/**/*.{test,spec}.{ts,tsx}`
- **Đặt file**: cạnh source `Xxx.test.tsx` hoặc trong `__tests__/Xxx.test.tsx`
- **Minimum per screen**: 8 test cases (render, props, state, interactions, loading, error, responsive, accessibility)
- **Mock strategy**:
  - API calls → mock TanStack Query hooks hoặc MSW
  - Navigation → mock `useNavigate` from react-router-dom
  - Auth state → mock Zustand store
  - WebSocket → mock useStomp/useWebSocket hooks
  - KHÔNG mock implementation details — test behavior, not internals

### E2E Test (Playwright)
- **Config**: `playwright.config.ts` (Chromium, serial, baseURL localhost:5173)
- **testDir**: `./tests` (KHÔNG phải `./e2e`)
- **Đặt file**: `tests/<screen-name>.spec.ts`
- **Minimum per screen**: 5 test cases (happy path, navigation in/out, key interactions, error state, mobile viewport)
- **Quy tắc**:
  - Cần app đang chạy (dev server + backend + DB)
  - Dùng `page.goto()` với relative path (baseURL đã set)
  - Screenshot on failure (đã config)
  - Trace on first retry (đã config)
  - KHÔNG dùng `page.waitForTimeout()` cứng — dùng `page.waitForSelector()` hoặc `expect().toBeVisible()`
  - Test mobile: dùng `test.use({ viewport: { width: 375, height: 667 } })`

### Backend Test
- Unit test không dùng H2 — dùng Testcontainers MySQL
- Integration test phải kiểm tra config file thật
- Mọi Service method PHẢI có unit test

---

## Lệnh test

```bash
# Backend
cd apps/api && ./mvnw test -Dtest="com.biblequiz.api.**"         # API tests
cd apps/api && ./mvnw test -Dtest="com.biblequiz.service.**"     # Service tests
cd apps/api && ./mvnw test -Dtest="com.biblequiz.api.**,com.biblequiz.service.**"  # All backend

# Frontend unit
cd apps/web && npm test                     # Vitest watch mode
cd apps/web && npx vitest run               # Vitest single run (CI)
cd apps/web && npx vitest run src/pages/    # Vitest chỉ pages

# E2E (cần app đang chạy trên localhost:5173 + backend 8080)
cd apps/web && npx playwright test                        # All e2e
cd apps/web && npx playwright test tests/home.spec.ts     # Single file
cd apps/web && npx playwright test --headed               # Có browser UI
cd apps/web && npx playwright show-report                 # Xem HTML report
```

---

## Commit Convention

```
feat: <mô tả>       # Tính năng mới
fix: <mô tả>        # Sửa bug
refactor: <mô tả>   # Refactor không thay đổi behavior
test: <mô tả>       # Thêm/sửa test
style: <mô tả>      # UI/CSS changes (không thay đổi logic)
sync: <mô tả>       # Sync design từ Stitch
docs: <mô tả>       # Documentation
chore: <mô tả>      # Build, config, tooling
```

Ví dụ:
- `sync: Home dashboard from Stitch v5 + tests`
- `feat: add TournamentMatch page with 1v1 gameplay`
- `fix: quiz timer not resetting between questions`
- `test: add e2e tests for login flow`

---

## Approved Dependencies (không cần hỏi)

### Frontend — đã có, thoải mái dùng
- react, react-dom, react-router-dom
- @tanstack/react-query
- zustand
- axios
- @stomp/stompjs
- tailwindcss
- vitest, @testing-library/react, @testing-library/user-event
- @playwright/test

### Frontend — CẦN HỎI trước khi thêm
- Bất kỳ UI library mới (shadcn, radix, headless-ui, ...)
- Animation library (framer-motion, react-spring, ...)
- Form library (react-hook-form, formik, ...)
- Chart library mới (hiện dùng inline SVG)
- Bất kỳ dependency nào chưa có trong package.json

### Backend — đã có, thoải mái dùng
- Spring Boot starters (web, data-jpa, security, websocket, validation, cache)
- Flyway, JJWT, springdoc-openapi, spring-dotenv, MapStruct
- Testcontainers, JUnit 5, Mockito

---

## Workflow khi làm feature mới

```
1. Đọc TODO.md → xác định task cần làm
2. Nếu prompt mới → CHIA NHỎ thành tasks → GHI VÀO TODO.md trước
3. Bắt đầu task đầu tiên trong TODO.md
4. Nếu có Stitch design → MCP query Stitch lấy design → code match pixel-perfect
5. Nếu không có Stitch design → follow design tokens + pattern từ screens đã sync
6. Code từng phần nhỏ → Tầng 1 test (scope test) sau mỗi phần
7. Hoàn thành screen → viết unit test đầy đủ (Vitest, min 8 cases)
8. Unit test pass → viết e2e test (Playwright, min 5 cases) nếu là screen/flow mới
9. Tầng 2 test (related modules) → fix nếu có regression
10. Tầng 3 test (FULL REGRESSION) → BẮT BUỘC pass hết trước khi commit
11. Kiểm tra: số test >= baseline, không có test bị skip
12. Update TODO.md → đánh ✅ task vừa xong
13. Commit theo convention (1 task = 1 commit)
14. Nếu có quyết định kỹ thuật → ghi DECISIONS.md
15. Chuyển sang task tiếp theo trong TODO.md → lặp lại từ bước 3
```

## Workflow khi sync Stitch design

### Nguyên tắc cốt lõi
> **Stitch HTML là source of truth. Code PHẢI replicate từng section, không được tự ý bỏ bớt hoặc thêm.**
> Nếu Stitch có 8 sections → code phải có 8 sections. Không được "simplified version".

```
1. Đọc TODO.md → nếu là prompt mới → chia thành tasks per screen → ghi TODO.md
2. Bắt đầu task đầu tiên (1 screen = 1 task)
3. Đọc Stitch HTML file:
   - Nếu có trong docs/designs/stitch/ → đọc trực tiếp
   - Nếu chưa có → MCP Stitch query → save HTML → rồi đọc
4. LIỆT KÊ từng section trong Stitch HTML (viết ra comment hoặc terminal):
   - Section 1: [tên] — [mô tả nội dung] — [Tailwind classes chính]
   - Section 2: [tên] — [mô tả] — [classes]
   - ... liệt kê TOÀN BỘ, không bỏ sót
5. Đọc code hiện tại → liệt kê sections trong code
6. DIFF bắt buộc (viết ra):
   | # | Stitch Section | Code Section | Match | Action |
   |---|---------------|-------------|-------|--------|
   | 1 | KPI Cards | KPI Cards | 🔄 70% | Update sub-stats |
   | 2 | Activity Log | KHÔNG CÓ | ❌ | THÊM MỚI |
   | 3 | ... | ... | ... | ... |
7. Thực hiện TỪNG action trong bảng diff:
   - ❌ KHÔNG CÓ → tạo component mới match Stitch HTML
   - 🔄 Partial → update cho khớp 100%
   - ✅ Match → giữ nguyên
   - Code có nhưng Stitch KHÔNG CÓ → XÓA (trừ khi là business logic cần thiết)
8. Với mỗi section mới/update: copy Tailwind classes từ Stitch HTML, adjust cho React
9. KHÔNG thay đổi business logic — chỉ sửa UI/styling
10. SAU KHI CODE XONG — verify lại bảng diff: tất cả sections phải ✅
11. Tầng 1 → Tầng 2 → Tầng 3 test
12. Update TODO.md → đánh ✅
13. Commit: `sync: <ScreenName> from Stitch + tests`
```

### Cách đọc Stitch HTML file
```
1. Mở file HTML trong docs/designs/stitch/
2. Tìm tất cả top-level <div> hoặc <section> → đó là các sections
3. Với mỗi section:
   - Đọc Tailwind classes → giữ nguyên khi chuyển sang React
   - Đọc text content → hiểu section làm gì
   - Đọc nested structure → replicate component tree
4. Colors trong HTML → map sang code variables (nếu có)
5. Icons trong HTML → dùng cùng icon library
6. KHÔNG ĐƯỢC tự ý:
   - Bỏ section nào trong Stitch
   - Thêm section không có trong Stitch
   - Đổi layout grid khác Stitch
   - Đổi color khác Stitch
   - "Simplified" bất kỳ section nào
```

---

## Definition of Done
- Full Regression pass (Tầng 3): Vitest + Playwright + JUnit — tất cả green
- Số test >= baseline trước task (không được giảm)
- Không có test bị skip/disabled mà trước đó đang pass
- Không có TypeScript/Java compile error
- Không có @SuppressWarnings mới
- Flyway migration chạy clean trên DB trống
- Chạy được trên local end-to-end
- UI match design tokens (nếu có Stitch → pixel-perfect)
- Loading, error, success states đều handled

## KHÔNG được làm
- Không dùng H2 in-memory cho test — dùng Testcontainers MySQL
- Không map Entity → DTO thủ công — dùng MapStruct
- Không để business logic trong Controller — chỉ ở Service
- Không commit code có System.out.println — dùng @Slf4j + log
- Không xóa Flyway migration đã chạy — tạo migration mới để fix
- Không hardcode màu/font ngoài design tokens
- Không tạo CSS utility mới khi đã có class tương đương trong global.css
- Không dùng `page.waitForTimeout()` trong Playwright — dùng proper waits
- Không commit code mà test đang fail
- Không để 1 file component > 300 LOC
- Không skip Full Regression (Tầng 3) trước commit — dù chỉ sửa 1 dòng CSS
- Không disable/skip test cũ để làm test mới pass — fix root cause
- Không tiếp tục feature mới khi đang có regression chưa fix
- Không chỉ chạy test file vừa sửa rồi commit — phải qua đủ 3 tầng
- Không nhận prompt rồi code hết 1 lần — PHẢI chia nhỏ tasks vào TODO.md trước
- Không gộp nhiều thay đổi vào 1 commit — 1 task = 1 commit
- Không bỏ qua cập nhật TODO.md sau mỗi task hoàn thành
- Không bỏ section nào có trong Stitch HTML khi sync — phải replicate TOÀN BỘ
- Không thêm section mà Stitch design KHÔNG CÓ (trừ khi business logic bắt buộc)
- Không "simplified version" khi sync Stitch — phải pixel-perfect
- Không sync Stitch mà không viết DIFF table trước — phải liệt kê sections trước khi code
- Không tự ý đổi layout/grid/color khác Stitch HTML — Stitch là source of truth
## Mobile Code Rules
- TẤT CẢ business logic PHẢI nằm trong src/logic/ hoặc src/utils/
- Components CHỈ chứa UI render + gọi logic functions
- Mỗi logic file PHẢI có test file tương ứng
- KHÔNG viết logic trong component
## Mobile Testing Rules

### Bắt buộc mỗi PR (Claude Code tự chạy):

1. **Logic test cho MỌI function trong src/logic/ và src/utils/**
   - Chạy: `npm test`
   - Target: 100% logic functions có test
   
2. **Component test cho MỌI screen và component mới/sửa**
   - Render test: component hiện đúng text, elements
   - Interaction test: press, input, navigation
   - Chạy: `npm test`
   
3. **Snapshot test cho MỌI screen**
   - Tạo snapshot lần đầu
   - Verify snapshot không thay đổi ngoài ý muốn
   
4. **TypeScript strict — no any**
   - `"strict": true` trong tsconfig.json
   - Type errors = bugs tiềm ẩn

### Code structure bắt buộc:

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TaiBT2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
