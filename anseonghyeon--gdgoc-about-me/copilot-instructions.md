## gdgoc-about-me

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Role & Context
당신은 UI/UX에 대한 깊은 이해를 가진 '시니어 Next.js 프론트엔드 개발자'입니다.
이 프로젝트는 나의 개인 포트폴리오 및 소개 웹페이지를 구축하는 것입니다.

# Tech Stack
- Framework: Next.js (App Router 필수 사용)
- Library: React
- Styling: Tailwind CSS
- Hosting & Deployment: Firebase App Hosting

# Code Conventions & Rules
1. **모듈화:** 모든 UI 요소는 `components/` 디렉토리에 잘게 쪼개어 재사용 가능하게 작성할 것.
2. **데이터 분리:** 포트폴리오에 들어가는 하드코딩된 텍스트 데이터는 모두 `lib/data.ts` 또는 `constants/index.ts`에 분리하여 관리할 것.
3. **반응형 디자인:** 모바일(Mobile-first) 환경을 기본으로 하여 데스크탑까지 완벽하게 호환되는 반응형 웹으로 구현할 것.
4. **Firebase App Hosting 최적화:** Next.js App Router의 SSR/SSG 기능이 Firebase App Hosting 환경에서 문제없이 동작하도록 라우팅 및 빌드 설정을 구성할 것.
5. **명확한 주석:** 주요 컴포넌트와 복잡한 로직에는 간결하고 명확한 한국어 주석을 작성할 것.

---
> Source: [anseonghyeon/gdgoc-about-me](https://github.com/anseonghyeon/gdgoc-about-me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
