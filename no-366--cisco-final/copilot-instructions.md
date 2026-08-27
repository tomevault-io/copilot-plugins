## cisco-final

> - 모든 UI 컴포넌트는 ShadCN을 사용해야 합니다.

## Component Guidelines

### ShadCN Components

- 모든 UI 컴포넌트는 ShadCN을 사용해야 합니다.
- 컴포넌트 사용 전 설치 여부를 확인해야 합니다: `/components/ui/` 디렉토리 체크
- 컴포넌트 설치 명령어를 사용해야 합니다: `npx shadcn@latest add [component-name]`
    - 주의: `npx shadcn-ui@latest add` 명령어는 deprecated. 반드시 `npx shadcn@latest add` 명령어를 사용해야 합니다.

### Icons

- 모든 아이콘은 Lucide React를 사용해야 합니다
- 아이콘 import 방법: `import { IconName } from "lucide-react"`
- 예시: `import { Search, Menu } from "lucide-react"`

### Component Structure

- 컴포넌트는 `/components` 디렉토리 내에 위치해야 합니다
- UI 컴포넌트는 `/components` 디렉토리 내에 위치해야 합니다
- 페이지별 컴포넌트는 `/app` 디렉토리 내 해당 라우트에 위치해야 합니다

### Color Scheme

- 애플리케이션 전체에서 일관된 색상을 사용하며 테마 변경에 용이하기 위해 먼저 테마를 정하고 tailwind.config.js에 정의합니다
- 모든 색상은 체계화를 위해 tailwind.config.js에 정의된 색상을 사용합니다

### Best Practices

- TypeScript 타입은 반드시 정의해야 합니다
- 컴포넌트는 재사용 가능하도록 설계해야 합니다

---
> Source: [No-366/cisco-final](https://github.com/No-366/cisco-final) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
