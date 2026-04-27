## aiscamcheck

> - 이름: AI Scout (aiscout)

# CLAUDE.md (aiscout 프로젝트)

## 프로젝트 정보
- 이름: AI Scout (aiscout)
- 스택: Next.js 16 App Router + TypeScript + Tailwind CSS v4 + MDX
- 배포: Vercel (git push 시 자동 배포)
- 자동화: GitHub Actions Cron (매일 09:00 KST)

## 디렉토리 구조
- `app/` — Next.js App Router 페이지
- `components/` — React 컴포넌트 (ComparisonTable, PricingCard, ToolRating, FaqSection 포함)
- `content/posts/` — 자동 생성 MDX 포스트
- `lib/` — posts.ts (MDX 파싱), affiliate.ts (제휴 링크)
- `scripts/` — 자동화 스크립트 (collect-and-generate.ts가 메인)
- `tests/` — Vitest 테스트

## 주요 명령어
- `npm run dev` — 개발 서버
- `npm run build` — 빌드
- `npm test` — 테스트 실행
- `npm run generate` — 포스트 수동 생성 (ANTHROPIC_API_KEY 필요)

## 코딩 규칙
- App Router 사용 (pages/ 금지)
- Server Component 기본, Client Component 최소화
- API 키는 서버 사이드에서만 사용
- PostType: NEW_TOOL_REVIEW | VS_COMPARISON | PRICING_GUIDE | UPDATE_SUMMARY | HOW_TO_GUIDE

## 🔄 Stuck Detection & Reset Protocol

When struggling with a feature (3+ failed attempts, growing complexity,
or cascading fixes), STOP and ask yourself:

> "Knowing what we know now, if we were to start reimplementing this
> feature from scratch, how would we do things differently, particularly
> with an eye for refactoring to reduce code complexity and fragmentation.
> What should we have done prior to even starting this feature?"

### Process:
1. **Detect** — 같은 에러 3회 반복, 파일 5개 이상 동시 수정, 또는 fix-on-fix 패턴 발생 시
2. **Stop** — 현재 접근 방식을 즉시 중단
3. **Reflect** — 위 질문에 답변을 작성 (root cause, 잘못된 가정, 누락된 설계)
4. **Plan** — 새로운 설계를 먼저 문서화 (코드 작성 전)
5. **Reset** — 변경사항 revert 후 새 설계 기반으로 처음부터 재구현

## ⚡ Token Optimization (from ECC)

### 기본 설정
- 일반 작업: sonnet 모델 사용 (비용 60% 절감)
- 복잡한 아키텍처/디버깅: opus로 전환
- MAX_THINKING_TOKENS: 10000 (기본 31999 대비 70% 절감)
- AUTOCOMPACT: 50% (기본 95% 대비 조기 컴팩션으로 품질 유지)

### Strategic Compact 규칙
- /compact 타이밍: 리서치 완료 후 → 구현 전, 마일스톤 완료 후 → 다음 작업 전, 디버깅 후 → 기능 작업 전, 실패한 접근 폐기 후 → 새 접근 전
- /compact 금지: 구현 중간 (변수명, 파일 경로, 부분 상태 유실됨)
- /clear: 무관한 작업 간 전환 시 (무료, 즉시 리셋)
- /cost: 세션 중 토큰 소비 모니터링

## 📝 Coding Style Rules (from ECC)

### 불변성 우선
- const 기본 사용, let은 반드시 필요할 때만
- 객체/배열 변경 대신 스프레드 연산자 사용
- 함수는 순수 함수(pure function) 우선

### 파일 구조
- 한 파일 = 한 책임 (300줄 초과 시 분리)
- 명확한 네이밍: 파일명은 내용을 설명 (utils.ts ❌ → formatDate.ts ✅)
- 관련 코드는 같은 디렉토리에 배치

### 주석 원칙
- WHY를 설명, WHAT은 설명하지 않음
- ✅ "API 과부하 방지를 위한 지수 백오프 적용"
- ❌ "카운터를 1 증가"

### 에러 핸들링
- 에러를 삼키지 않음 (빈 catch 블록 금지)
- 사용자에게 보여줄 에러와 로깅할 에러를 구분
- 에러 메시지에 컨텍스트 포함 (무엇이, 왜, 어떻게 해결)

## 🔀 Git Workflow Rules (from ECC)

### 커밋 메시지 형식
`
<type>(<scope>): <description>

type: feat, fix, docs, style, refactor, test, chore
scope: 선택적, 영향 범위
description: 현재형, 소문자 시작, 마침표 없음
`

### 예시
- eat(auth): add Google OAuth login
- ix(api): handle null response from external service
- 
efactor(blog): extract post generation into separate module

### PR 규칙
- 한 PR = 한 기능/수정
- 자체 코드 리뷰 후 머지
- 빌드 깨지는 코드 머지 금지

## 🔒 Security Rules (from ECC)

### 절대 금지
- 하드코딩된 시크릿/API 키/비밀번호 (.env 사용)
- .env 파일 커밋 (.gitignore 필수)
- 사용자 입력 무검증 사용

### 필수 체크
- 환경변수로 시크릿 관리
- API 키 노출 시 즉시 revoke & rotate
- 의존성 보안 취약점 확인 (
pm audit)

## 🧪 Testing Rules (from ECC)

### TDD 우선
1. 실패하는 테스트 작성 (RED)
2. 최소한의 코드로 통과 (GREEN)
3. 리팩토링 (REFACTOR)

### 테스트 원칙
- 새 기능 = 테스트 필수
- 버그 수정 = 재현 테스트 먼저
- 외부 의존성은 모킹
- 테스트는 독립적 (순서 무관하게 실행 가능)

## 🔄 Research-First Development (from ECC)

코딩 전에 항상 리서치:
1. **Search** — 기존 솔루션/라이브러리/패턴 검색
2. **Read** — 관련 문서/API 레퍼런스 읽기
3. **Plan** — 설계 먼저 (CLAUDE.md나 주석으로 계획 작성)
4. **Code** — 그 다음 코딩
5. **Verify** — 빌드, 테스트, 린트 확인

## 📚 LLM Knowledge Base System

### 구조
- kb/raw/ — 원본 자료 (사람이 추가)
- kb/wiki/ — LLM이 컴파일하는 위키 (LLM만 편집)
- kb/output/ — Q&A 결과물

### KB 컴파일 규칙
raw/에 새 자료가 추가되면:
1. 자료를 읽고 요약 작성
2. 관련 concept/tool 문서에 내용 반영
3. 백링크 추가 (관련 문서 연결)
4. INDEX.md 업데이트 (문서 목록 + 최근 업데이트)
5. 기존 문서와 불일치하는 내용이 있으면 표시

### KB 문서 작성 규칙
- 모든 위키 문서 상단에 메타데이터 포함:
  `
  ---
  title: 문서 제목
  category: concept | tool | comparison | trend
  created: YYYY-MM-DD
  updated: YYYY-MM-DD
  sources: [raw/ 파일 경로 목록]
  related: [관련 위키 문서 경로 목록]
  ---
  `
- 문서 간 상대 링크 사용: [Claude Code](../tools/claude-code.md)
- 출처 항상 명시: (출처: raw/articles/xxx.md)

### KB Q&A 규칙
위키에 대한 질문을 받으면:
1. INDEX.md 먼저 읽어서 관련 문서 파악
2. 관련 문서들 읽기
3. 답변을 output/ 에 .md로 저장
4. 답변이 새로운 인사이트를 포함하면 wiki/에 반영

## LLM Knowledge Base System

### 구조
- kb/raw/ : 원본 자료 (사람이 추가)
- kb/wiki/ : LLM이 컴파일하는 위키 (LLM만 편집)
- kb/output/ : Q&A 결과물

### KB 컴파일 규칙
raw/에 새 자료가 추가되면:
1. 자료를 읽고 요약 작성
2. 관련 concept/tool 문서에 내용 반영
3. 백링크 추가 (관련 문서 연결)
4. INDEX.md 업데이트 (문서 목록 + 최근 업데이트)
5. 기존 문서와 불일치하는 내용이 있으면 표시

### KB 문서 작성 규칙
- 모든 위키 문서 상단에 메타데이터:
  title, category, created, updated, sources, related
- 문서 간 상대 링크 사용
- 출처 항상 명시

### KB Q&A 규칙
위키에 대한 질문을 받으면:
1. INDEX.md 먼저 읽어서 관련 문서 파악
2. 관련 문서들 읽기
3. 답변을 output/ 에 .md로 저장
4. 답변이 새로운 인사이트를 포함하면 wiki/에 반영

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dudurim88255-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
