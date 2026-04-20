## next-proto

> 이 프로젝트는 Feature-Sliced Design과 도메인 중심 아키텍처를 결합한 구조를 따릅니다.

# Next.js 관리자 대시보드 아키텍처 가이드

## 📁 프로젝트 구조

이 프로젝트는 Feature-Sliced Design과 도메인 중심 아키텍처를 결합한 구조를 따릅니다.

### 핵심 디렉토리 구조

```
src/
├── app/ # 라우팅 및 전역 설정
│   ├── (pages)/                  # 페이지 그룹
│   │   └── page-name/
│   ├── providers/                # 전역 provider
│   └── styles/                   
│       ├── animation.css         # 애니메이션 CSS 정의
│       ├── globals.css           # 전역 CSS 설정
│       └── font.ts               # 폰트 정의
├── view/                         # 페이지별 UI 컴포넌트 및 lib
│   └── page-name/   
│       ├── ui/                     
│       └── lib/ 
├── widgets/                      # 재사용 가능한 복합 컴포넌트 
├── domains/                      # 도메인별 로직
│   ├── api/                      # CRUD
│   ├── lib/                      # hooks 또는 기타 유틸 함수
│   ├── model/                    # type, validation, store 등
│   └── index.ts     
└── shared/                       # 공통
    ├── api/                      # api instance
    ├── config/                   # endpoint, route 설정
    ├── lib/                      # 공통 hoos 및 유틸 함수
    ├── model/                    # 공통 type
    ├── ui/                       # 공통 UI 컴포넌트트
    └── index.ts
```

## 🔧 아키텍처 규칙

### 1. 레이어별 의존성 규칙

- **app/**: 라우팅과 전역 설정만 담당. 비즈니스 로직 금지
- **pages/**: 특정 페이지에서만 사용되는 UI와 훅
- **widgets/**: 여러 페이지에서 재사용되는 복합 컴포넌트
- **domain/**: 특정 도메인의 모든 비즈니스 로직 포함
- **shared/**: 모든 레이어에서 사용 가능한 공통 코드

### 2. 도메인 구조 규칙

각 도메인 폴더는 다음 구조를 반드시 따라야 합니다:

```
domain/[domain-name]/
├── 📁api/           # API 호출 함수
├── 📁lib/           # 도메인별 React 훅 및 기타 유틸리티 함수수
├── 📁model/         # 도메인별 타입, validation, store 정의
├── 📁config/        # 도메인별 설정 및 상수
├── 📁ui/            # 도메인별 UI 컴포넌트
└── 📄index.ts       # 도메인 public API
```

### 3. 파일명 규칙

- **디렉토리**: kebab-case (예: `common-code`, `user-profile`)
- **React 컴포넌트**: PascalCase (예: `UserTable.tsx`)
- **훅**: camelCase with `use` prefix (예: `useUserData.ts`)
- **유틸리티**: camelCase (예: `formatDate.ts`)
- **타입**: PascalCase (예: `UserType.ts`)
- **API**: camelCase (예: `userApi.ts`)

### 4. Import 규칙

```typescript
// ✅ 올바른 import 순서
import React from 'react'                    // 외부 라이브러리
import { NextPage } from 'next'              // Next.js 관련

import { UserForm } from '@/pages/user'      // pages 레이어

import { Header } from '@/widgets/header'    // widgets 레이어

import { UserTable } from '@/domain/user'   // domain 레이어

import { Button } from '@/shared/ui'         // shared 레이어

// ❌ 잘못된 의존성 방향
import { userApi } from '@/domain/user' // pages에서 domain 직접 접근 금지
```

### 5. 컴포넌트 구조 규칙

```typescript
// ✅ 컴포넌트 파일 구조
import { FC } from 'react'

// 타입 정의
interface Props {
  // props 타입
}

// 컴포넌트
const ComponentName: FC<Props> = ({ ...props }) => {
  // 로직
  
  return (
    // JSX
  )
}
export default ComponentName

// 헬퍼 함수들
export const helperFunction = () => {
  // ...
}

// 정적 데이터
export const STATIC_DATA = {
  // ...
}
```

이 아키텍처를 준수하여 확장 가능하고 유지보수가 용이한 관리자 대시보드를 구축하세요.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rladydqls99) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
