## claude-code-docs-ko

> Claude Code 공식 문서를 기반으로 한국어 문서를 관리하고, Claude Code 활용법을 체계화하는 프로젝트.

# Claude Code 문서 프로젝트

## 프로젝트 개요
Claude Code 공식 문서를 기반으로 한국어 문서를 관리하고, Claude Code 활용법을 체계화하는 프로젝트.

## 디렉토리 구조
```
docs/          → 영문 원본 문서 (Mintlify에서 가져온 마크다운)
ko-docs/       → 한국어 번역/로컬라이즈 문서
.claude/
  skills/      → 커스텀 워크플로우 스킬
  rules/       → 경로별 조건부 규칙
  settings.json → 훅, 권한, 자동화 설정
```

## 빌드 & 테스트
```bash
# 마크다운 린팅
npx markdownlint-cli2 "docs/**/*.md" "ko-docs/**/*.md"

# 링크 검증
npx markdown-link-check docs/*.md

# 문서 미리보기 (로컬)
npx serve docs/
```

## 코딩 컨벤션

### 마크다운 작성 규칙
- 헤딩은 `#`부터 순차적으로 사용 (h1 → h2 → h3)
- 코드 블록에 반드시 언어 지정 (```bash, ```json, ```typescript 등)
- 테이블은 가독성 위해 컬럼 정렬
- 파일명: `숫자-kebab-case.md` (예: `01-introduction.md`)

### 한국어 번역 규칙
- 기술 용어는 영문 유지 (예: hook, skill, agent, MCP, REPL)
- 명령어, 코드, 파일 경로는 번역하지 않음
- 자연스러운 한국어 — 직역보다 의역 우선
- 문체: 경어체 (합니다/입니다)
- 첫 등장 시 `영문(한글)` 병기 후 이후 한글만 사용

### 파일 관리
- 새 문서 추가 시 `docs/`와 `ko-docs/` 동시 생성
- 영문 원본 수정 시 `ko-docs/` 대응 파일도 업데이트
- 각 문서 상단에 원본 소스 URL 주석 포함

## 자주 쓰는 명령어
```bash
# 특정 문서 번역
/translate docs/12-hooks.md

# 문서 리뷰
/review-docs ko-docs/guides/hooks.md

# 새 문서 생성
/new-doc "topic-name"
```

## 주의사항
- `docs/` 원본은 가급적 수정하지 않음 (원본 보존)
- 번역 시 코드 블록 내용은 절대 번역하지 않음
- `.claude/settings.json`은 팀 공유용이므로 개인 설정은 `.claude/settings.local.json`에

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aisyncclub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
