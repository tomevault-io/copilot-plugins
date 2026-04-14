## clansync

> React 컴포넌트 작성 규칙 및 shadcn/ui 사용 패턴


# React 컴포넌트 규칙

## 컴포넌트 구조

```tsx
// ✅ Props 타입은 컴포넌트 바로 위에 정의
interface ClanCardProps {
  clan: Clan;
  onJoin?: (clanId: string) => void;
}

export function ClanCard({ clan, onJoin }: ClanCardProps) {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{clan.name}</CardTitle>
      </CardHeader>
    </Card>
  );
}
```

## shadcn/ui 사용

```tsx
// ✅ shadcn/ui 컴포넌트 우선 사용
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';

// ❌ 직접 스타일 작성 지양 (shadcn 컴포넌트가 있는 경우)
<button className="bg-blue-500 px-4 py-2 rounded">클릭</button>
```

## 파일명 규칙
- 컴포넌트: `PascalCase.tsx` (예: `ClanCard.tsx`)
- 유틸리티/훅: `camelCase.ts` (예: `useClanData.ts`)
- 페이지: `page.tsx`, 레이아웃: `layout.tsx`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kuble)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kuble)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
