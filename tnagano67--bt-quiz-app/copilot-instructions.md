## bt-quiz-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

BT管理システム — 教育機関向け小テスト・成績管理システム。元は GAS + Google スプレッドシートで構築されていたものを、モダンスタックで再構築。学習・練習目的。複数科目対応で、生徒機能（科目選択・小テスト受験・履歴・成績チャート）と教員機能（科目/生徒/問題/グレード定義のCRUD・成績ダッシュボード・CSVインポート/エクスポート）を実装済み。

実装フェーズの履歴・DBスキーマ(SQL)・今後の候補は `PLAN.md` を参照。

## 技術スタック

- **Next.js 16** App Router、`src/` ディレクトリ構成、TypeScript（strict mode）
- **React 19**
- **Tailwind CSS 4**（`@import "tailwindcss"` 構文、`@theme inline` でカスタムプロパティ定義 — `tailwind.config` ファイルなし）
- **Supabase**（`@supabase/ssr` でSSR対応認証、`@supabase/supabase-js` でDB操作）
- **Chart.js** + react-chartjs-2（成績チャート、`next/dynamic` で遅延読み込み）
- **date-fns** + date-fns-tz（Asia/Tokyo タイムゾーン対応）
- **ESLint 9** flat config（`eslint.config.mjs`、`eslint-config-next/core-web-vitals` + `/typescript`）
- **Vitest** でユニットテスト
- **Playwright** でE2Eテスト
- **Vercel** でデプロイ

## コマンド

```bash
npm run dev          # 開発サーバー起動 (localhost:3000)
npm run build        # 本番ビルド（Turbopack）
npm run start        # 本番サーバー起動
npm run lint         # ESLint 実行
npm run test         # Vitest テスト一度実行
npm run test:watch   # Vitest ウォッチモード
npx vitest run src/lib/csv-utils.test.ts  # 単一テストファイル実行
npm run test:e2e     # Playwright E2Eテスト実行
npm run test:e2e:ui  # Playwright UIモード（ブラウザ付きデバッグ）
npm run test:e2e:headed  # ブラウザ表示付きで実行
```

ユニットテストファイルはソースファイルの隣に `*.test.ts`（または `*.test.tsx`）で配置（コロケーション方式）。テスト用モックユーティリティは `src/test-utils/` に配置。E2Eテストは `e2e/` ディレクトリに配置。

## アーキテクチャ

### ルーティング（App Router）

- `/` — ロール判定: 教員→`/teacher`、生徒→`/student`、未登録→メッセージ表示、未認証→`/login`
- `/login` — Google OAuth ログイン画面
- `/auth/callback` — OAuth コールバック（Route Handler）→ `/` へリダイレクトしてロール判定
- `/student` — 生徒ホーム（科目カードグリッド表示・各科目の進捗）
- `/student/quiz` — 小テスト（Client Component、`Suspense` ラッパー、`?subject={subjectId}` 必須）
- `/student/history` — 学習履歴・再受験
- `/teacher` — 教員ダッシュボード（概要統計・グレード分布チャート・合格率推移チャート・科目フィルタ）
- `/teacher/students` — 生徒一覧（URLパラメータでフィルター・科目フィルタ）、CSVインポート
- `/teacher/students/new` — 生徒登録
- `/teacher/students/[studentId]` — 生徒詳細（30日統計・チャート）
- `/teacher/students/[studentId]/edit` — 生徒編集
- `/teacher/subjects` — 科目一覧・登録・編集・削除
- `/teacher/subjects/new` — 科目登録
- `/teacher/subjects/[subjectId]/edit` — 科目編集
- `/teacher/questions` — 問題一覧（科目フィルタ）、CSVインポート
- `/teacher/questions/new` — 問題登録（科目選択）
- `/teacher/questions/[questionId]/edit` — 問題編集
- `/teacher/grades` — グレード定義一覧（科目フィルタ）
- `/teacher/grades/new` — グレード定義登録（科目選択）
- `/teacher/grades/[gradeId]/edit` — グレード定義編集
- `/teacher/teachers` — 教員一覧、CSVインポート
- `/teacher/teachers/new` — 教員登録
- `/teacher/teachers/[teacherId]/edit` — 教員編集
- `/teacher/export` — 成績CSVエクスポート（Client Component、フィルター・件数確認・ダウンロード）
- `/api/teacher/export` — CSVダウンロード用 Route Handler（BOM付きUTF-8）

各ルートに `loading.tsx`（スケルトンUI）と `error.tsx`（リトライボタン付き、`useEffect` で `error.digest` をコンソールにログ出力）を配置済み。

### ロール分離とカラーテーマ

- **生徒側**: 青系（`blue-600` 等）、`max-w-3xl` コンテナ
- **教員側**: ティール系（`teal-700` 等）、`max-w-5xl` コンテナ（テーブル用に広め）
- 各ロールは独立した `layout.tsx` と専用ヘッダー（`Header.tsx` / `TeacherHeader.tsx`）を持つ
- **ヘッダーナビゲーション**: アクティブタブに下ボーダー表示（生徒: `border-blue-600`、教員: `border-teal-600`）。`TeacherHeader` はモバイル（`md` 未満）でハンバーガーメニュー、デスクトップでタブ表示
- **モバイルテーブル対応**: `StudentTable` は日付スコア列を `hidden lg:table-cell` でデスクトップのみ表示。`QuestionTable` は選択肢1〜4列を `hidden md:table-cell` でデスクトップのみ表示

### Server Component / Client Component の使い分け

- **Server Component**: ホーム画面、履歴、教員一覧/詳細ページ、テーブル系コンポーネント（`StudentTable`、`QuestionTable`、`GradeTable`、`SubjectTable`）、表示系コンポーネント（`SubjectCard`、`QuizQuestion`、`QuizResult`、`HistoryItem`、`StatisticsCard`）
- **Client Component** (`"use client"`): クイズページ、ヘッダー（`usePathname`）、`ScoreChart`/`GradeDistributionChart`/`PassRateTrendChart`（Chart.js、各ページから `next/dynamic` で遅延読み込み、`useMemo` で `data`/`options` メモ化）、`StudentFilter`（URL パラメータ操作）、フォーム系（`QuestionForm`/`StudentForm`/`GradeForm`/`SubjectForm`/`TeacherForm`）、CSVインポート系（`CsvImport`/`StudentCsvImport`）、`Pagination`、`ConfirmDialog`（削除確認モーダル）、各 `*DeleteButton`（5種）、ログイン画面、エクスポートページ

### 主要な lib モジュール

- `src/lib/supabase/` — Supabase クライアント: `client.ts`（ブラウザ用）、`server.ts`（SSR用・`cookies()` 使用）、`middleware.ts`（セッション更新・認証ガード）
- `src/lib/types/database.ts` — DB テーブルの型: `Subject`, `Student`, `StudentSubjectProgress`, `Teacher`, `GradeDefinition`, `Question`, `QuizRecord`
- `src/lib/quiz-logic.ts` — `shuffleArray`（Fisher-Yates）、`shuffleChoices`、`gradeQuiz`、`verifyScore`（サーバー側再採点）
- `src/lib/grade-logic.ts` — `calculateGradeAdvancement`（連続日数・昇級判定）
- `src/lib/date-utils.ts` — `getTodayJST`、`getRecentDates`、`isTakenToday` 等（Asia/Tokyo）
- `src/lib/csv-utils.ts` — `parseCsvRows`（RFC 4180 準拠CSVパーサー、クォート内改行対応）、`generateCsvText`（2D配列→CSV文字列変換）
- `src/lib/validation.ts` — `validateQuestionInput`、`validateStudentInput`、`validateTeacherInput`、`validateSubjectInput`（CSVインポート等の入力値検証を一元化）
- `src/lib/export-utils.ts` — `getGradeFilter`（Map O(1) ルックアップ）、`calculateStudentStats`、`formatStudentExportRow`、`formatRecordExportRow`（CSVエクスポートのロジック）
- `middleware.ts`（ルート） — `/student/*` と `/teacher/*` ルートの認証ガード

### Server Actions

- `src/app/student/quiz/actions.ts` — 小テスト結果保存（認証・本人確認・日次制限・サーバー側再採点・進級計算・DB保存）。進捗は `student_subject_progress` テーブルで科目別に管理
- `src/app/teacher/subjects/actions.ts` — 科目のCRUD。作成時に全生徒分の `student_subject_progress` を自動作成（1000件バッチ）。削除時に参照チェック
- `src/app/teacher/questions/actions.ts` — 問題のCRUD + CSVインポート（upsert、`subject_id` 必須、複合 UNIQUE `(subject_id, question_id)`、同一ファイル内の重複 `question_id` は後勝ちで自動重複排除）
- `src/app/teacher/students/actions.ts` — 生徒のCRUD（登録・編集・削除）+ CSVインポート。作成時に全科目分の `student_subject_progress` を自動作成（`getFirstGradeNames()` で全科目・全グレードを2クエリ並列取得 + Map グループ化）。削除時は `quiz_records` → `student_subject_progress` → `students` の順に明示的に関連データを先に削除
- `src/app/teacher/grades/actions.ts` — グレード定義のCRUD（削除時に `student_subject_progress` の参照チェック、複合 UNIQUE `(subject_id, grade_name)` / `(subject_id, display_order)`）
- `src/app/teacher/teachers/actions.ts` — 教員のCRUD（登録・編集・削除）+ CSVインポート（upsert + `ignoreDuplicates` で既存メールをスキップ）。`updateTeacher` は `.select("id")` で更新結果を検証（RLS による 0 行更新のサイレント失敗を検知）
- `src/app/teacher/export/actions.ts` — `countExportRows`（件数プレビュー、科目フィルタ対応）、`getGradeNames`（科目別グレード選択肢取得）、`getSubjects`（科目一覧取得）

全 Server Action で `verifyTeacher()` による教員権限チェックを実施。変更後は `revalidatePath` でキャッシュを無効化。

### パスエイリアス

`@/*` → `./src/*`

### 認証フロー

Supabase Auth 経由の Google OAuth。`middleware.ts` が `/student/*` と `/teacher/*` を保護し、未認証ユーザーを `/login` にリダイレクト。`/auth/callback` で認証コードをセッションに変換後 `/` へリダイレクト。ルートページ (`/`) で `teachers` → `students` の順にメール照合してロール判定・リダイレクト。どちらにも該当しない場合は「未登録」メッセージを表示（ログインループ防止）。

### 環境変数

`.env.local` に以下が必要:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`

E2Eテスト実行時は追加で以下が必要:
- `SUPABASE_SERVICE_ROLE_KEY`（Supabase ダッシュボード → Settings → API）
- `E2E_TEACHER_EMAIL` / `E2E_TEACHER_PASSWORD`（テスト用教員アカウント）
- `E2E_STUDENT_EMAIL` / `E2E_STUDENT_PASSWORD`（テスト用生徒アカウント）

### コアビジネスロジック

- **小テスト送信フロー**: クライアント側で即時採点（`quiz-logic.ts`）→ 結果表示 → Server Action (`quiz/actions.ts`) でサーバー側再採点（`verifyScore()`）・DB保存。合否判定は `grade_definitions.pass_score` に基づく。科目ごとに独立して管理。
- **グレード進級** (`grade-logic.ts`): 合格 + 本日未受験 → 連続日数+1 → 必要日数到達で昇級・リセット。不合格 → 連続日数を0にリセット。進捗は `student_subject_progress` で科目別に管理。
- **日次制限**: 1日1回のみ成績記録に保存（科目ごと）。`student_subject_progress.last_challenge_date` で判定。Asia/Tokyo タイムゾーン。
- **再受験**: `?retry=id1,id2,...&subject={subjectId}` クエリパラメータで同じ問題セットを再生成。結果は保存しない。
- **選択肢シャッフル**: `ShuffledChoice.originalIndex` で元のインデックスを追跡。回答は `originalIndex`（0-based）で管理。DB上の `correct_answer` は 1-based。
- **CSVインポート**: `parseCsvRows`（`csv-utils.ts`）でパース → クライアント側バリデーション+プレビュー → Server Action でバリデーション（`validation.ts`）→ DB 保存。問題は upsert、生徒は新規挿入と既存更新を分離。UTF-8/Shift_JIS 自動判定。
- **CSVエクスポート**: `/teacher/export` でフィルター選択 → 件数確認（Server Action、`head: true`）→ Route Handler (`/api/teacher/export`) でBOM付きUTF-8 CSVダウンロード。生徒一覧（統計付き）と受験記録詳細の2種類。

### データベース（Supabase）

テーブル: `subjects`、`students`、`student_subject_progress`、`teachers`、`grade_definitions`、`questions`、`quiz_records` — すべて RLS 有効。メールアドレスで Google アカウントと紐付け。RLS ポリシーは `auth.jwt() ->> 'email'` でメール照合。教員は全テーブルを閲覧・編集可能（`teachers` テーブルの存在チェック）。`teachers` テーブルの SELECT ポリシーは `auth.role() = 'authenticated'`（認証済みユーザーなら読み取り可能）— ロール判定（`/` ページ）で認証直後に参照するため、自己参照の循環を避ける設計。`subjects` も認証済みユーザーが SELECT 可能。`student_subject_progress` は生徒が自分の分のみ SELECT/UPDATE 可能。スキーマの SQL 定義は `PLAN.md` を参照。

### Supabase クエリの注意点

- **行数制限**: Supabase（PostgREST）はデフォルトで最大1000行しか返さない。`.limit()` を設定してもサーバー側の `max-rows` 設定で制限される。大量のレコードを取得する場合は `.range()` によるページネーションが必要（参考: `teacher/page.tsx` の `fetchAllRecentRecords()`）。
- **TIMESTAMPTZ フィルター**: `taken_at` 等の TIMESTAMPTZ カラムに対する `.gte()` / `.lte()` フィルターでは、日付文字列にタイムゾーンを明示する必要がある（例: `${date}T00:00:00+09:00`）。タイムゾーンなしの `YYYY-MM-DD` 文字列では正しくフィルタされない。
- **`.in()` の URL 長制限**: `.in()` に大量の UUID（1000件超）を渡すと PostgREST の URL 長制限を超えてリクエストが失敗する。フィルタなしなら `.range()` ページネーションで全件取得、フィルタありなら `.in()` を200件ずつバッチ分割して結合する（参考: `teacher/page.tsx` のグレード分布チャート取得）。
- **クエリ並列化**: 独立したクエリは `Promise.all` で並列実行してパフォーマンスを改善（参考: `student/page.tsx` の4クエリ並列取得、`student/history/page.tsx` の統計2クエリ並列実行）。
- **クエリエラーチェック**: Server Component のクエリ結果は `error` を必ずチェックし、エラー時は `throw new Error(...)` で `error.tsx` にフォールバックさせる。Server Actions のバッチ操作（削除・挿入）も中間エラーチェックを実施。
- **RLS サイレント失敗**: RLS 有効テーブルで UPDATE/DELETE ポリシーが未設定の場合、Supabase はエラーを返さず 0 行更新/削除する。新しい CRUD 操作を追加する際は対応する RLS ポリシーの存在を確認すること。`.update().select()` で更新結果を検証するとサイレント失敗を検知できる。

### アクセシビリティ（WCAG AA 準拠）

- **スキップナビゲーション**: 生徒/教員レイアウトに `sr-only focus:not-sr-only` スキップリンク配置、`<main id="main-content">` で遷移先を設定
- **フォーム**: 全 `<label>` に `htmlFor`、全入力に `id` を対応付け。ラジオグループは `<fieldset>/<legend>` で囲む
- **エラー・通知**: フォームエラーに `role="alert"`、CSVインポート結果に `role="status"`
- **テーブル**: 全 `<th>` に `scope="col"`、空ヘッダーに `aria-label`
- **ドロップダウンメニュー**: `aria-expanded`、`aria-haspopup`、`role="menu"`/`role="menuitem"`、Escape キーで閉じる
- **Pagination**: `<nav aria-label="ページネーション">`、`aria-current="page"`、前へ/次へに `aria-label`
- **コントラスト**: 低コントラストの `text-gray-300`/`text-gray-400`/`text-yellow-600` を修正済み
- **キャンセルボタン**: ナビゲーション用途のため `<Link>` に統一（`<button>` ではなく）
- **削除確認ダイアログ**: `ConfirmDialog` コンポーネント（`window.confirm()`/`alert()` を置き換え）。Escape/背景クリックで閉じる、`role="dialog"` + `aria-modal`
- **フォーカスリング**: `globals.css` で `input`/`select`/`textarea` にグローバルなフォーカスリングスタイルを適用（teal-500 の `box-shadow`）
- **タッチターゲット**: テーブル操作列の「編集」「削除」ボタンに `px-2 py-1` パディング + ホバー背景色。ログアウトボタンに `px-3 py-1.5 rounded-lg` + `hover:bg-gray-100`

### テストインフラ

- **Vitest**（ユニットテスト）: `vitest.config.ts`（`src/**/*.test.ts` と `src/**/*.test.tsx` を対象）
- **Supabase モック**: `src/test-utils/supabase-mock.ts` — `createMockSupabase()` でチェーン可能なクエリビルダーモックを生成。テーブル・操作ごとのレスポンス設定、`setTableResponse()` による動的切替に対応。`vi.mock("@/lib/supabase/server", () => mockModule)` で利用。
- **ユニットテスト対象**: lib モジュール（`quiz-logic`、`grade-logic`、`date-utils`、`csv-utils`、`validation`、`export-utils`）、Server Actions（`quiz/actions`、`questions/actions`、`students/actions`（`createStudent`/`updateStudent`/`deleteStudent`/`importStudents`）、`grades/actions`、`teachers/actions`（`createTeacher`/`updateTeacher`/`importTeachers`/`deleteTeacher`）、`export/actions`）
- **Playwright**（E2Eテスト）: `playwright.config.ts`、テストファイルは `e2e/` ディレクトリ
  - 3プロジェクト構成: `setup`（認証・シードデータ）、`teacher`（教員テスト）、`student`（生徒テスト）
  - 認証方式: Supabase Admin API でテストユーザー作成 → `signInWithPassword()` → storageState にクッキー保存（Google OAuth をバイパス）
  - `e2e/helpers/seed.ts`: `service_role` キーでテストデータ投入・クリーンアップ。テスト科目の upsert → グレード定義・問題・生徒・`student_subject_progress` の順に投入
  - `e2e/fixtures/test-data.ts`: テスト用定数（`TEST_SUBJECT` 固定UUID、問題ID 9001〜9010、一時データは 9100〜）。全テストデータに `subject_id` 付き
  - `e2e/global-setup.ts`: 認証セッション作成 + シードデータ投入（setup project として実行）
  - E2Eテスト対象: 教員ダッシュボード・生徒/問題/グレード管理（CRUD・科目選択）・CSVインポート・CSVエクスポート・生徒ダッシュボード（科目カード）・小テスト受験（`?subject=` 必須）・履歴表示（全18テスト）

### シードスクリプト

- `scripts/seed-quiz-records.sql` — ダミー受験記録の生成スクリプト（Supabase SQL Editor で実行）。過去30日分のデータを全生徒に対して約70%の参加率で生成。既存の `quiz_records` を全削除してから挿入する。本番環境では使用しないこと。
- `scripts/migrate-subjects.sql` — 複数科目対応マイグレーション（Supabase SQL Editor で実行）。`subjects` / `student_subject_progress` テーブル作成、既存テーブルへの `subject_id` 追加、`students` からの進捗カラム削除、RLS ポリシー設定。既存データはデフォルト科目「英語」に紐付け。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tnagano67) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
