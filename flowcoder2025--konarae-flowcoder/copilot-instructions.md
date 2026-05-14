## konarae-flowcoder

> > **프로젝트 정보**: `/docs/prd.md` 참조

# 루트 가이드 (헌법) - 허브

> **프로젝트 정보**: `/docs/prd.md` 참조
>
> **🏛️ 역할**: 전역 원칙 + 중앙 허브 (모든 하위 가이드의 진입점)
> **🔗 구조**: Hub-Spoke 아키텍처 - 이 파일에서 모든 하위 가이드로 연결

```
                    ┌─────────────────┐
                    │   CLAUDE.md     │
                    │   (루트 허브)    │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
   ┌─────────┐         ┌─────────┐         ┌─────────┐
   │ /src/app│         │/src/lib │         │ /prisma │
   │  (라우팅) │         │ (백엔드) │         │  (DB)   │
   └─────────┘         └─────────┘         └─────────┘
        │
        ├── /components ── /features ── /hooks ── /types
```

---

## 1. 프로젝트 컨텍스트

> ⚠️ **필수**: 작업 시작 전 `/docs/prd.md`를 먼저 읽어 프로젝트 정보를 파악하세요.

| 항목 | 참조 위치 |
|-----|----------|
| 프로젝트명 | `/docs/prd.md` → 프로젝트명 섹션 |
| 목적 | `/docs/prd.md` → 목적 섹션 |
| 범위 | `/docs/prd.md` → 범위/스코프 섹션 |
| 대상 사용자 | `/docs/prd.md` → 타겟 유저 섹션 |

### 1.1 이 Skill의 핵심 가치

**"Primary Color만 바꾸면 브랜드 완성"**

- 디자인 토큰 기반 일관된 UI 시스템
- 자연어 요청 시 정의된 룰에 따라 완성형 UI 생성
- 프로젝트 독립적 재사용 가능한 구조

---

## 2. 기술 스택

| 영역 | 기술 |
|-----|------|
| Framework | Next.js 15 + React 19 + TypeScript |
| Styling | Tailwind CSS 4 + shadcn/ui (new-york) |
| Database | Supabase PostgreSQL + Prisma ORM + pgvector |
| Auth | NextAuth.js + ReBAC 권한 시스템 |
| AI/ML | Gemini 2.5 Pro Vision (문서 분석) + OpenAI Embeddings |
| Storage | Supabase Storage |
| Deploy | Vercel (Main) + Railway (Worker) |

**참조 스킬**: `fdp-backend-architect` (백엔드 아키텍처)

---

## 3. 허브-스포크 네비게이션 맵

### 3.1 📍 빠른 참조 (Quick Navigation)

| 작업 목적 | 가이드 위치 | 핵심 내용 |
|----------|------------|----------|
| **페이지/API 개발** | `/src/app/claude.md` | App Router, API Routes, 라우팅 패턴 |
| **UI 컴포넌트** | `/src/components/claude.md` | shadcn/ui, Button, Modal, 접근성 |
| **기능 모듈** | `/src/features/claude.md` | Feature 구조, 상태관리, 네이밍 |
| **백엔드/유틸** | `/src/lib/claude.md` | Prisma, Auth, ReBAC, i18n |
| **커스텀 훅** | `/src/hooks/claude.md` | 훅 패턴, 네이밍, 테스트 |
| **타입 정의** | `/src/types/claude.md` | 타입 체계, 네이밍 컨벤션 |
| **DB 스키마** | `/prisma/claude.md` | 모델 관계, 마이그레이션, pgvector |

### 3.2 🗂️ 전체 디렉토리 구조

```
/{project_root}
├── CLAUDE.md                    # 🏛️ [현재 파일] 루트 허브 (헌법)
│
├── /src
│   ├── /app                     # 🚀 Next.js App Router
│   │   ├── claude.md            # → App Router 가이드
│   │   ├── globals.css          #    디자인 토큰 정의
│   │   ├── /(app)               #    인증된 사용자 라우트
│   │   ├── /admin               #    관리자 라우트
│   │   ├── /api                 #    API Routes
│   │   └── /login               #    로그인 페이지
│   │
│   ├── /components              # 🎨 UI 컴포넌트
│   │   ├── claude.md            # → 컴포넌트 가이드
│   │   ├── /ui                  #    shadcn/ui 기반 컴포넌트
│   │   ├── /layout              #    레이아웃 컴포넌트
│   │   ├── /common              #    공통 컴포넌트
│   │   └── /{domain}            #    도메인별 컴포넌트
│   │
│   ├── /features                # 📦 기능 모듈
│   │   └── claude.md            # → 기능 개발 가이드
│   │
│   ├── /lib                     # ⚙️ 유틸리티 & 백엔드
│   │   ├── claude.md            # → 백엔드/권한 가이드
│   │   ├── auth.ts              #    NextAuth 설정
│   │   ├── prisma.ts            #    Prisma 클라이언트
│   │   ├── rebac.ts             #    ReBAC 권한 시스템
│   │   ├── matching.ts          #    RAG 매칭 엔진
│   │   ├── /crawler             #    크롤러 시스템
│   │   ├── /documents           #    문서 분석 시스템
│   │   └── /supabase            #    Supabase 연동
│   │
│   ├── /hooks                   # 🪝 커스텀 훅
│   │   └── claude.md            # → 훅 개발 가이드
│   │
│   └── /types                   # 📝 타입 정의
│       └── claude.md            # → 타입 체계 가이드
│
├── /prisma                      # 🗄️ 데이터베이스
│   ├── claude.md                # → DB 스키마 가이드
│   ├── schema.prisma            #    Prisma 스키마
│   └── /migrations              #    마이그레이션
│
├── /scripts                     # 🔧 유틸리티 스크립트
│   └── cleanup-db.ts            #    DB 정리
│
└── /docs                        # 📚 상세 스펙 (자동 생성 금지)
    ├── /architecture            #    시스템 개요
    ├── /foundations             #    전역 규칙
    ├── /checklists              #    품질 검증
    └── /changes                 #    변경 이력
```

### 3.3 🔄 가이드 계층 구조

```
CLAUDE.md (루트 허브 - 헌법)
    │
    ├─→ /src/app/claude.md
    │       └─→ API Routes, Pages, Layouts
    │
    ├─→ /src/components/claude.md
    │       └─→ UI 컴포넌트, shadcn/ui, 접근성
    │
    ├─→ /src/features/claude.md
    │       └─→ Feature 모듈, 상태관리
    │
    ├─→ /src/lib/claude.md
    │       └─→ Auth, ReBAC, Prisma, i18n
    │
    ├─→ /src/hooks/claude.md
    │       └─→ 커스텀 훅 패턴
    │
    ├─→ /src/types/claude.md
    │       └─→ 타입 정의 체계
    │
    └─→ /prisma/claude.md
            └─→ DB 스키마, 마이그레이션
```

### 3.4 📋 작업별 가이드 선택

| 작업 유형 | 1차 참조 | 2차 참조 |
|----------|---------|---------|
| 새 페이지 생성 | `/src/app/claude.md` | `/src/components/claude.md` |
| API 엔드포인트 | `/src/app/claude.md` | `/src/lib/claude.md` |
| UI 컴포넌트 | `/src/components/claude.md` | 루트 `CLAUDE.md` (버튼 규칙) |
| 새 기능 모듈 | `/src/features/claude.md` | `/src/lib/claude.md` |
| DB 스키마 변경 | `/prisma/claude.md` | `/src/lib/claude.md` |
| 권한 시스템 | `/src/lib/claude.md` | `/prisma/claude.md` |
| 문서 분석/AI | `/src/lib/claude.md` | `/src/app/claude.md` (API) |

---

## 4. 전역 원칙 (변경 금지)

### 4.1 디자인 토큰 원칙

> **"Primary Color만 바꾸면 전역 반영"**

```css
:root {
  --primary: hsl(168 64% 50%);   /* 이 값만 변경 → 브랜드 변경 */
  --primary-foreground: hsl(0 0% 100%);
}
```

- 모든 색상은 CSS Variables 참조
- 하드코딩된 색상 사용 **금지** (`bg-[#xxx]` 금지)

### 4.2 버튼 전역 제한

#### 주사용 버튼 (Primary Use)

| Variant | 용도 | 사용 빈도 |
|---------|------|----------|
| `default` | Primary 액션 (CTA) | ⭐⭐⭐ 매우 높음 |
| `outline` | Secondary 액션 | ⭐⭐⭐ 매우 높음 |

#### 예비 버튼 (Reserved - 사용자 승인 시)

| Variant | 용도 | 사용 조건 |
|---------|------|----------|
| `destructive` | 위험 액션 (삭제 등) | 사용자 명시적 요청 시 |
| `ghost` | 최소 강조 | 사용자 명시적 요청 시 |

**규칙**:
- 기본적으로 `default` + `outline`만 사용
- 예비 버튼은 사용자가 요청할 때만 적용
- **버튼 variant 추가 확장 금지**
- 하위 claude.md에서 재정의 금지

### 4.3 네이밍 체계

```
{TYPE}.{DOMAIN}.{CONTEXT}.{NUMBER}
```

| TYPE | 용도 | 예시 |
|------|------|------|
| SID | Screen ID | `SID.AUTH.LOGIN.001` |
| LID | Label ID | `LID.MODAL.DELETE.001` |
| BTN | Button ID | `BTN.PRIMARY.SUBMIT.001` |

### 4.4 i18n 원칙

- 모든 UI 텍스트는 `text-config.ts`에서 관리
- 컴포넌트 내 하드코딩 한글 **금지**
- 톤 코드 체계:

| 톤 코드 | 용도 | 예시 |
|--------|------|------|
| `Confirm` | 긍정적 확인 | "저장되었습니다" |
| `Destructive` | 파괴적/위험 | "삭제하시겠습니까?" |
| `Soft` | 부드러운 안내 | "입력해 주세요" |
| `Neutral` | 중립적 정보 | "총 3개" |

### 4.5 접근성 (a11y)

**기본 원칙**:
- **WCAG 2.1 Level AA** 준수
- 색상 대비 4.5:1 이상
- 모든 인터랙션 키보드 접근 가능

**모달 접근성 필수 요구사항**:
- 열릴 때 포커스가 모달 내부로 이동
- Tab 키로 모달 내부만 순환 (포커스 트랩)
- ESC 키로 닫기
- 닫힐 때 트리거 요소로 포커스 복귀
- `role="dialog"` + `aria-modal="true"`
- `aria-labelledby` 또는 `aria-label` 필수

**상태도 작성 규칙 (State Machine)**:
```yaml
States: [CLOSED, OPENING, OPEN, CLOSING]

Events:
  - OPEN_MODAL: 모달 열기 요청
  - CLOSE_MODAL: 모달 닫기 요청
  - ANIMATION_END: 애니메이션 완료

Guards:
  - canOpen: "currentState === CLOSED"
  - canClose: "currentState === OPEN"

Transitions:
  CLOSED → OPENING: Event=OPEN_MODAL, Guard=canOpen
  OPENING → OPEN: Event=ANIMATION_END
  OPEN → CLOSING: Event=CLOSE_MODAL, Guard=canClose
  CLOSING → CLOSED: Event=ANIMATION_END
```

---

## 5. 개발 프로세스

### 5.1 워터폴 (스펙 선행)

```
요구사항 → 네이밍 → 토큰 → 컴포넌트 → 상태도 → i18n → 구현
```

### 5.2 애자일 (Feature Batch)

- 배치 크기: 연관 화면 1~3개 / 2~5일 분량
- 완료 기준: 동작 확인 + 에러 없음

---

## 6. 응답 & 문서 정책 (토큰 효율)

### 6.1 핵심 원칙 (필수 준수)

```
1. 문서(/docs)는 자동으로 생성/갱신하지 않는다
2. 문서 업데이트는 사용자가 명시적으로 요청할 때만 수행
3. 문서 업데이트 시 전체 재작성 금지, 변경된 범위의 델타만 기록
4. 구현은 Feature Batch 단위, 문서 정리는 배치 종료 후 선택
5. 응답 형식: (1) 설계 요약 → (2) 코드, 상세 문서는 요청 시만
```

### 6.2 응답 기본 형식

```
✅ 기본 응답: 설계 요약 (3~5줄) + 코드
❌ 금지: 매 요청마다 문서 자동 생성
```

### 6.3 문서 업데이트 트리거

| 상황 | 문서 작업 |
|-----|----------|
| 코딩 요청 | ❌ 문서 작성 안함 |
| Feature Batch 완료 | ⏳ 사용자 요청 시만 |
| "문서 업데이트해줘" | ✅ 델타만 업데이트 |
| 새 컴포넌트 추가 | 📋 체크리스트로 검증 |

### 6.4 델타 업데이트 규칙

- 전체 문서 재작성 **금지**
- 추가/수정된 섹션만 업데이트
- `/docs/changes/changelog.md`에 변경 기록

---

## 7. 충돌 해결

### 7.1 우선순위

1. **루트 claude.md** (이 파일)
2. 하위 claude.md
3. /docs 상세 스펙

### 7.2 하위 claude.md 제약

- 전역 원칙 재정의 **금지**
- 버튼 variant 확장 **금지**
- 토큰 네이밍 변경 **금지**

---

## 8. 작업 마무리 프로세스

### 8.1 필수 단계

1. **최종 피드백 확인**: 마무리 전 사용자에게 피드백 요청
2. **문서 업데이트**: 변경사항 발생 시 관련 `claude.md` 파일 먼저 수정
3. **품질 검증**: 타입체크 + 빌드테스트 실행
4. **버전 관리**: 커밋 + 푸쉬

### 8.2 검증 명령어

```bash
npx tsc --noEmit      # 타입체크
npm run build         # 빌드테스트
git add . && git commit -m "feat: 작업 내용" && git push
```

### 8.3 프로세스 순서

```
작업 완료 → 피드백 요청 → claude.md 업데이트 → 타입체크 → 빌드 → 커밋 → 푸쉬
```

---

## 9. 핵심 시스템 아키텍처

### 9.1 기업 문서 관리 시스템

```
CompanyDocument (메타데이터)
├── CompanyDocumentAnalysis (Gemini 2.5 Pro Vision 분석 결과)
└── CompanyDocumentEmbedding (OpenAI text-embedding-3-small 벡터)
```

**지원 문서 유형** (10가지):
- business_registration, corporation_registry, sme_certificate
- financial_statement, employment_insurance, export_performance
- certification, company_introduction, business_plan, patent

**문서 분석 플로우**:
1. 업로드 → Supabase Storage 저장
2. Gemini Vision으로 멀티모달 분석 (PDF/이미지)
3. OpenAI embeddings로 벡터화 (pgvector HNSW 인덱스)

### 9.2 비동기 임베딩 시스템

```
SupportProject.needsEmbedding = true (크롤러 설정)
    ↓
Vercel Cron (매일 01:00 KST) → Railway Worker 트리거
    ↓
Railway Worker: 배치 처리 (50개씩, 무제한 실행시간)
    ↓
document_embeddings 테이블에 저장
```

### 9.3 RAG 매칭 시스템

- `document_embeddings`: 통합 임베딩 테이블 (프로젝트 + 문서)
- 벡터 유사도 검색 → 매칭 점수 산출
- N번 API 호출 → 1번 배치 호출 최적화 (99% 성능 향상)

### 9.4 크롤러 시스템

```
CrawlSource → CrawlJob → SupportProject + ProjectAttachment
```

**한글 파일명 복원**:
- CP949 → UTF-8 → Latin-1 → UTF-8 복원 전략
- EUC-KR variant 복원 지원
- mojibake 패턴 감지 (혚혞혱, 챘챙챗 등)

---

## 10. 개발 규칙 (필수 준수)

### 10.1 Prisma 우선 원칙 🔴

> **수동 SQL 생성 금지 - 벡터 연산만 예외**

| 허용 | 금지 |
|-----|------|
| `prisma.$queryRaw` (벡터 검색 `<=>`, `<->`) | 테이블/컬럼 생성 SQL |
| `prisma.$executeRaw` (벡터 INSERT) | 인덱스 생성 SQL |
| Prisma 스키마 + `db push` | 마이그레이션 SQL |

**규칙**:
- 모든 테이블/모델은 `prisma/schema.prisma`에 정의
- 기존 테이블도 Prisma 스키마로 관리 (`@@map()` 사용)
- pgvector 벡터 컬럼: `Unsupported("vector(1536)")` 사용
- 벡터 인덱스(HNSW, GIN): Prisma `@@index` 사용

```prisma
// ✅ 올바른 예시 - Prisma 스키마
model DocumentEmbedding {
  embedding Unsupported("vector(1536)")
  @@index([embedding], map: "idx_embeddings_hnsw")
  @@map("document_embeddings")
}
```

```sql
-- ❌ 금지 - 수동 SQL
CREATE TABLE document_embeddings (...);
CREATE INDEX idx_embeddings_hnsw ...;
```

### 10.2 환경 변수 규칙 🔴

> **Prisma 명령 실행 전 `.env.local` 로드 필수**

```bash
# ✅ 올바른 방법
set -a && source .env.local && set +a && npx prisma db push

# ❌ 금지 - 환경 변수 없이 실행
npx prisma db push  # DIRECT_URL not found 에러 발생
```

**필수 환경 변수**:
- `DATABASE_URL`: Prisma 연결 (pgbouncer)
- `DIRECT_URL`: 마이그레이션용 직접 연결

**위치**: `.env.local` (gitignore됨)

### 10.3 권한 시스템 규칙 🔴

> **ReBAC 사용 - RLS(Row Level Security) 금지**

| 사용 | 금지 |
|-----|------|
| ReBAC (`/src/lib/rebac.ts`) | Supabase RLS 정책 |
| `RelationTuple` 모델 | `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` |
| `check()`, `grant()`, `revoke()` | `CREATE POLICY ...` |

**ReBAC 패턴**:
```typescript
// ✅ 올바른 권한 체크
import { check, grant } from "@/lib/rebac"

const canEdit = await check(userId, "company", companyId, "editor")
await grant("company", companyId, "owner", "user", userId)
```

```sql
-- ❌ 금지 - RLS 정책
CREATE POLICY "users can view own data" ON companies
  FOR SELECT USING (auth.uid() = user_id);
```

**이유**: Supabase RLS는 Prisma ORM과 충돌, ReBAC는 애플리케이션 레벨에서 유연한 권한 관리 제공

---

## 변경 이력

| 날짜 | 버전 | 변경 |
|-----|------|------|
| 2025-12-05 | 1.0.0 | 초기 생성 |
| 2025-12-05 | 1.1.0 | 계층 분리 (헌법만 유지) |
| 2025-12-05 | 2.0.0 | PRD 참조 방식 + 버튼 규칙 명확화 + 접근성 강화 |
| 2025-12-05 | 2.1.0 | 토큰 효율 원칙 추가 + /docs 구조 개선 |
| 2025-12-15 | 2.2.0 | 기업 문서 관리, 비동기 임베딩, RAG 매칭, 크롤러 시스템 추가 |
| 2025-12-15 | 3.0.0 | Hub-Spoke 아키텍처 강화, 7개 하위 claude.md 가이드 통합 |
| 2025-12-16 | 3.1.0 | 개발 규칙 섹션 추가 (Prisma 우선, 환경변수, ReBAC) |

---
> Source: [flowcoder2025/Konarae_FlowCoder](https://github.com/flowcoder2025/Konarae_FlowCoder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
