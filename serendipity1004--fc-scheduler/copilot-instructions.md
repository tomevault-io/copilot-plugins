## fc-scheduler

> 스케줄러 애플리케이션의 종합적인 프로젝트 개요 및 개발 가이드라인


# 스케줄러 프로젝트 개요

## 프로젝트 설명
이 프로젝트는 **스케줄러** 애플리케이션으로, Next.js 15로 구축되어 일정 관리 및 조직화를 위한 애플리케이션입니다. 이 프로젝트는 Vibe Coding Fast Campus 커리큘럼의 일부이며, 현대적인 웹 개발 실습을 위한 학습 프로젝트입니다.

## 필수 기억
- Shadcn MCP 항상 사용해서 컴포넌트 및 UI 구현.

## 기술 스택

### 핵심 프레임워크
- **Next.js 15.5.2** with App Router (React 19.1.0)
- **TypeScript 5** 타입 안전성을 위해
- **Turbopack** 빠른 개발 빌드를 위해

### 스타일링 & UI
- **Tailwind CSS 4** 커스텀 테마 설정과 함께
- **shadcn/ui** 컴포넌트 ("new-york" 스타일 변형)
- **Lucide React** 아이콘용
- **tw-animate-css** 애니메이션용
- **Geist** 폰트 패밀리 (Sans & Mono) Vercel에서 제공
- **date-fns** 날짜 처리 및 포맷팅용

### 개발 도구
- **ESLint 9** Next.js 설정과 함께
- **PostCSS** CSS 처리용
- **class-variance-authority** 컴포넌트 변형용
- **clsx** 및 **tailwind-merge** 조건부 스타일링용

## 프로젝트 구조

### 주요 파일들
- [package.json](mdc:package.json) - 프로젝트 의존성 및 스크립트
- [components.json](mdc:components.json) - shadcn/ui 설정
- [src/app/layout.tsx](mdc:src/app/layout.tsx) - 폰트 설정이 포함된 루트 레이아웃
- [src/app/page.tsx](mdc:src/app/page.tsx) - 메인 애플리케이션 페이지
- [src/app/globals.css](mdc:src/app/globals.css) - Tailwind 및 커스텀 CSS 변수가 포함된 글로벌 스타일
- [src/lib/utils.ts](mdc:src/lib/utils.ts) - 유틸리티 함수들 (클래스 병합을 위한 cn 헬퍼)

### 디렉토리 구조
```
src/
├── app/                 # Next.js App Router
│   ├── layout.tsx      # 루트 레이아웃
│   ├── page.tsx        # 메인 스케줄러 페이지
│   └── globals.css     # 글로벌 스타일
├── components/         # React 컴포넌트들
│   ├── ui/            # shadcn/ui 컴포넌트들
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── dialog.tsx
│   │   ├── calendar.tsx
│   │   ├── input.tsx
│   │   ├── textarea.tsx
│   │   ├── select.tsx
│   │   └── badge.tsx
│   ├── calendar/      # 달력 관련 컴포넌트들
│   │   ├── calendar-header.tsx    # 헤더 (월 네비게이션)
│   │   └── calendar-grid.tsx      # 메인 달력 그리드
│   └── events/        # 일정 관련 컴포넌트들
│       ├── event-dialog.tsx       # 일정 추가/수정 다이얼로그
│       └── event-list.tsx         # 일정 목록
└── lib/                # 유틸리티 라이브러리
    └── utils.ts        # 공통 유틸리티 (cn 함수)
```

## 개발 가이드라인

### 컴포넌트 명명 규칙
- **모든 컴포넌트 파일은 kebab-case를 사용해야 합니다**
- 예시: `BlogHeader.tsx` 대신 `blog-header.tsx`
- 이는 프로젝트의 모든 React 컴포넌트에 적용됩니다

### 코드 재사용성
- **코드를 리팩토링할 때 기존 컴포넌트 재사용을 선호합니다**
- 기존 UI 컴포넌트를 활용하여 컴포넌트 중복을 피합니다
- 커스텀 컴포넌트의 기반으로 shadcn/ui 컴포넌트를 사용합니다

### 스타일링 접근법
- 스타일링을 위해 Tailwind CSS 클래스를 사용합니다
- 조건부 클래스를 위해 [src/lib/utils.ts](mdc:src/lib/utils.ts)의 `cn()` 유틸리티 함수를 활용합니다
- [src/app/globals.css](mdc:src/app/globals.css)에 정의된 확립된 색상 시스템을 따릅니다
- CSS 커스텀 속성을 사용하여 라이트 및 다크 테마를 지원합니다

### shadcn/ui 통합
- 컴포넌트는 "new-york" 스타일 변형을 사용하도록 설정되어 있습니다
- 기본 색상은 "neutral"로 설정되어 있습니다
- 테마를 위해 CSS 변수가 활성화되어 있습니다
- 컴포넌트는 `@/components/ui/` 디렉토리에 배치해야 합니다
- 확립된 별칭을 사용합니다: `@/components`, `@/lib`, `@/hooks`

## 사용 가능한 스크립트
- `npm run dev` - Turbopack으로 개발 서버 시작
- `npm run build` - Turbopack으로 프로덕션 빌드
- `npm run start` - 프로덕션 서버 시작
- `npm run lint` - ESLint 실행

## 현재 상태
프로젝트는 **완전한 일정관리 웹사이트**로 구현되었으며 다음 기능들이 포함되어 있습니다:

### ✅ 구현된 기능들
- **달력 헤더**: 월/년 표시, 이전달/다음달 네비게이션, "오늘" 버튼, "새 일정" 추가 버튼
- **달력 그리드**: 전체 화면을 차지하는 달력 레이아웃, 각 날짜에 일정 표시 (색상 점), 날짜 클릭 시 일정 추가, 오늘 날짜 하이라이트
- **일정 다이얼로그**: 일정 추가/수정 모달, 제목/날짜/색상/설명 입력, 8가지 색상 선택 가능
- **일정 목록**: 선택된 날짜의 일정 표시, 일정 편집/삭제 기능, 반응형 레이아웃
- **메인 페이지 통합**: 모든 컴포넌트 연결, 상태 관리 (일정 CRUD), 반응형 그리드 레이아웃

### 🎨 UI/UX 특징
- **반응형 디자인**: 데스크톱에서는 달력(2/3) + 일정목록(1/3), 모바일에서는 세로 스택
- **직관적인 인터랙션**: 날짜 클릭으로 일정 추가, 색상으로 일정 구분
- **현대적인 디자인**: shadcn/ui 컴포넌트로 일관된 디자인
- **접근성**: 키보드 네비게이션, 스크린 리더 지원
- **컨테이너 기반 레이아웃**: 헤더와 메인 콘텐츠가 동일한 컨테이너 너비 사용

### 📦 설치된 shadcn/ui 컴포넌트들
- button, card, dialog, calendar, input, textarea, select, badge

## 개발 노트
- 프로젝트는 최신 React 19 기능을 사용합니다
- 더 빠른 개발 빌드를 위해 Turbopack이 활성화되어 있습니다
- 스타일링 시스템은 라이트 및 다크 모드를 지원합니다
- 모든 컴포넌트는 접근성을 염두에 두고 구축되어야 합니다
- 프로젝트는 App Router와 함께 최신 Next.js 15 모범 사례를 따릅니다

## 🚀 사용 방법
1. **일정 추가**: "새 일정" 버튼 클릭 또는 날짜 클릭 → 제목, 날짜, 색상, 설명 입력 후 저장
2. **일정 관리**: 달력에서 색상 점으로 일정 확인 → 오른쪽 일정 목록에서 편집/삭제
3. **네비게이션**: 화살표 버튼으로 월 이동, "오늘" 버튼으로 현재 날짜로 이동

## 개발을 위한 다음 단계
1. ✅ ~~메인 스케줄러 인터페이스 컴포넌트 생성~~ (완료)
2. ✅ ~~캘린더/스케줄 관리 기능 구현~~ (완료)
3. **데이터 지속성 추가** (로컬 스토리지 또는 데이터베이스)
4. **필요시 사용자 인증 구현**
5. ✅ ~~반응형 디자인 최적화 추가~~ (완료)
6. ✅ ~~접근성 기능 구현~~ (완료)
7. **고급 기능 추가**: 드래그앤드롭으로 일정 이동, 반복 일정, 알림 기능
8. **성능 최적화**: 가상화를 통한 대용량 일정 처리
9. **테마 확장**: 다크모드 토글, 커스텀 색상 테마

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/serendipity1004)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/serendipity1004)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
