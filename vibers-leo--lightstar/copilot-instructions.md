## lightstar

> - **전략 진단 리포트**: `data/STRATEGY_ANALYSIS.md`

## 전략 문서 (개발 전 반드시 숙지)
- **전략 진단 리포트**: `data/STRATEGY_ANALYSIS.md`
- **PM 공통 지침**: 맥미니 루트 `pm.md`

### 전략 핵심 요약
- AI 기반 통합 분석이 경쟁사 대비 차별화 핵심 (사주+관상+MBTI+혈액형)
- 온보딩 → 세션 시간 15분 달성 → 재방문율 40% 달성이 3단계 성장 경로
- SNS 공유 기능이 바이럴 성장의 열쇠 (공유 시 10-20% 바이럴 계수)
- 프리미엄 구독(Pro 9,900원)으로 월 수익 최소 495만원 달성 가능
- Phase 2에서 마케팅(SEO/SNS) 집중 필요, 사용자 발견이 최대 과제

### 빌더 공통 지침
- **gstack 빌더 철학**: 맥미니 루트 `gstack.md` — Boil the Lake, Search Before Building, 스프린트 프로세스
- **개발 프로세스**: Think → Plan → Build → Review → Test → Ship → Reflect
- **핵심 규칙**: 테스트 동시 작성, 새 패턴 도입 전 검색, 압축률 기반 추정

---

# 바이브 철학관 (Vibe Philosophy Agent 3.0)

## 프로젝트 개요

사주, 관상, MBTI, 혈액형을 통합한 AI 철학 상담 서비스.
Gemini 2.0 Flash 기반 실시간 대화형 상담 시스템.

## 기술 스택

- React 19 + TypeScript + Vite 6
- Tailwind CSS v4 (npm, `@tailwindcss/vite` 플러그인)
- Google Generative AI (`@google/generative-ai`) - 서버 측만 사용
- Vercel Serverless Functions (API 키 보안)
- lucide-react (아이콘)
- react-markdown (메시지 렌더링)

## 프로젝트 구조

```
lightstar/
├── index.html                  # HTML 엔트리 (폰트만 로드)
├── vite.config.ts              # Vite 설정 (Tailwind 플러그인, /api 프록시)
├── tsconfig.json               # TypeScript 설정
├── vercel.json                 # Vercel 배포 설정
├── api/                        # Vercel Serverless Functions
│   ├── chat.ts                 # 채팅 API (Gemini 프록시, Rate Limiting)
│   └── analyze-face.ts         # 관상 분석 API (Gemini Vision)
├── src/
│   ├── index.tsx               # 엔트리 포인트
│   ├── app/App.tsx             # 메인 앱 (상태 관리, 세션 제어)
│   ├── types/index.ts          # 타입 정의
│   ├── constants/prompts.ts    # 시스템 프롬프트 (클라이언트용, 서버에도 복사됨)
│   ├── components/
│   │   ├── chat/ChatInterface.tsx     # 메시지 표시, 입력
│   │   ├── control/ControlPanel.tsx   # 프로필 입력, 모드 선택
│   │   ├── modals/SessionRestoreModal.tsx
│   │   ├── modals/OnboardingModal.tsx # (구현 예정)
│   │   ├── ui/Toast.tsx               # 토스트 알림 (Portal)
│   │   └── ui/MobileDrawer.tsx        # 모바일 드로어
│   ├── services/api.ts         # Vercel API 호출 클라이언트
│   ├── utils/
│   │   ├── toast.ts            # 토스트 전역 관리 (pub/sub)
│   │   ├── storage.ts          # LocalStorage 세션 저장/복원
│   │   └── fileValidation.ts   # 이미지 파일 검증/압축
│   └── styles/globals.css      # Tailwind v4 테마 + 커스텀 CSS
├── .env.local                  # GEMINI_API_KEY (gitignore됨, 로컬 개발용)
└── OKR.md                      # 프로젝트 OKR
```

## 개발 명령어

```bash
npm run dev      # 프론트엔드 개발 서버 (localhost:3300)
npx tsc --noEmit # 타입 체크
npm run build    # 프로덕션 빌드
npm run preview  # 빌드 결과 미리보기
```

**로컬에서 API 함수 테스트:**
```bash
npx vercel dev --listen 3001  # Vercel 개발 서버 (API 함수 제공)
npm run dev                    # Vite가 /api를 localhost:3001로 프록시
```

## 환경 변수

- `GEMINI_API_KEY`: Gemini API 키
  - **로컬**: `.env.local`에 설정
  - **프로덕션**: Vercel 대시보드 환경 변수로 설정
  - 클라이언트 번들에 절대 포함되지 않음 (서버 측만 접근)

## 아키텍처

### API 보안

클라이언트 → `/api/chat` (Vercel Function) → Gemini API
- API 키는 서버 환경 변수에서만 읽음
- IP 기반 Rate Limiting (채팅: 분당 20회, 관상분석: 분당 5회)
- 입력 검증 (메시지 2000자 제한, XSS 방어)
- `dist/` 번들에 API 키/Gemini SDK 코드 없음

---

## OKR 기반 개발 지침

**모든 작업은 OKR.md에 정의된 목표를 달성하는 방향으로 진행한다.**

### Objective

> "사용자가 자신의 내면을 깊이 이해하고, 삶의 방향성을 찾을 수 있는 가장 신뢰받는 AI 철학관 플랫폼이 된다"

### Key Results 달성 기준

| KR | 목표 | 작업 시 고려사항 |
|----|------|----------------|
| KR 1 | 세션 시간 15분+ | 사용자가 오래 머물고 싶은 UX를 만들 것 |
| KR 2 | 만족도 85%+ | AI 응답 품질과 에러 처리에 집중 |
| KR 3 | 재방문율 40%+ | 데이터 영속성, 세션 복원, 공유 기능 |
| KR 4 | 응답 3초, 에러 1%미만 | 성능 최적화, 에러 핸들링 강화 |

### 작업 시 반드시 지킬 것

1. **기능 추가/수정 전에 "이 작업이 어떤 KR에 기여하는가?"를 먼저 확인**
2. **Output(했다)보다 Outcome(결과가 나타났다)에 집중**
3. **OKR.md의 Initiative 체크리스트를 작업 진행/완료 시 업데이트할 것**

---

## 코드 컨벤션

### 도사 페르소나 유지

- 모든 사용자 대면 메시지는 **도사 페르소나**를 유지할 것
- `alert()` 절대 사용 금지 → `showToast()` 사용
- 기술적 에러 메시지를 사용자에게 직접 노출하지 말 것

### 에러 처리

- 서버 API 에러: `api/chat.ts`에서 에러 코드별 분기 (도사 페르소나 메시지)
- 클라이언트 에러: `services/api.ts`에서 HTTP 상태별 처리
- UI 에러: `utils/toast.ts`의 `showToast(type, message)` 사용

### 파일 업로드

- `utils/fileValidation.ts`의 `validateImageFile()` 필수 사용
- 제한: 5MB, JPEG/PNG/WebP만 허용
- MIME + 확장자 이중 검증

### 데이터 저장

- 세션 데이터: `utils/storage.ts` 사용 (LocalStorage, 24시간 유효)
- faceImage(Base64)는 저장하지 않음 (용량/보안)
- faceFeatures(텍스트)만 저장

### 스타일링

- Tailwind CSS v4 (`src/styles/globals.css`에서 `@theme` 설정)
- 커스텀 색상: `void-950/900/800`, `gold-100~600`, `mystic-100~900`
- glassmorphism: `.glass-panel`, `.glass-input` 클래스 활용
- 폰트: Pretendard(본문), Noto Serif KR(제목), Gowun Batang(AI 응답)

### 모바일 대응

- 데스크톱: ControlPanel은 사이드바로 표시
- 모바일: 세션 전에는 ControlPanel 직접 표시, 세션 후에는 MobileDrawer로 접근
- breakpoint: `md:` (768px)

---

## 진행 상황

### Phase 1: 핵심 안정화 - 완료
- Toast 시스템, 파일 검증, 세션 영속성, 모바일 드로어, 에러 처리

### Phase 2A: 인프라 기반 - 완료
- src/ 디렉토리 재편, Tailwind v4 npm 마이그레이션, importmap 제거
- Vercel Serverless Functions (API 키 보안), vercel.json 배포 설정
- 클라이언트 번들에서 API 키/Gemini SDK 완전 제거

### Phase 2B: 코드 리팩토링 - 진행 예정
- App.tsx 상태 분리 (커스텀 훅), ControlPanel 분리

### Phase 2C: 새 기능 구현 - 진행 예정
- 별자리 모드, 결과 카드, SNS 공유, 온보딩 모달

### Phase 3: 측정 및 고도화 - 향후
- GA4, Rate Limiting 고도화, 성능 최적화, SEO

**상세 로드맵은 OKR.md 참조**

---

## 개발 규칙

### 코드 스타일
- TypeScript strict mode 사용
- 한글 우선: 모든 UI 텍스트와 주석은 한국어
- 시맨틱 라인 브레이크: 긴 텍스트는 의미 단위로 줄바꿈

### Git 규칙
- 커밋 메시지: 한글 (feat:, fix:, refactor:, chore: 접두사)
- .env 파일 절대 커밋 금지
- `git add .` 사용 금지 → 특정 파일만 add

### 디자인 준수
- 디자인 가이드: `DESIGN_GUIDE.md` 참조
- 반응형: md (768px) 브레이크포인트 기준
- 접근성: lang="ko" 유지, word-break: keep-all

### AI Recipe 이미지 API

이 프로젝트는 **AI Recipe 중앙 이미지 서비스**를 사용합니다.
Vite 앱이므로 클라이언트가 아닌 `api/lib/` 서버 코드에서 사용합니다.

#### 사용 가능한 함수

```typescript
import { searchStockImage, generateAIImage } from './lib/ai-recipe-client';
```

#### Stock Image 검색
```typescript
const image = await searchStockImage('mystical fortune telling', {
  orientation: 'squarish',
  size: 'medium',
});
```

#### AI 이미지 생성
```typescript
const image = await generateAIImage('mystical tarot card design, korean traditional aesthetic, fortune telling', {
  size: 'medium',
  provider: 'auto',
});
```

#### 주요 용도
- 운세 결과 카드 이미지
- 사주/관상 시각 자료
- 공유용 결과 이미지

#### 주의사항
- `api/` 서버리스 함수에서만 사용 (API 키 보호)
- Rate Limit: 1000회/일
- AI Recipe 서버 실행 필요: http://localhost:3300

### 상위 브랜드
- 회사: 계발자들 (Vibers)
- 도메인: vibers.co.kr


## 세션로그 기록 (필수)
- 모든 개발 대화의 주요 내용을 `session-logs/` 폴더에 기록할 것
- 파일명: `YYYY-MM-DD_한글제목.md` / 내용: 한글
- 세션 종료 시, 마일스톤 달성 시, **컨텍스트 압축 전**에 반드시 저장
- 상세 포맷은 상위 CLAUDE.md 참조

---
> Source: [vibers-leo/lightstar](https://github.com/vibers-leo/lightstar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
