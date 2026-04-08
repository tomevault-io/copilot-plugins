## dana-mentoring

> 이 파일은 Claude Code(claude.ai/code)가 이 저장소에서 작업할 때 참고하는 가이드입니다.

# CLAUDE.md

이 파일은 Claude Code(claude.ai/code)가 이 저장소에서 작업할 때 참고하는 가이드입니다.

## 명령어

```bash
npm run dev      # 개발 서버 시작 (localhost:3000)
npm run build    # 프로덕션 빌드
npm run start    # 프로덕션 서버 시작
npm run lint     # ESLint 실행
```

## 아키텍처

**Next.js 16 App Router** 기반 프로젝트로 TypeScript와 Tailwind CSS v4를 사용합니다.

- `app/` — App Router 페이지 및 레이아웃. 모든 라우트는 하위 디렉토리의 `page.tsx` 파일로 정의됩니다.
- `app/layout.tsx` — 모든 페이지를 감싸는 루트 레이아웃. Geist 폰트와 전역 메타데이터를 설정합니다.
- `app/globals.css` — 전역 스타일. Tailwind v4 문법인 `@import "tailwindcss"`를 사용합니다 (`@tailwind` 디렉티브 아님).

**주요 규칙:**
- 경로 별칭 `@/*`는 프로젝트 루트를 가리킵니다.
- Tailwind v4는 `tailwind.config.js` 파일이 아닌 PostCSS(`@tailwindcss/postcss`)로 설정됩니다.
- TypeScript strict 모드가 활성화되어 있습니다.
- API 라우트는 `app/api/` 하위에 작성합니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HarenKei)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HarenKei)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
