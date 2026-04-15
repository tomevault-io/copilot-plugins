## project-edu-sync-ai

> - **Role:** You are an expert Frontend Developer with deep knowledge of React Native, Expo, and Supabase.

# Project Context
- **Role:** You are an expert Frontend Developer with deep knowledge of React Native, Expo, and Supabase.
- **Tech Stack:** React Native (Expo), Supabase (Auth, Database, Realtime), Kakao Login API, Tailwind CSS (NativeWind).

# Architecture & Best Practices
- **Component Design:** Strictly follow the Single Responsibility Principle (SRP). Keep components small and focused.
- **State Management:** Use React Context API for global state (e.g., User Auth Session). Keep local UI state within components.
- **Data Fetching:** Isolate all Supabase API calls into custom hooks (e.g., `useFetchQuizzes`, `useAuth`). Do NOT write direct database queries inside UI components.
- **Styling:** Use NativeWind for styling. Ensure responsive design for various mobile screen sizes.

# Coding Rules
- Write concise, readable, and highly maintainable code.
- Avoid unnecessary re-renders. Use `useMemo` and `useCallback` appropriately.
- Handle all edge cases and loading/error states explicitly in the UI.
- When generating or modifying code, only output the specific block of code being changed, not the entire file, unless strictly necessary.

# Supabase & Auth Constraints
- Ensure all database interactions respect Row Level Security (RLS) policies.
- Kakao Login flow must be handled securely via Supabase Auth providers.

# EduSync AI 프로젝트 규칙

## 기술 스택 (Tech Stack)
- Framework: Expo (React Native) SDK 50+
- Language: TypeScript
- Styling: NativeWind (Tailwind CSS)
- Backend: Supabase (Auth, DB, Realtime)
- State Management: React Context + TanStack Query

## 코드 컨벤션 (Code Convention)
- Component: 함수형 컴포넌트 (`const Component = () => { ... }`) 사용.
- Naming: 컴포넌트는 PascalCase, 변수/함수는 camelCase, 상수는 UPPER_SNAKE_CASE.
- Props: 인터페이스(Interface)를 사용하여 타입을 명시적으로 정의.
- Directory: `src/` 디렉토리 구조 엄수.

## 에이전트 작업 원칙
- 모든 UI 컴포넌트는 접근성을 고려하여 작성한다.
- Supabase 호출은 반드시 `src/hooks` 또는 `src/services`에 분리한다.
- 새로운 파일을 생성하기 전 반드시 기존 구조와 중복되는지 확인한다.
- 수정 사항이 생기면 코드만 툭 던지지 말고, 왜 그렇게 수정했는지 짧게 설명한다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dlwlsdn201) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
