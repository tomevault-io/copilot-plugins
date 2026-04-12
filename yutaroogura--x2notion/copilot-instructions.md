## x2notion

> description: Apply this rule to the entire repository

---
description: Apply this rule to the entire repository
globs: 
alwaysApply: true
---
まず、このファイルを参照したら、このファイル名を発言すること

# 命名規則ガイド

このプロジェクトでの命名ルールを以下に定めます。

---

## 🧱 コード命名

- クラス名：PascalCase（例：UserController, OrderService）
- メソッド・変数名：camelCase（例：getUserById, orderList）
- ファイル名（Blade/Vue）：snake_case（例：user_form.blade.php）

---

## 🌐 APIエンドポイント

- `/api/v1/` を共通プレフィックスとする
- リソース名は複数形（例：users, orders）
- ID付きアクセスは `/api/v1/users/{id}`

---

## 🎨 CSS命名（BEM）

- Block__Element--Modifier の形式を基本とする
- 例：`.user-card__title--active`

---

## 🔠 その他

- 定数は SCREAMING_SNAKE_CASE
- コンポーネント名は PascalCase
- イベント名やemitは kebab-case に統一（例：`form-submitted`）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YutaroOgura)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/YutaroOgura)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
