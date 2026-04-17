## education-scheduler

> 教育施設のリソース（教室・講師・講座）管理に特化したカレンダーサービス。

# ScholaTile

教育施設のリソース（教室・講師・講座）管理に特化したカレンダーサービス。

## 1. Architecture & Tech Stack

### Frontend
- **Framework:** Preact (仮想DOM、軽量・高速)
- **Language:** TypeScript
- **State Management:** `@preact/signals` (細粒度なリアクティビティによる高パフォーマンス)
- **Styling:** Vanilla CSS + CSS Grid (複数コマ跨ぎ・マルチビューのネイティブサポート)
- **Internationalization:** `i18next`, `react-i18next` (キーベースの翻訳、ブラウザロケール動的切り替え)
- **Build Tool:** Vite

### Backend
- **Runtime:** Node.js (Express)
- **Language:** TypeScript (`ts-node-dev` による開発)
- **Database:** PostgreSQL
- **ORM:** Prisma 7 (型安全なアクセス、driver-adapter による高速通信)
- **Authentication:** JWT (JSON Web Token) + `bcryptjs`. セッションは `HttpOnly` Cookie で管理。

---

## 2. Key Features

### Core Scheduling (スケジューリング)
- **動的時限表示:** 1日の時限数（TimePeriod）はDB設定により可変。名称、開始・終了時間を保持。
- **イベント行の統合:** 祝日、休暇、学校行事（ScheduleEvent）を最上部の固定行に統合表示。
- **マルチビュー:** 1日 / 1週間 / 1ヶ月 / 3ヶ月 / 6ヶ月 / 1年 の表示切り替えに対応。3ヶ月・6ヶ月・1年ビューは、システム設定で指定された開始月日を基準に期間を区切って表示。
- **個人月間予定ビュー (Personal Monthly View):** 
  - ユーザーメニューからアクセス可能。紐付けられた講師本人の予定をカレンダー形式（7曜5週等）で集約表示。
  - **レスポンシブ・フィット:** CSS Grid を活用し、画面の高さに合わせて全週が収まるよう動的にリサイズ（スクロール不要）。
  - **時限の可視化:** 各日8時限分を垂直方向に等分割し、複数時限に跨る授業は単一のカードとして高さで期間を表現。時限番号（例: 「1-4」）をラベル表示。
- **1年ビューの開始日設定:** 組織の運用に合わせて、1年ビューの開始月日（例: 4月1日、9月1日等）をシステム設定で変更可能。
- **重なり回避ロジック:** 
  - イベント行（最上部）とリソース行（各行内）の両方で、時間的に重なる要素を垂直方向にオフセットして自動回避。
- **ダブルブッキング警告:** 授業の登録・更新時、リソース（教室・講師）の重複を検知し警告。

### Resource & Label Management (リソース・ラベル管理)
- **リソースタイプ:** 「教室 (Room)」「講師 (Teacher)」「講座 (Course)」の3種類。
- **リソースのフィルター機能:** grid-corner に配置されたフィルターボタンから、表示するリソース（行）をチェックボックスで動的に絞り込み可能。
- **表示ラベルの動的変更:** リソース名や「メイン講師」「補佐講師」「課目 (Subject)」等のラベルをDBで一括管理・変更可能。

- **講師とユーザーの紐付け:** 講師リソースを特定のシステムユーザーと 1:1 で紐付け可能。
- **講座の詳細管理:** 開始/終了年月日、メイン教室、管理講師（主任・補佐）、および関連する課目（Subject）と合計時限数を管理。
- **授業方式（Delivery Method）:** 対面、オンライン、オンデマンド等の方式を定義し、各授業に複数割り当て可能。

### Administration (管理機能)
- **CRUD 画面:** 時限、教室、講師、講座、授業、行事、祝日、授業方式、ユーザー、システム設定の各管理画面。
- **インポート機能:** 
  - 祝日: Nager.Date API または JSON ファイルからインポート。
  - 講座課目: CSV からの一括インポート。
- **エクスポート機能:**
  - スケジュール: 講師本人が自身の予定を iCalendar (.ics) 形式で書き出し可能。
  - タイムテーブル: 表示中のビュー（個人月間予定を含む）を Excel (.xlsx) 形式でエクスポート可能（セルの色やレイアウトを維持）。
- **講座の複製:** 関連する課目設定を含めた講座の複製が可能。
- **講座間での授業複製:** 他の講座から指定期間の授業を、講師をクリアし、複製先講座のメイン教室を割り当てた状態で複製可能（重複回避機能付き）。
- **システム設定:** 一般ユーザーのサインアップ可否や、1年ビューの開始月日のカスタマイズが可能。
- **ユーザー管理 & 権限:** 
  - ロール（ADMIN, TEACHER, STUDENT）による RBAC。
  - **講師の授業管理:** 
    - 講座 of 「主任講師」または「副主任講師」は、その講座の授業をフル管理（追加・編集・削除）可能。
    - 授業の「メイン講師」または「サブ講師」として割りられている講師は、その授業の**「授業方式」のみ**編集が可能（他の項目は読み取り専用）。
  - 管理者は全リソースのフルアクセス権限を保持。

---

## 3. Implementation Rules & Conventions

### Coding Standards
- **Naming:** 
  - Component: PascalCase (e.g., `LessonManager.tsx`)
  - Function/Variable: camelCase
  - API Routes: RESTful (e.g., `GET /api/lessons`, `POST /api/courses`)
- **State:** グローバルまたは複雑な共有状態には Preact Signals を優先的に使用する。
- **CSS:** Vanilla CSS を使用。CSS Grid/Flexbox を最大限活用する。Component ごとに `.css` ファイルを分け、import を勝手に削除しないこと。

### Development Workflow
- **Specification First:** 仕様変更時はまず `GEMINI.md` を更新し、定義を確定させてから着手する。
- **Data Integrity:** DB保存時、空文字は原則として `null` として処理する。
- **Safety:** Git への commit/push は明示的な指示がない限り行わない。

### UI Layering (z-index)
1. `100`: `grid-corner` (左上交差点)
2. `35` / `34`: `date-header` / `period-header`
3. `30`: `event-label`
4. `26`: `event-card`
5. `25`: `grid-label` (リソース行ラベル)
6. `18`: `event-cell` (イベント行背景)

---

## 4. Core Domain Models (Data Interfaces)

### Base Types
```typescript
export type ViewType = 'day' | 'week' | 'month' | '3month' | '6month' | 'year';
export type ResourceType = 'room' | 'teacher' | 'course';
export type UserRole = 'ADMIN' | 'TEACHER' | 'STUDENT';
```

### Main Entities
- **Resource:** `id, name, type, order, userId, startDate, endDate, mainRoomId, chiefTeacherId, assistantTeacherIds, mainTeacherLabel, subTeacherLabel`
- **Lesson:** `id, subject, startDate, startPeriodId, endDate, endPeriodId, roomId, teacherId, courseId, location, subTeacherIds, deliveryMethodIds, remarks, externalTeacher, externalSubTeachers`
- **ScheduleEvent:** `id, name, startDate, startPeriodId, endDate, endPeriodId, color, location, showInEventRow, resourceIds`
- **DeliveryMethod:** `id, name, color, order`
- **TimePeriod:** `id, name, startTime, endTime, order` (IDは `p1`, `p2` ... 形式を維持)
- **Holiday:** `id, name, date, start, end`
- **ResourceLabels:** `room, teacher, course, event, mainTeacher, subTeacher, mainRoom, deliveryMethod, subject`

---

## 5. Roadmap & Project Status

### Implemented Features (Completed)
- [x] Preact + Signals + CSS Grid によるマルチビュー（1日/1週/1ヶ月/3ヶ月/6ヶ月/1年）・Sticky レイアウト
- [x] 個人月間予定ビュー (Responsive な 7曜カレンダー形式、Excel 書き出し対応)
- [x] リソースのフィルター機能 (grid-corner のチェックボックスによる行の絞り込み)
- [x] Node.js + Prisma + PostgreSQL バックエンド & JWT 認証 (HttpOnly Cookie)
- [x] 国際化 (i18n) 完全実装 (日・英対応)
- [x] 全リソースの CRUD 管理画面 (時限, 教室, 講師, 講座, 授業, 行事, 祝日, 授業方式, ユーザー)
- [x] イベント行・リソース行の重なり自動回避ロジック
- [x] 祝日・課目データのインポート機能
- [x] システム設定管理 (パブリックサインアップ等)
- [x] 講座担当講師による限定的な授業管理権限
- [x] 授業担当講師による授業方式の限定編集権限
- [x] 講座の複製機能 (関連課目含む)
- [x] 講師によるスケジュールの iCalendar (.ics) エクスポート機能
- [x] 3ヶ月/6ヶ月/1年ビューの開始月日のカスタマイズ機能
- [x] ダークテーマ / ライトテーマの完全対応

### Upcoming Tasks (Next Steps)
- [ ] ドラッグ＆ドロップによる授業の移動・編集機能
- [ ] 印刷用レイアウトの最適化
- [ ] AI によるスケジューリング最適化/支援機能の検討
- [ ] パフォーマンス最適化 (大量リソース表示時のレンダリング抑制)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yasumichi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
