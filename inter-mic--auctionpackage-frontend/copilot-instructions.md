## auctionpackage-frontend

> - コンポーネント: PascalCase (例: `MemberFormFields.tsx`)

# コーディング規約

## ファイル命名規則
- コンポーネント: PascalCase (例: `MemberFormFields.tsx`)
- フック: camelCase + `use` prefix (例: `useCommonSetup.ts`)
- 型定義: PascalCase (例: `UserData`)
- CSSモジュール: camelCase + `.module.css` (例: `CommonRegister.module.css`)

## インポート順序
1. React関連
2. 外部ライブラリ
3. カスタムフック
4. 型定義
5. コンポーネント
6. API
7. スタイル

```typescript
import React from "react";
import { useState, useEffect } from "react";
// 外部ライブラリ
import { toast } from "react-toastify";
// カスタムフック
import { useCommonSetup } from "@/hooks/useCommonSetup";
// 型定義
import { UserData } from "@/types/admin/member/register";
// コンポーネント
import { MemberFormFields } from "@/components/common/MemberFormFields";
// API
import { useUserRegistAPI } from "@/hooks/api/admin/user/useUserRegistAPI";
// スタイル
import styles from "@/styles/CommonRegister.module.css";
```

## TypeScript規約
- 型定義は `src/types/` ディレクトリに配置
- インターフェース名は `Props` または具体的な名前
- 型エクスポートは `export type` を使用

```typescript
export type UserData = {
  userId?: number;
  loginId?: string;
  userName?: string;
  // ...
};

interface Props {
  member: UserData;
  handleChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  errors?: Errors;
}
```

## コンポーネント規約
- 関数コンポーネントを使用
- Props型は明示的に定義
- デフォルトエクスポートは避け、名前付きエクスポートを使用

```typescript
export const ComponentName: React.FC<Props> = ({ prop1, prop2 }) => {
  // コンポーネント実装
};
```

## フック規約
- カスタムフックは `use` プレフィックス
- 複数のフックをまとめる場合は `useCommonSetup` のような共通フックを作成
- APIフックは `src/hooks/api/` に配置

## エラーハンドリング
- APIエラーは `useApiRequest` フックで統一処理
- フォームエラーは `errors` オブジェクトで管理
- トースト通知は `react-toastify` を使用

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/inter-mic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
