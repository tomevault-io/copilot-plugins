## meetly

> 모임 일정 조율 시스템은 여러 사람이 참여하는 모임의 일정을 효율적으로 조율하고 관리하는 웹 기반 플랫폼입니다. 이 프로젝트는 1인 개발로 진행되며, 모던 웹 기술 스택을 활용하여 구현됩니다.

# 1. 프로젝트 개요

모임 일정 조율 시스템은 여러 사람이 참여하는 모임의 일정을 효율적으로 조율하고 관리하는 웹 기반 플랫폼입니다. 이 프로젝트는 1인 개발로 진행되며, 모던 웹 기술 스택을 활용하여 구현됩니다.

기술 스택

```text
프로젝트는 다음과 같은 기술로 구성됩니다:

• 프론트엔드: Nextjs로 사용자 인터페이스 구현
• 백엔드: NestJS와 TypeORM을 활용한 서버 및 비즈니스 로직 처리
• 데이터베이스: postgres 사용한 관계형 데이터 관리
• 패키지 매니저: pnpm을 통한 의존성 관리

핵심 목표

시스템의 주요 목표는 모임 참여자들이 개별적으로 가능한 일정을 투표하고, 이를 시각적 히트맵으로 표현하여 모두가 참여 가능한 최적의 시간을 찾는 것입니다. 두 가지 모임 유형을 지원합니다:
• 시간 단위 모임: 미팅, 식사 등 짧은 기간의 약속에 적합하며 날짜와 시간을 함께 선택
• 날짜 단위 모임: 여행, 워크샵 등 장기간 모임에 적합하며 출발 날짜와 기간을 설정

주요 기능
시스템은 사용자 인증, 모임 생성 및 관리, 일정 투표, 히트맵 시각화, 최종 일정 확정, 4단계 자동 알림(모임 전날, 당일 오전, 30분 전, 시작 시간) 기능을 제공합니다. 관계형 데이터베이스를 통해 사용자, 모임, 일정, 투표, 알림 데이터 간의 관계를 효율적으로 관리합니다.

개발 접근 방식
프로젝트는 MVP(Minimum Viable Product) 우선으로 진행되며, 핵심 기능을 먼저 구현한 후 단계적으로 고급 기능을 추가하는 방식을 따릅니다. 명확한 요구사항 정의와 데이터베이스 설계를 기반으로 확장 가능한 아키텍처를 구축합니다.
```

# 2. 기술 스택

- backend: nestjs
- frontend: nextjs(v15 / AppRouter)
- db: postgres
- 모노 레포 구조

# 3. 프로젝트 구조

```
├── apps
│   ├── backend
│   │   ├── http
│   │   │   ├── auth.http
│   │   │   ├── http-client.env.json
│   │   │   └── user.http
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── README.md
│   │   ├── src
│   │   │   ├── app.module.ts
│   │   │   ├── auth
│   │   │   │   ├── auth.controller.spec.ts
│   │   │   │   ├── auth.controller.ts
│   │   │   │   ├── auth.module.ts
│   │   │   │   ├── auth.service.spec.ts
│   │   │   │   ├── auth.service.ts
│   │   │   │   ├── decorator
│   │   │   │   │   ├── authorization.decorator.ts
│   │   │   │   │   ├── public.decorator.ts
│   │   │   │   │   └── rbac.decorator.ts
│   │   │   │   ├── guard
│   │   │   │   │   ├── auth.guard.ts
│   │   │   │   │   ├── google-auth.guard.ts
│   │   │   │   │   └── rbac.guard.ts
│   │   │   │   ├── middleware
│   │   │   │   │   └── bearer-token.middleware.ts
│   │   │   │   └── strategy
│   │   │   │       └── google.strategy.ts
│   │   │   ├── common
│   │   │   │   └── constants
│   │   │   │       └── env.constant.ts
│   │   │   ├── controller
│   │   │   │   └── module
│   │   │   │       ├── module.service.spec.ts
│   │   │   │       └── module.service.ts
│   │   │   ├── main.ts
│   │   │   └── user
│   │   │       ├── dto
│   │   │       │   └── create-user.dto.ts
│   │   │       ├── entiy
│   │   │       │   └── user.entity.ts
│   │   │       ├── user.controller.spec.ts
│   │   │       ├── user.controller.ts
│   │   │       ├── user.module.ts
│   │   │       ├── user.service.spec.ts
│   │   │       └── user.service.ts
│   │   ├── test
│   │   │   ├── app.e2e-spec.ts
│   │   │   └── jest-e2e.json
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   └── frontend
│       ├── eslint.config.js
│       ├── index.html
│       ├── next-env.d.ts
│       ├── next.config.ts
│       ├── package.json
│       ├── public
│       │   └── vite.svg
│       ├── README.md
│       ├── src
│       │   ├── app
│       │   │   ├── globals.css
│       │   │   ├── layout.tsx
│       │   │   └── page.tsx
│       │   ├── App.css
│       │   ├── App.tsx
│       │   ├── assets
│       │   │   └── react.svg
│       │   ├── index.css
│       │   └── main.tsx
│       ├── tsconfig.app.json
│       ├── tsconfig.json
│       ├── tsconfig.node.json
│       └── vite.config.ts
├── CLAUDE.md
├── package.json
├── packages
│   └── shared
│       └── package.json
├── pnpm-lock.yaml
├── pnpm-workspace.yaml
└── README.md
```

# 4. 핵심 명령어

```text
**개발 서버(backend, frontend) 시작**: pnpm run dev
**개발 frontend서버 시작**: pnpm --filter frontend dev
**개발 backend서버 시작**: pnpm --filter backend start:dev
```

# 5. git, github 규칙

- 커밋 메시지 규칙
  모노레포에서는 어떤 앱/패키지를 수정했는지 명시.

```bash
git commit -m "feat(frontend): Add login page"
git commit -m "fix(backend): Fix TypeORM connection"
git commit -m "chore(shared): Add common types"
git commit -m "docs: Update README"
```

- 브랜치 전략

```bash
# 기능 개발
git checkout -b feature/frontend-login
git checkout -b feature/backend-auth

# 특정 앱만 수정
git checkout -b fix/frontend-ui-bug
git checkout -b fix/backend-database-connection
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yeolsss)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yeolsss)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
