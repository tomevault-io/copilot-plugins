## resource-planning-system

> **KARS** (Kangsters Auto Resource-management System) - 보조기기 업체용 통합 자원 관리 시스템

# KARS - Claude Code 설정

## 프로젝트 개요
**KARS** (Kangsters Auto Resource-management System) - 보조기기 업체용 통합 자원 관리 시스템

### 기술 스택
- **프론트**: React 18.2 + Next.js 15.1.3 + TypeScript + Tailwind CSS
- **상태 관리**: Zustand + React Query (TanStack Query)
- **UI**: Ant Design + Framer Motion

---

## 코딩 규칙

### 스타일
- React 함수형 컴포넌트 + TypeScript strict 모드
- Tailwind CSS 클래스 우선 사용

### 명명 규칙
| 대상       | 규칙        | 예시                                |
| ---------- | ----------- | ----------------------------------- |
| 컴포넌트   | PascalCase  | `OrderRequestForm.tsx`              |
| 훅         | camelCase   | `useOrder.ts` (use 접두사 필수)     |
| 타입 파일  | kebab-case  | `order-record.ts`                   |

### 대화 규칙
1. 친근한 반말 어투
2. 불확실한 내용은 명확히 표시
3. 단계적 사고 과정으로 답변
4. 한국어로 응답

---

## 작업 규칙

**워크플로우:**
1. **요청 분석**: 요구사항을 파악하고 유형 판단 (Feature / Bug / Improvement / Epic)
2. **스펙 확인**: `specs/`에 관련 스펙이 있으면 참조, 없으면 `/new-spec`으로 생성 검토
3. **코드베이스 탐색**: 관련 파일, 영향 범위, 기존 패턴 분석
4. **브랜치 생성**: `feature/설명` 또는 `fix/설명`
5. **구현**: 승인된 내용에 따라 코드 작성
6. **커밋**: 변경사항을 의미 있는 단위로 커밋 (스펙 있으면 `feat({스펙파일명}): {설명}`)

**스펙 관리 (`/specs/`):**
- `/new-spec {설명}`으로 요구사항을 구조화된 스펙 문서로 생성
- 수락 기준 3개+ → 스펙 생성 권장. 단순 버그 수정 → 스펙 불필요
- 체크박스 진행률 = 상태 (별도 status 필드 없음)
- 완료된 스펙은 `specs/done/`으로 이동
- 템플릿: `specs/_template.md`

**대규모 변경 분할 규칙 (필수):**
> v2.8.1에서 26개 파일 동시 변경 후 빌드/타입체크/서브에이전트가 타임아웃·무한 지연된 사고에서 도출.

- 구현은 **기능별 3~5파일 단위**로 분할 → 커밋 → 빌드 확인 → 다음 기능
- 10개+ 파일을 한 번에 수정하지 않는다
- 서브에이전트에는 **해당 기능 관련 파일만** 전달 (전체 diff 금지)
- 이미 대규모 변경이 쌓였으면: `git diff --stat` → 기능별 그룹핑 → 그룹별 순차 처리

**티켓 관리:**
- GitHub Issues 사용 금지 — `/BACKLOG.md`에서 로컬 관리
- 완료 시 체크박스 `[x]` 표시

**스펙 vs 백로그 구분:**
- `BACKLOG.md` — 할 일 목록 (1줄 단위, 우선순위 추적)
- `specs/` — 요구사항 문서 (배경, 수락 기준, 설계 메모 포함)

---

## 핵심 개발 규칙

### API & 데이터
- **React Query**: `['resource', params]` 키 구조
- **통신**: Axios
- **인증**: JWT + Zustand 상태 관리
- **캐시 무효화**: 데이터 변경 후 `queryClient.invalidateQueries`로 자동 UI 업데이트 (페이지 새로고침 금지)
- **에러 처리**: `/docs/api-error-handling.md` 참고

### 날짜 처리
| 규칙              | 설명                                     |
| ----------------- | ---------------------------------------- |
| 빈 값             | `undefined` 반환 (빈 문자열 X)           |
| 서버 전송         | `undefined` 필드 제거                    |
| 형식              | `YYYY-MM-DDTHH:MM:SS+09:00` (KST)        |

### 테마 색상
| 용도         | 색상                              |
| ------------ | --------------------------------- |
| 일반 기능    | 파란색 (`blue-500`, `blue-600`)   |
| 휠체어 발주  | 보라색 (`purple-500`, `purple-600`) |
| 경고/에러    | 빨간색 (`red-500`, `red-600`)     |

### 모달 & z-index 관리
| 레이어       | z-index | 용도                              |
| ------------ | ------- | --------------------------------- |
| 기본 모달    | `z-50`  | 1차 모달                          |
| 중첩 모달    | `z-[60]` | 2차 모달                          |
| 최상위 모달  | `z-[70]` | 알림, 긴급 다이얼로그             |

모달 위에 다른 모달이 열릴 경우 z-index를 10 단위로 증가. 모달 닫기 시 하위 모달들도 함께 닫기.

---

## 주요 명령어

```bash
npm run dev          # 개발 서버 실행
npm run build        # 프로덕션 빌드 (Claude 안에서 실행 금지! 별도 터미널에서만)
tsc --noEmit         # 타입 체크만 (Claude 안에서 빌드 대체, 10~30초)
npm run type-check   # TypeScript 타입 체크
npm run lint         # ESLint 검사
```

---

## 업데이트 관리

`/update-changelog` 슬래시 커맨드로 자동 처리 (버전 bump + CHANGELOG + /update 페이지 생성).

변경 시 반드시 업데이트: `CHANGELOG.md`, `src/constants/version.ts`, 관련 `/docs` 문서.

---

## 상세 규칙 (`.claude/rules/`)

| 파일 | 내용 |
|------|------|
| `agent-io-format.md` | 응답 형식 규칙 |
| `subagent-workflow.md` | 서브에이전트 오케스트레이션, 빌드 규칙, 실패 제한 |
| `permissions.md` | 권한 시스템, 창고 접근 권한 패턴 |
| `pricing.md` | 가격 입력 (총액→자동계산), 영세율 |
| `form-patterns.md` | 3-레이어 검증, 자동 채우기 패턴 |
| `docs-index.md` | 기능별 참조 문서 인덱스 |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/S-Soo100)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/S-Soo100)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
