## gugbab-integrated-admin-poc

> Feature-Sliced Design은 확장 가능하고 유지보수 가능한 프론트엔드 애플리케이션을 구축하기 위한 아키텍처 방법론입니다.

# Feature-Sliced Design (FSD) 아키텍처 가이드라인

## FSD 개요

Feature-Sliced Design은 확장 가능하고 유지보수 가능한 프론트엔드 애플리케이션을 구축하기 위한 아키텍처 방법론입니다.

## 레이어 구조

### 1. app (애플리케이션 레이어)
- **역할**: 애플리케이션의 진입점과 전역 설정
- **위치**: `src/app/`
- **내용**: 라우팅, 전역 스타일, 프로바이더, 레이아웃

```typescript
// src/app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <Providers>
          {children}
        </Providers>
      </body>
    </html>
  );
}
```

### 2. pages (페이지 레이어)
- **역할**: 페이지별 컴포넌트 조합
- **위치**: `src/pages/`
- **내용**: 페이지 컴포넌트, 라우팅 로직

```typescript
// src/pages/UserProfile/index.tsx
import { UserProfileWidget } from '@/widgets/UserProfile';
import { UserSettingsFeature } from '@/features/UserSettings';

export default function UserProfilePage() {
  return (
    <div>
      <UserProfileWidget />
      <UserSettingsFeature />
    </div>
  );
}
```

### 3. widgets (위젯 레이어)
- **역할**: 독립적인 UI 블록
- **위치**: `src/widgets/`
- **내용**: 여러 기능을 조합한 복합 컴포넌트

```typescript
// src/widgets/UserProfile/index.tsx
import { UserInfoFeature } from '@/features/UserInfo';
import { UserAvatarFeature } from '@/features/UserAvatar';

export function UserProfileWidget() {
  return (
    <div className="user-profile">
      <UserAvatarFeature />
      <UserInfoFeature />
    </div>
  );
}
```

### 4. features (기능 레이어)
- **역할**: 비즈니스 기능 단위
- **위치**: `src/features/`
- **내용**: 특정 기능의 UI, 로직, API

```typescript
// src/features/UserInfo/index.tsx
import { useUserInfo } from './api/useUserInfo';
import { UserInfoForm } from './ui/UserInfoForm';

export function UserInfoFeature() {
  const { user, updateUser } = useUserInfo();
  
  return <UserInfoForm user={user} onSubmit={updateUser} />;
}
```

### 5. entities (엔티티 레이어)
- **역할**: 비즈니스 엔티티
- **위치**: `src/entities/`
- **내용**: 도메인 모델, 타입, 유틸리티

```typescript
// src/entities/User/index.ts
export interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
}

// src/entities/User/api/userApi.ts
export const userApi = {
  getUser: (id: string): Promise<User> => {
    // API 구현
  },
  updateUser: (user: User): Promise<User> => {
    // API 구현
  },
};
```

### 6. shared (공유 레이어)
- **역할**: 재사용 가능한 공통 코드
- **위치**: `src/shared/`
- **내용**: UI 컴포넌트, 유틸리티, 타입, 상수

```typescript
// src/shared/ui/Button/index.tsx
export interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'secondary';
}

export function Button({ children, onClick, variant = 'primary' }: ButtonProps) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {children}
    </button>
  );
}
```

## 의존성 규칙

### 허용되는 의존성
- **app**: 모든 레이어에 의존 가능
- **pages**: widgets, features, entities, shared에 의존 가능
- **widgets**: features, entities, shared에 의존 가능
- **features**: entities, shared에 의존 가능
- **entities**: shared에만 의존 가능
- **shared**: 다른 레이어에 의존 불가

### 금지되는 의존성
- 하위 레이어가 상위 레이어에 의존하는 것
- 같은 레이어 내에서 다른 슬라이스에 의존하는 것 (shared 제외)

## 슬라이스 구조

각 슬라이스는 다음과 같은 구조를 가집니다:

```
feature/
├── ui/           # UI 컴포넌트
├── api/          # API 관련 로직
├── model/        # 비즈니스 로직
├── lib/          # 유틸리티 함수
├── config/       # 설정
└── index.ts      # Public API
```

## 개발 가이드라인

### 1. 슬라이스 생성 규칙
- **기능별 분리**: 비즈니스 기능 단위로 슬라이스 생성
- **독립성**: 각 슬라이스는 독립적으로 동작 가능해야 함
- **Public API**: `index.ts`를 통한 명확한 Public API 제공

### 2. 네이밍 규칙
- **파스칼 케이스**: 컴포넌트, 인터페이스, 타입
- **카멜 케이스**: 함수, 변수, 훅
- **스네이크 케이스**: 파일명, 폴더명

### 3. 의존성 관리
- **단방향 의존성**: 상위 레이어에서 하위 레이어로만 의존
- **순환 의존성 방지**: 순환 참조 금지
- **공유 코드**: `shared` 레이어에 재사용 가능한 코드 배치

## 참고 문서
- [Feature-Sliced Design 공식 문서](https://feature-sliced.design/kr/docs/get-started/overview)
- [FSD 아키텍처 가이드](https://feature-sliced.design/kr/docs/concepts/architecture)
- [FSD 레이어 설명](https://feature-sliced.design/kr/docs/concepts/layers)
description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/puk0806) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
