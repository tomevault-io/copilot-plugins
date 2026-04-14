## shoemoa-frontend

> This document provides a summary of the Shoemoa Frontend project, intended for use by the Gemini CLI.

# Project: Shoemoa Frontend

This document provides a summary of the Shoemoa Frontend project, intended for use by the Gemini CLI.

## Project Overview

Shoemoa Frontend is the user interface for the Shoemoa shoe shopping mall. It is a [Next.js](https://nextjs.org/) application written in [TypeScript](https://www.typescriptlang.org/) and styled with [Tailwind CSS](https://tailwindcss.com/) and [shadcn/ui](https://ui.shadcn.com/). The project was initially generated using [Vercel v0](https://v0.dev/).

The application includes user-facing features like sign-up/login, product lists with search and sort, product details, a shopping cart, and an order page. It also has an admin section for managing products. The frontend is designed to be responsive and will eventually connect to a Spring Boot REST API for backend services.

## Building and Running

### Development

To run the application in development mode:

```bash
npm run dev
```

This will start a development server, typically on [http://localhost:3000](http://localhost:3000).

### Production Build

To build the application for production:

```bash
npm run build
```

The build artifacts will be stored in the `.next` directory. Note that `typescript.ignoreBuildErrors` is set to `true` in `next.config.mjs`, so TypeScript errors will not block the build.

To start a production server:

```bash
npm run start
```

### Linting

To check the code for linting errors:

```bash
npm run lint
```

## Development Conventions

*   **UI Components**: The project uses `shadcn/ui` for many of its UI components. These are located in `components/ui`.
*   **API Integration**: API-related code can be found in `lib/api`. The base URL for the API is configured via the `VITE_API_BASE_URL` environment variable.
*   **Mock Data**: Mock data for development is located in `lib/mock-data.ts`.
*   **Deployment**: The project is configured for continuous deployment on [Vercel](https://vercel.com/). Pushing to the `main` branch will trigger a new deployment.

## Agent Instructions

- 에이전트는 항상 한국어로 응답해야 합니다.
- 모든 아티팩트 문서(`implementation_plan.md`, `walkthrough.md`, `task.md` 등)는 항상 한국어로 작성해야 합니다.
- Git 커밋 메시지는 **Angular Commit Convention**을 따르며, 설명은 항상 한국어로 작성해야 합니다. (예: `feat(auth): 로그인 기능 구현`)

---

---

## 핵심 개발 규칙 및 보안 지침

- **인증 패턴 준수**: 모든 인증 관련 개발은 [docs/auth.md](file:///Users/jino/study/project/shoemoa/shoemoa-frontend/docs/auth.md)에 정의된 **In-Memory Access Token + HTTP-only Refresh Token (Silent Refresh)** 패턴을 엄격히 따라야 합니다.
- **상태 관리**: 전역 인증 상태는 `lib/hooks/use-auth.tsx`의 `AuthProvider`를 통해 관리하며, 직접적인 토큰 접근은 지양합니다.
- **UI 일관성**: 모든 사용자 페이지는 `app/(user)/layout.tsx`를, 관리자 페이지는 `app/admin/layout.tsx`를 통해 헤더를 제공해야 합니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aqwsde321)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aqwsde321)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
