## blog-admin

> 블로그 관리자 페이지로, Clean Architecture 패턴을 적용한 Next.js 애플리케이션입니다.

# Blog Admin - Project Guide

블로그 관리자 페이지로, Clean Architecture 패턴을 적용한 Next.js 애플리케이션입니다.

## Architecture Overview

```
src/
├── domain/           # 비즈니스 로직 계약 (엔티티, 리포지토리 인터페이스)
├── application/      # 유스케이스 (비즈니스 로직 조합)
├── infrastructure/   # 기술 구현 (API 클라이언트, 리포지토리 구현체)
└── presentation/     # UI 레이어 (React 컴포넌트, 훅, 컨텍스트)
```

각 레이어별 상세 문서는 해당 디렉토리의 `CLAUDE.md`를 참조하세요.

## Tech Stack

- **Framework**: Next.js 16 (App Router)
- **Language**: TypeScript
- **State Management**: TanStack React Query v5
- **HTTP Client**: Axios
- **Auth**: JWT (js-cookie)
- **Editor**: @uiw/react-md-editor
- **Styling**: Tailwind CSS v4

## Key Patterns

### Dependency Injection
```
Presentation → Application → Domain ← Infrastructure
```
- `DependencyProvider`에서 모든 의존성 주입
- Use Case는 Repository 인터페이스에만 의존

### Data Flow
```
API (snake_case) → Mapper → Domain (camelCase) → Use Case → Hook → Component
```

### Naming Conventions

| Layer | Pattern | Example |
|-------|---------|---------|
| Entity | PascalCase | `Post`, `Category` |
| Repository Interface | `I` prefix | `IPostRepository` |
| Use Case | Entity + Action | `CreatePostUseCase` |
| Hook | `use` prefix | `usePosts`, `useCreatePost` |
| Component | Feature + Type | `PostForm`, `MediaGrid` |

## Commands

```bash
npm run dev      # 개발 서버 (http://localhost:3000)
npm run build    # 프로덕션 빌드
npm run lint     # ESLint 검사
npm run test     # Jest 단위 테스트
npm run test:e2e # Playwright E2E 테스트
```

## Environment Variables

```env
NEXT_PUBLIC_API_URL=https://blog-api.dltmxm.link/api/admin
```

## Project Structure

```
src/
├── app/                      # Next.js App Router
│   ├── (authenticated)/      # 인증 필요 라우트 그룹
│   ├── (post-editor)/        # 에디터 전용 레이아웃
│   └── login/                # 로그인 페이지
├── domain/                   # 도메인 레이어
│   ├── entities/             # Post, Category, Tag, Media, Project, User
│   └── repositories/         # 리포지토리 인터페이스
├── application/              # 애플리케이션 레이어
│   └── usecases/             # 도메인별 유스케이스
├── infrastructure/           # 인프라 레이어
│   ├── api/                  # ApiClient
│   ├── auth/                 # TokenStorage
│   ├── mappers/              # API ↔ Domain 변환
│   └── repositories/         # 리포지토리 구현체
└── presentation/             # 프레젠테이션 레이어
    ├── components/           # React 컴포넌트
    ├── hooks/                # 커스텀 훅
    ├── context/              # AuthContext
    └── providers/            # DependencyProvider
```

## Error Handling

| Layer | Strategy |
|-------|----------|
| Domain | 명시적 에러 throw |
| Use Case | 입력 검증, 검증 실패 시 throw |
| Repository | 404 → null 반환, 그 외 → throw |
| Presentation | Toast 알림, Error Boundary |

## API Integration

- Base URL: `NEXT_PUBLIC_API_URL` 환경변수
- Authentication: Bearer 토큰 (Authorization 헤더)
- API 필드: snake_case
- 페이지네이션: `page`, `per_page`, `total`, `total_pages`

---
> Source: [DongGwui/blog_admin](https://github.com/DongGwui/blog_admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
