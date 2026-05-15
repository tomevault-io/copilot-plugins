## project-structure

> - **백엔드 코드**: [backend/](mdc:backend/) 폴더에만 작성

# 프로젝트 구조 규칙

## 폴더 분리 원칙
- **백엔드 코드**: [backend/](mdc:backend/) 폴더에만 작성
- **프론트엔드 코드**: [frontend/](mdc:frontend/) 폴더에만 작성
- **절대 금지**: 백엔드와 프론트엔드 코드 혼재

## 프론트엔드 구조
```
frontend/src/
├── components/
│   ├── ui/          # Shadcn UI 컴포넌트
│   ├── guide/       # 가이드 관련 컴포넌트
│   ├── admin/       # 관리자 기능 컴포넌트
│   └── common/      # 공통 컴포넌트
├── pages/           # 페이지 컴포넌트
├── hooks/           # 커스텀 훅
├── services/        # API 서비스
├── types/           # TypeScript 타입 정의
└── utils/           # 유틸리티 함수
```

## 핵심 파일들
- 프로젝트 요구사항: [prd.md](mdc:prd.md)
- AI 개발 표준: [shrimp-rules.md](mdc:shrimp-rules.md)
- 프론트엔드 설정: [frontend/package.json](mdc:frontend/package.json)
description:
globs:
alwaysApply: false
---

---
> Source: [jang-seung-hee/rental-news](https://github.com/jang-seung-hee/rental-news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
