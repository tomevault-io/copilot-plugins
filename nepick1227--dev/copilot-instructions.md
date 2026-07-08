## dev

> > Codex가 이 프로젝트에서 코드를 생성할 때 자동으로 참조하는 규칙 파일입니다.

# NePick 프로젝트 — Codex 코드 규칙

> Codex가 이 프로젝트에서 코드를 생성할 때 자동으로 참조하는 규칙 파일입니다.

## 프로젝트 개요

- **서비스**: 맛집·카페 방문 기록 & 추천 앱 (모바일 웹)
- **스택**: Next.js 16 (App Router) · TypeScript · Tailwind CSS v4 · Supabase · 카카오맵 SDK
- **배포**: Vercel (nepick.kr)
- **경로 별칭**: `@/*` → 프로젝트 루트 (`./`)

## 디렉토리 구조

```
nepick/
├── app/                    # Next.js App Router (페이지 라우팅만)
│   ├── layout.tsx
│   ├── page.tsx            # 랜딩 (→ /auth/login 리다이렉트)
│   ├── error.tsx           # Next.js 전역 에러 바운더리
│   ├── api/
│   │   ├── auth/naver/route.ts           # 네이버 OAuth 시작
│   │   ├── auth/naver/callback/route.ts  # 네이버 OAuth 콜백
│   │   └── kakao-search/route.ts         # 카카오 장소 검색 프록시
│   ├── auth/
│   │   ├── login/page.tsx
│   │   ├── callback/route.ts
│   │   ├── terms/page.tsx
│   │   ├── signup/page.tsx
│   │   ├── error/page.tsx       # 인증 오류 안내
│   │   └── naver/verify/page.tsx
│   ├── home/page.tsx
│   ├── record/
│   │   ├── page.tsx
│   │   └── [id]/edit/page.tsx
│   ├── mypick/page.tsx
│   └── profile/
│       ├── page.tsx
│       ├── edit/page.tsx
│       ├── permissions/page.tsx
│       ├── withdrawal/page.tsx
│       └── withdrawal/done/page.tsx
├── components/             # 공통 UI 컴포넌트
│   ├── ui/                 # Button, Input, Modal, Toast, Spinner, Badge, Chip, Textarea
│   ├── layout/             # Header, BottomNav, PageContainer
│   └── map/                # KakaoMap, MapMarker, BottomSheet
├── features/               # 페이지별 비즈니스 로직 + 전용 컴포넌트
│   ├── auth/               # LoginForm, SignupForm, TermsAgreement
│   ├── home/               # MapView, RankingSheet, StoreCard
│   ├── record/             # RecordForm, StoreSearch, ImageUpload
│   ├── mypick/             # Timeline, RecordCard, MonthFilter
│   └── profile/            # ProfileView, ProfileEditForm
├── hooks/                  # 커스텀 훅
│   ├── use-kakao-map.ts    # 카카오맵 인스턴스
│   └── use-toast.ts        # 토스트 메시지
├── lib/                    # 외부 서비스 클라이언트
│   ├── supabase/
│   │   ├── client.ts       # 브라우저용 (CSR)
│   │   └── server.ts       # 서버용 (SSR/RSC)
│   └── kakao/
│       └── map.ts          # SDK 로드 + 유틸
├── styles/
│   └── tokens.ts           # 디자인 토큰 (색상, 폰트, 레이아웃)
├── types/
│   ├── database.ts         # Supabase 테이블 타입
│   └── kakao.d.ts          # 카카오맵 SDK 전역 타입
└── utils/
    ├── format.ts           # 날짜·텍스트 포맷
    └── validation.ts       # 입력값 검증
```

## 네이밍 규칙

| 대상 | 규칙 | 예시 |
|------|------|------|
| 파일/폴더 | kebab-case | `store-card.tsx`, `use-auth.ts` |
| React 컴포넌트 | PascalCase | `StoreCard`, `BottomSheet` |
| 함수/변수 | camelCase | `handleSubmit`, `isLoading` |
| 상수 | UPPER_SNAKE_CASE | `MAX_COMMENT_LENGTH` |
| 타입/인터페이스 | PascalCase | `Store`, `RecordFormData` |
| DB 컬럼 | snake_case | `visited_at`, `pick_count` |

## 컴포넌트 작성 패턴

```tsx
// 1. import 순서: 외부 라이브러리 → 내부 모듈 → 타입
import { useState, useCallback } from "react";
import { useRouter } from "next/navigation";
import { colors } from "@/styles/tokens";
import type { Store } from "@/types/database";

// 2. Props 타입은 반드시 interface로 정의
interface StoreCardProps {
  store: Store;
  onClick?: (storeId: number) => void;
}

// 3. default export 함수형 컴포넌트
export default function StoreCard({ store, onClick }: StoreCardProps) {
  // hooks 최상단 선언
  const [isActive, setIsActive] = useState(false);

  // 핸들러는 useCallback
  const handleClick = useCallback(() => {
    onClick?.(store.id);
  }, [store.id, onClick]);

  // 조건부 early return
  if (!store) return null;

  return (
    <div className="flex items-center gap-3 p-4 border-b border-border">
      ...
    </div>
  );
}
```

**핵심 규칙:**
- 파일 당 200줄 이하 권장 → 초과 시 하위 컴포넌트로 분리
- 인라인 스타일(`style={{}}`) 사용 금지 → Tailwind 클래스 사용
- `any` 타입 사용 금지 → `unknown` + 타입 가드 사용
- SVG 아이콘은 `components/ui/icons/` 에 컴포넌트로 분리

## 디자인 토큰 (app/globals.css @theme inline)

색상은 `globals.css`의 `@theme inline {}` 블록에 CSS 변수로 정의됩니다. Tailwind 클래스(`text-primary`, `bg-surface` 등)로 바로 사용할 수 있습니다. 새 색상이 필요하면 `tailwind.config.ts`가 아닌 `globals.css @theme inline`에 추가합니다.

```
/* 브랜드 */
primary:         #D32F2F   주요 액션, 에러 상태
primary-dark:    #B71C1C   호버/강조
primary-soft:    #FFF1F1   선택된 항목 배경
primary-border:  #F3B4B4   선택된 항목 테두리

/* 텍스트 */
text-primary:    #111827   본문
text-secondary:  #6B7280   보조
text-tertiary:   #9CA3AF   비활성/플레이스홀더

/* UI */
border:          #E5E7EB   선/구분선
bg:              #FAFAFA   페이지 배경
surface:         #FFFFFF   카드/시트 배경

/* 성공 */
success:         #10B981   성공 아이콘
success-text:    #047857   성공 텍스트
success-border:  #34D399   성공 테두리
success-soft:    #ECFDF5   성공 배경

/* 비활성 */
disabled-bg:     #E5E7EB
disabled-text:   #9CA3AF
```

### 추천 배지 색상

추천 배지는 별도 색상 토큰 없이 아래 조합을 사용합니다 (`components/ui/Badge.tsx` 참조):

```
추천(👍):   bg-primary-soft text-primary
보통(😐):   bg-bg text-text-secondary
비추(👎):   bg-bg text-text-secondary
```

### CSS 유틸리티 클래스 (globals.css)

```
nepick-fade-in   진입 애니메이션 (240ms, translateY 8px→0 + opacity)
                 stagger는 [animation-delay:Xms] Tailwind 임의값으로 추가
safe-area-pb     padding-bottom: max(16px, env(safe-area-inset-bottom))
                 → 탭바·바텀시트 footer 등 iOS 홈 인디케이터 영역에 사용
safe-area-pb-lg  padding-bottom: calc(24px + env(safe-area-inset-bottom))
                 → 페이지 고정 CTA 버튼 영역에 사용
```

## Supabase 데이터 레이어 패턴

```ts
// features/*/api.ts — 데이터 함수 분리
// 에러는 throw, 컴포넌트에서 try-catch 처리

export async function fetchRecords(userId: string) {
  const supabase = createClient();        // @/lib/supabase/client
  const { data, error } = await supabase
    .from("records")
    .select("*, stores(*)")
    .eq("user_id", userId)
    .order("visited_at", { ascending: false });

  if (error) throw error;
  return data;
}
```

- `service_role` 키는 Edge Function에서만 사용합니다.
- 카카오 REST API 키는 Edge Function(`supabase/functions/kakao-search/`)을 통해서만 사용합니다.
- `NEXT_PUBLIC_` 접두사 변수만 프론트엔드에서 사용 가능합니다.

## 카카오맵 규칙

- JS Key는 SDK 로드에만 사용 (`@/lib/kakao/map.ts`의 `loadKakaoMapSDK()` 호출)
- 지도 인스턴스는 `useRef`로 관리, 언마운트 시 마커/오버레이 정리
- REST API Key는 절대 프론트 코드에 포함하지 않음

## 에러 처리

```ts
// features 레이어: throw
if (error) throw error;

// 컴포넌트 레이어: try-catch + Toast
try {
  await createRecord(data);
  showToast("저장되었습니다");
} catch {
  showToast("저장에 실패했습니다. 다시 시도해주세요");
} finally {
  setIsSubmitting(false);
}
```

## Git 커밋 메시지

```
feat: 카카오 로그인 연동
fix: 닉네임 중복 검사 오류 수정
style: 홈 지도 마커 색상 변경
refactor: useAuth 세션 관리 로직 분리
chore: Supabase SDK 버전 업데이트
```

## 보안 규칙

- `.env.local`은 절대 git commit 하지 않음
- `KAKAO_REST_API_KEY`는 Edge Function 환경변수로만 설정
- 이미지 업로드 시 MIME 타입 검사 + 5MB 이하 제한

## 보안 / QA 필수 가드레일

이 프로젝트를 이어받는 사람과 AI 모델은 작업 전 `docs/security-qa-handoff.md`를 먼저 확인합니다.

- OWASP Top 10 관점으로 접근권한, 인증, 입력 검증, 공급망, 로깅, 보안 설정을 매 변경마다 재점검합니다.
- 프론트 번들/F12 Network에 `SUPABASE_SERVICE_ROLE_KEY`, `NAVER_CLIENT_SECRET`, `KAKAO_REST_API_KEY`, `GEMINI_API_KEY`가 노출되면 안 됩니다.
- `NEXT_PUBLIC_` 값은 공개되어도 되는 값만 사용합니다. 비밀키를 `NEXT_PUBLIC_`로 만들지 않습니다.
- Supabase RLS 변경은 기존 적용 가능성이 있는 migration을 수정하지 말고 새 migration으로 추가합니다.
- exposed schema의 table/view는 RLS와 정책을 먼저 확인합니다. view는 가능하면 `security_invoker = true`로 만듭니다.
- `SECURITY DEFINER` 함수는 정말 필요한 경우에만 사용하고, 함수 내부에서 `auth.uid()` 검사를 수행한 뒤 `REVOKE ALL FROM PUBLIC`과 필요한 role `GRANT`를 명시합니다.
- 사용자의 개인 기록과 이미지 저장소는 개인 데이터입니다. 이미지는 기본 private bucket + signed URL로 다룹니다.
- 공용 가게 데이터는 브라우저에서 직접 `stores.insert/update`하지 않습니다. 카카오 REST API로 서버에서 검증한 뒤 저장합니다.
- 기록 데이터는 개인 `records`에 쌓이며, 한 사용자가 같은 가게에 여러 기록을 남길 수 있습니다.
- 탈퇴 계정의 닉네임은 재사용 가능해야 합니다. 활성 계정끼리만 닉네임 중복을 막습니다.
- 외부 API/Gemini/카카오/인증 route는 입력 크기 제한, 값 검증, 요청 제한을 고려합니다.
- 새 dependency를 추가하거나 버전을 바꾸면 `npm audit`, `npm run lint`, `npm run build`를 실행하고 결과를 PR/커밋 설명에 남깁니다.
- 보안 헤더/CSP를 바꾸면 카카오맵, Supabase 이미지 signed URL, 폰트, 로그인 흐름이 깨지지 않는지 브라우저 Console/Network로 확인합니다.
- 운영 DB 적용은 Supabase migration 적용 여부를 확인한 뒤 진행합니다. 적용 전에는 로컬 코드가 통과해도 운영 보안이 완성된 것으로 보지 않습니다.

## 참고 문서

- 상세 작업 리스트 및 전체 가이드: `NePick_작업리스트_코드규칙가이드.md`
- DB 스키마 / 정책정의서: `NePick_정책정의서_v0.2.2.xlsx`
- 보안/QA 인수인계 체크리스트: `docs/security-qa-handoff.md`

---
> Source: [nepick1227/dev](https://github.com/nepick1227/dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
