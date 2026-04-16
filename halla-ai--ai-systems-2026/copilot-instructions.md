## ai-systems-2026

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

제주한라대학교 인공지능학과 4학년 "에이전틱 워크플로우와 하네스 엔지니어링" 강의 사이트.
Astro 6 + Starlight 0.38 기반 정적 문서 사이트. 배포: https://ai-systems-2026.halla.ai

## Commands

```bash
pnpm install --frozen-lockfile   # 의존성 설치
pnpm run dev                     # 개발 서버 (localhost:4321)
pnpm run build                   # 정적 빌드 → dist/
pnpm run preview                 # 빌드된 사이트 미리보기
pnpm run astro check             # 타입 체크

make new-week N=07               # 주차별 MDX 스캐폴딩
make new-lab N=05                # 실습 MDX 스캐폴딩
make status                      # 콘텐츠 현황 확인
```

## Architecture

- **Framework**: Astro 6.0 + @astrojs/starlight 0.38
- **Package Manager**: pnpm 10.30.3 (package.json의 packageManager 필드로 고정)
- **배포**: GitHub Actions → GitHub Pages (main push 시 자동)

### 콘텐츠 구조

```
src/content/docs/
├── index.mdx              # 홈 (splash 템플릿)
├── syllabus.md             # 강의계획서
├── weeks/week-{01-16}.mdx  # 16주차 강의노트 (Korean)
├── labs/lab-{01-12}.mdx    # 12개 실습 (Korean)
├── capstone/               # 캡스톤 프로젝트
├── reference/              # 참고자료
├── contribute/             # 기여 가이드
└── en/                     # English locale (mirrors root structure)
    ├── index.mdx
    ├── syllabus.md
    ├── weeks/week-{01-16}.mdx
    ├── labs/lab-{01-12}.mdx
    ├── capstone/
    ├── reference/
    └── contribute/
```

- **다국어**: Korean(root, `/`) + English(`/en/`). Starlight 내장 언어 전환기 사용.

### 핵심 설정 파일

- `astro.config.mjs` — 사이드바 네비게이션, 사이트 설정
- `src/content.config.ts` — 콘텐츠 스키마 (week, phase, difficulty 등 커스텀 필드)
- `src/styles/custom.css` — 디자인 시스템 (다크 모드 우선, Space Grotesk + Pretendard 폰트)

### 콘텐츠 작성 규칙

- MDX 파일에서 Starlight 컴포넌트 사용: `<Aside>`, `<Steps>`, `<Badge>`, `<Card>`, `<CardGrid>`, `<Tabs>`, `<TabItem>`
- 커스텀 CSS 클래스: `.assignment-box` (과제 블록), `.diagram` (다이어그램), `.phase-tag`, `.week-meta`
- 이미지는 `src/assets/weeks/week-XX/`에 저장 후 상대경로로 import
- 프론트매터 필수 필드: title, description, week, phase, phase_title, difficulty

### 사이드바 변경

새 페이지 추가 시 `astro.config.mjs`의 sidebar 배열에 항목 추가 필요. 5개 Phase로 구분된 구조 유지.

## 과제 제출 (학생)

```
assignments/[lab/week]-XX/[학번]/
```

PR 제목 형식: `[제출] Lab 01 - 20230001 홍길동`
PR 전 반드시 `pnpm run build` 성공 확인.

## 언어

- 콘텐츠: 한국어 기본, 기술 용어는 영어 혼용
- 커밋 메시지: 한국어 가능, conventional commits 형식 권장

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/halla-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
