## prectxe-site

> Always prefer Serena's symbolic tools (find_symbol, get_symbols_overview, replace_symbol_body, etc.) over grep, read_file, and text-based edits for code search and modification.

# CLAUDE.md

## Tool Usage
Always prefer Serena's symbolic tools (find_symbol, get_symbols_overview, replace_symbol_body, etc.) over grep, read_file, and text-based edits for code search and modification.
This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Dev Commands

- `bun run dev` — Next.js dev server (Turbopack)
- `bun run build` — `prisma generate` + `next build`
- `bun run type-check` — `tsc --noEmit`
- `bun run check` — Biome lint + format combined
- `bun run check:fix` — Auto-fix Biome issues
- Before pushing: `bun run type-check && bun run check`
- Prisma: `bunx prisma migrate dev -n "<msg>"`, `bunx prisma generate`, `bunx prisma studio`
- Pre-commit hook (Husky): `biome check --write --no-errors-on-unmatched` via lint-staged
- **No test framework** — no unit/integration test setup exists yet

## Tech Stack

- **Next.js 16** (App Router) + **React 19** + **TypeScript** (strict) + **Bun**
- **PostgreSQL** via **Prisma** ORM (Neon), **Iron Session** (cookie auth, 7-day expiry)
- **Biome v2** for lint/format — line width 80, single quotes (JS), double quotes (JSX), auto-organizes imports
  - Key rules: `noUnusedImports: warn`, `noExplicitAny: warn`, `useImportType: warn`, `noUnusedVariables: warn`, `useExhaustiveDependencies: warn`
- **Tailwind CSS v3** + **shadcn/ui** (Radix) + **CVA**
- **TanStack Query** (client state — `refetchOnFocus: false`, `retry: 3`), **React Hook Form** + **Zod** (validation)
- **TipTap** (rich text), **@dnd-kit** (drag-drop), **react-day-picker** (calendar)
- **Cloudflare Images/Stream** (media hosting), **PortOne V2** (payments)
- **Resend** (email), **Aligo/Solapi** (SMS — provider via `SMS_PROVIDER` env)
- Deployed on **Vercel**

## Project Structure

```
src/
├── app/                    # Next.js App Router
│   ├── (home)/             # 홈페이지 — 단일 page.tsx
│   ├── (content)/          # Public 콘텐츠 흐름 (programs, journal, drops, archive, discover, tickets/order)
│   ├── (page)/             # Static info + entity directory (about, terms, privacy, refund-policy, partnership, artists, artworks, venues, forms)
│   ├── (auth)/             # 인증 + admin 그룹 (admin layout 포함)
│   ├── api/                # API routes
│   └── scan/               # QR 외부 카메라 fallback (/scan/[token])
├── modules/<domain>/       # Feature modules
│   ├── server/actions.ts   # Server actions ('use server')
│   └── ui/
│       ├── views/          # 페이지 단위 큰 컴포넌트 (DropDetailView, FormBuilderView 등)
│       └── components/     # 작은 building block — form, card, list, section 등 모두 (※ 별도 section 폴더 없음, 통합됨)
├── components/             # 공용 UI
│   ├── ui/                 # shadcn primitives (Button, Dialog, ConfirmDialog 등)
│   ├── admin/              # admin 공통 (AdminNav, DeleteButton 등)
│   ├── layout/             # 사이트 layout (Header, Footer, PublicHeader, LegalPageLayout)
│   ├── shared/             # 작은 공용 utility (BackButton, FadeIn, FilterChip 등)
│   ├── icons/              # SVG 아이콘
│   ├── media/              # 미디어 표시 (CloudflareStreamVideo, MediaGallery, SortableMediaList)
│   ├── image/              # 이미지 표시·업로드 박스 (CarouselGallery, MultiImageBox, SingleImageBox)
│   ├── rich-editor/        # TipTap 에디터 — journal/drops에서 공유
│   └── seo/                # JSON-LD schema 컴포넌트
├── hooks/                  # 공용 커스텀 훅 (use-toast, use-single-image-upload 등)
├── lib/                    # 인프라 + 유틸 (auth, cdn, db, email, payment, pnl, sms, schemas, utils)
└── middleware.ts            # Route protection
```

**Import paths**: `@/*` → `./src/*`. **Filenames**: kebab-case. **UI language**: Korean.

### 폴더링 컨벤션

**`components/<topic>/` vs `modules/<x>/ui/components/`**:
- 여러 모듈에서 공유하는 컴포넌트는 `components/<topic>/`. 단일 모듈 전용은 `modules/<x>/ui/components/`.
- 예: `email-editor`는 email 모듈만 쓰니 `modules/email/ui/components/email-editor/`. `rich-editor`는 journal+drops 둘 다 쓰니 `components/rich-editor/`.

**`(content)` vs `(page)` 라우트 그룹**:
- `(content)`: 사이트의 콘텐츠 흐름(브랜드 메인 동선) — programs, journal, drops, archive, discover.
- `(page)`: 디렉토리/static info — entity 목록·상세(artists, artworks, venues, forms), 약관(terms, privacy, refund-policy).
- 굳이 분리할 가치가 모호하면 같은 그룹에 둬도 OK. 분류는 navigation 흐름 기준이지 보안 기준이 아님.

**`lib/utils/`** — 토픽별 분리:
- `cn.ts`(Tailwind), `date.ts`, `image-url.ts`, `upload.ts`, `text.ts`, `ticket-status.ts`, `ticket-token.ts`, `bank-transfer.ts`, `media-upload.ts`
- `index.ts`에서 모두 re-export → 호출자는 `@/lib/utils`로 import

### Modules

`programs`, `journal`, `artists`, `venues`, `artworks`, `auth`, `forms`, `sms`, `email`, `drops`, `tickets`, `home`, `estimates`, `pnl` — each with `server/actions.ts` + `ui/{views,components}/`. Query provider: `src/modules/providers.tsx`.

## Key Patterns

### Server Actions
- `'use server'` directive, return `{ success, error? }` or `{ success, data? }`
- Admin guard: `requireAdmin()` → `{ success, userId }`
- Auth check: `canManage(sessionId, authorId?)` — ADMIN or resource owner
- **Important**: `'use server'` files cannot export synchronous functions — move utility functions to `@/lib/utils`

### React 19 + Radix UI Compatibility
- Radix Select `name` prop causes infinite re-render inside `<form>` — use controlled state + hidden `<input>` instead
- Inline callback functions in hook deps cause infinite loops — use `useRef` pattern

### Image Upload (Cloudflare)
- Single-use upload URLs — must request new URL on retry
- Variants: `thumbnail`, `public`, `smaller`, `hires` via `getImageUrl(url, variant)`
- `use-single-image-upload` and `use-multi-image-upload` hooks
- `uploadPendingWithProgress()` returns `{ images }` synchronously for payload use (React state is async)
- Max file size: 50MB. Allowed types: JPEG, PNG, GIF, WEBP, HEIC
- HTML body image cleanup: `cleanupRemovedHtmlImages(oldHtml, newHtml)` auto-deletes removed Cloudflare images on content update

### Drops & Payments
- Drop types: `ticket` (TicketTier) and `goods` (GoodsVariant)
- **현재 결제는 무통장 입금 단독 라이브** (PG 심사 거절 사유, 자세한 컨텍스트는 메모리 참고). PortOne 카드 결제 코드는 `tickets/server/actions.ts`에 dormant 유지.
- Ticket 무통장: `createBankTransferOrder` → BankTransfer(pending, 24h 만료, depositorName=이름+주문번호4자리) + 재고 즉시 차감 + 안내 메일(`bank-transfer-pending`)
- 어드민 입금 확인: `confirmBankTransfer` → Order=paid + BankTransfer=confirmed + Ticket 발급 + Order.accessToken 발급 + 확정 메일(`order-confirmation`)
- Ticket 무료(0원): `createOrder` → `verifyPayment(orderId, 'free-...')`로 즉시 paid 처리 (PortOne 우회). Ticket·accessToken 같이 발급
- Ticket 카드 (dormant): `createOrder` → PortOne `requestPayment` → `verifyPayment` — UI에서 호출만 차단, 코드는 살아있음
- Goods 카드 (dormant): `createGoodsOrder` → PortOne `requestPayment` → `verifyPayment`
- `cancelOrder`: 재고 복구 + Payment cancelled + BankTransfer cancelled + Ticket cancelled (전부 cascade)
- 만료 처리: `cleanupExpiredBankTransferOrders` — 어드민 주문 페이지 진입 시 lazy 호출, 24h 미입금 자동 취소 + 재고 복구
- Drop visibility: requires `status !== 'draft'` AND `publishedAt !== null`
- Status change from draft auto-sets `publishedAt`
- Drop statuses: `draft` → `upcoming` → `on_sale` → `sold_out` / `closed`
- **Media**: images and videos unified in `DropMedia` model (`type: image | video`, `order` for DnD sort). No separate `DropImage`/`heroUrl`/`videoUrl` fields — first media by `order` acts as hero. Image `url` = Cloudflare Images URL; video `url` = Cloudflare Stream ID (HLS via `hls.js`).

### Tickets & QR 입장 시스템
- `Order.accessToken` (paid 시 발급) + `Ticket.token` (1장당 unguessable 토큰, randomBytes 16 → hex 32자)
- 구매자 마이페이지: `/tickets/order/[accessToken]` — 토큰만으로 접근(인증 X), QR SVG inline 렌더(qrcode lib)
- QR 페이로드 = URL: `${SITE_URL}/scan/${ticketToken}`
- 어드민 스캐너: `/admin/drops/[id]/scanner` — html5-qrcode 풀스크린, `fixed inset-0 z-100`로 admin layout 위에 덮음. `extractTicketToken`이 URL/raw 둘 다 인식
- 1.5초 디바운스로 동일 QR 중복 스캔 방지, sound feedback (Web Audio API)
- Fallback `/scan/[token]` — 외부 카메라 앱이 인식했을 때 도달, 어드민이면 스캐너 페이지 안내, 일반 사용자에겐 운영자 안내

### Email Templates
- Available templates in `src/lib/email/templates/`: `form-notification`, `newsletter`, `order-confirmation`, `bank-transfer-pending`
- Sent via Resend API through `src/lib/email/send.ts`
- `order-confirmation`은 paid 시점에 발송, `ticketsUrl` prop을 받아 "입장권 보기" CTA 자동 노출
- `bank-transfer-pending`은 무통장 주문 직후 발송 — 계좌 + 정확한 입금자명 + 마감시각 + 자동 취소 안내

### Newsletter Broadcasts (Resend Segments)
- 구독자는 Resend에 저장(`subscribeNewsletter` 액션) — 자체 DB에 구독자 모델 없음
- Resend 2026부터 Broadcasts API는 `segment_id` 필수. `src/lib/email/segments.ts`의 `getOrCreateNewsletterSegmentId()`가 기본 세그먼트(이름: `RESEND_SEGMENT_NAME` env, 기본 `Newsletter`)를 자동 탐지/생성
- 구독 시 `contacts.create` + `contacts.segments.add` 동시 수행 — 이미 구독된 사용자도 segment에 없으면 자동 편입(idempotent)
- 어드민 `/admin/email` > **뉴스레터 발송** 탭에서 `broadcasts.create({send:true})` 호출. 발송 기록은 `EmailCampaign.broadcastId`에 저장(수신자 목록은 Resend가 관리, `EmailRecipient` 미사용)
- 템플릿 푸터에 `{{{RESEND_UNSUBSCRIBE_URL}}}` 플레이스홀더 — Resend가 수신자별로 자동 치환

### OG Images
- Programs, Journal, Drops all have `opengraph-image.tsx` route handlers
- Noto Sans KR web font loaded for Korean text rendering

### Caching (read-path) — 캐시 + 편집 즉시 반영
공개 read 경로는 `unstable_cache`(별칭 `next_cache`)로 캐싱. 도메인 태그 + 서버액션 무효화로 트래픽엔 캐시, 어드민 편집엔 즉시 반영(2026-06-19 전 모듈 적용·프로덕션 검증).
- **패턴**: `next_cache(fn, ['key'], { revalidate: CACHE_TIMES.X, tags: ['domain'] })` + 편집 mutation(server action)에서 `updateTag('domain')`. `'domain'` = `programs|artists|journal|drops` 등.
- **중요**: `revalidatePath`는 route 캐시만 비움 → `unstable_cache` 데이터는 **무효화 안 됨**. 즉시 반영엔 반드시 `tags` + `updateTag`(또는 무프로파일 `revalidateTag`). `updateTag`는 서버액션 전용·즉시(read-your-own-writes), Next 16.2.4에서 unstable_cache와 동작 확인.
- **Date 직렬화**: `unstable_cache`는 Date를 ISO 문자열로 직렬화 → 캐시 함수에서 `.toISOString()` 변환하거나, 소비되는 날짜만 wrapper에서 `new Date()`로 복원(타입·런타임 일치). 소비 뷰가 이미 `new Date(...)`로 감싸면 안전.
- **i18n 대비**: 로케일별 라벨/날짜 포맷은 캐시 함수가 아닌 **렌더 시점**에 계산(캐시 키에 로케일 없음).
- 홈 featured-hero는 program/article/drop 모두 읽으므로 `tags: ['programs','journal','drops']`. 어드민 편집 폼은 캐시 미사용 경로(예: artists `getArtistById` vs `getArtistByIdWithCache`).
- 자세한 Next 16 캐시 API 변경/검증은 메모리 `project_nextjs16_cache_api` 참고.

## Database

- **Schema**: `prisma/schema.prisma`
- **Key models**: Program, Article, Artist, Venue, Artwork, Form/FormField/FormSubmission, Drop, TicketTier, GoodsVariant, Order/OrderItem, Payment, BankTransfer, Ticket, User
- **Migration caveat**: Neon에 `main`(prod) + `program-model-v1`(dev) 2개 브랜치. `bunx prisma db push`는 **dev에서만 안전**. main에는 Neon MCP `run_sql_transaction`으로 부분 마이그레이션. legacy 테이블 정리 후 enum 타입은 별도 `DROP TYPE` 필요 (자동 안 됨). 자세한 패턴은 메모리 `project_db_branches` 참고.
- **Prisma client**: Singleton pattern in `src/lib/db/prisma.ts` (global ref to prevent multiple instances in dev)
- **FK 인덱스**: Prisma는 PostgreSQL **외래키에 인덱스를 자동 생성하지 않음** → 새 모델/관계 추가 시 자주 조회하는 FK 컬럼에 `@@index([fkColumn])` 직접 추가할 것(없으면 부모→자식 `include`가 seq scan). 2026-06-19에 핫/성장 테이블 FK 26개 인덱스 일괄 추가(prod + schema 동기화).

### Form Data Safety (Critical)
- **NEVER** physically delete FormField — use soft delete (`archived: true`)
- All field queries filter `where: { archived: false }` except `getFormSubmissions()`

## Auth & Admin

- Iron Session cookie (`prectxe`): `id`, `name`, `isAdmin`
- 현재 시스템은 **어드민 전용 인증**. 일반 회원 가입/로그인 기능은 보류 상태 (메모리 `project_member_system_deferred` 참고)
- Public: `/programs/[slug]`, `/drops/[slug]`, `/journal/[slug]`, `/forms/[slug]` 등
- **Token 기반 public** (middleware matcher 외): `/tickets/order/[accessToken]` (구매자 마이페이지), `/scan/[token]` (외부 카메라 fallback)
- Private (`/admin/*`, `/*/new`, `/*/[id]/edit`): 미들웨어가 ADMIN 체크 후 진입 허용. 일반 회원은 / 로 redirect
- Public-only (로그인 시 /admin redirect): `/auth/signin`, `/auth/signup`
- 입장 스캐너: `/admin/drops/[id]/scanner` — admin layout 안에 있지만 풀스크린 fixed 컨테이너로 nav 시각적 우회
- Admin dashboard: 10 tabs + Drops/Orders/Revenue stats

## Next.js Config Notes

- **Server external packages**: `aligoapi`, `solapi` (required for SSR)
- **Image remote patterns**: `imagedelivery.net` (Cloudflare), `avatars.githubusercontent.com`, `assets.zyrosite.com`
- **Legacy redirects** (enabled with `ENABLE_PROGRAM_REDIRECTS=1`): `/projects/:slug` → `/programs/:slug`, `/events/:slug` → `/programs/:slug`, `/discover` → `/programs?status=upcoming`, `/archive` → `/programs?status=completed`

## Environment Variables

Required: `DATABASE_URL`, `NEXT_PUBLIC_SITE_URL`, `CLOUDFLARE_IMAGE_STREAM_API_ACCOUNT_ID`, `CLOUDFLARE_IMAGE_STREAM_API_TOKEN`, `CLOUDFLARE_IMAGE_STREAM_API_ACCOUNT_HASH`, `CLOUDFLARE_STREAM_CUSTOMER_CODE`, `COOKIE_PASSWORD` (min 32 chars)

Payment: `PORTONE_API_SECRET`, `PORTONE_WEBHOOK_SECRET`, `PORTONE_STORE_ID`, `PORTONE_CHANNEL_KEY`, `NEXT_PUBLIC_PORTONE_STORE_ID`, `NEXT_PUBLIC_PORTONE_CHANNEL_KEY`

Bank Transfer: `BANK_NAME`, `BANK_ACCOUNT_NUMBER`, `BANK_ACCOUNT_HOLDER`, `BANK_TRANSFER_EXPIRY_HOURS` (default 24)

Email: `RESEND_API_KEY`, `RESEND_SENDER_EMAIL`. 뉴스레터는 Resend Segment 기반 — `RESEND_SEGMENT_NAME` (선택, 기본 `Newsletter`)로 기본 세그먼트 자동 생성/재사용. SMS: `SMS_PROVIDER` (`aligo`|`solapi`) + 해당 provider keys.

Optional: `NEXT_PUBLIC_GA_ID`, `ENABLE_PROGRAM_REDIRECTS`, `TEST_ADMIN_*` (개발용)

## Development Workflow

- **Dev log**: `docs/dev-log.md` (recent entries only, archive in `docs/dev-log-archive.md`)
- **Commits**: Concise Korean/English, present tense. Optional scope: `module: message`
- **New features**: Create module → Zod schema → server actions → UI → routes → middleware
- **Pagination default**: `DEFAULT_PAGE_SIZE = 6` (in `src/lib/constants/constants.ts`)

---
> Source: [iamkimseunghyun/prectxe-site](https://github.com/iamkimseunghyun/prectxe-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
