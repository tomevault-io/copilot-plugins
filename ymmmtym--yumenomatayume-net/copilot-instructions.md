## yumenomatayume-net

> For comprehensive details about the project's tech stack, structure, setup, development workflow, deployment, and scripts, please refer to README.md.

# yumenomatayume.net Blog Management Agents

## Project Documentation

For comprehensive details about the project's tech stack, structure, setup, development workflow, deployment, and scripts, please refer to README.md.

## Content Management Agents

### 1. Linear Issue to Blog Article

**Trigger:** "linearの[ISSUE-ID]からブログ記事を書いて" または "create blog from linear [ISSUE-ID]"

**Workflow:**
1. Linear issue の詳細を取得
2. issue 内の画像を `public/images/` にダウンロード
3. 既存記事の形式に合わせて `app/content/blog/` にブログ記事を作成
4. 文章を校正
5. 画像パスを記事内で適切に参照
6. Linear の gitBranchName を使用してブランチを作成
7. 記事ファイルをコミット（メッセージ: `feat: [記事タイトル]の記事を追加`）
8. PR を作成（ユーザーが記事内容を確認後にマージ）
9. マージは `gh pr merge -md --auto` コマンドを実行する
10. **マージ完了後、Linear issue にコメントで公開報告（PR と記事 URL を含む）**

**Settings:**
- Blog directory: `app/content/blog/`
- Image directory: `public/images/`
- Article format: MDX with frontmatter
- Required frontmatter: title, description, pubDate, tags
- File naming: `{issue-title-kebab-case}.md`
- Image naming: `{descriptive-name}.png`

### 2. Article Proofreading Agent

**Trigger:** "記事を校正して" または "全記事の校正をして"

**Standards:**
- タイトル末尾に絵文字追加 (統一感のため)
- ですます調への統一
- 絵文字を適度に使用 (親しみやすさ向上)
- 読みやすい文章構造
- description の追加 (SEO向上)

### 3. UI/UX Enhancement Agent

**Trigger:** "UIを改善して" または "デザインを変更して"

**Recent Implementations:**
- タグ一覧の折りたたみ機能 (details要素使用)
- ダークモード対応
- レスポンシブデザイン
- 記事カードのホバーエフェクト

**CSS Guidelines:**
- Tailwind CSS v4使用
- `@theme` ブロックにはカスタムプロパティのみ
- 通常のCSSルールは `@theme` ブロック外に記述

## Content Guidelines

### Article Format
```markdown
---
title: "記事タイトル 🎯"
description: "記事の説明文"
pubDate: "YYYY-MM-DD"
tags: ["tag1", "tag2"]
heroImage: "https://cloudinary-url" # optional
---

記事内容...
```

### Writing Style

- ですます調で統一
- 絵文字を適度に使用
- 技術記事は具体例とコード例を含める
- 読者にとって分かりやすい構造

### Tag Management

- タグ一覧はデフォルト非表示 (details要素)
- 記事カードでは最初の3つのタグのみ表示
- タグページでの記事フィルタリング対応

## Development Workflow

### Branch Naming Convention
- Feature branches: `issue/{issue-number}`
- Example: `issue/1`, `issue/2`, `issue/3`

### Git Workflow
1. Create branch: `git checkout -b issue/1`
2. Commit with reference: `git commit -m "feat: 機能名 (#1)"`
3. Push and create PR: `gh pr create --title "機能名" --body "Closes #1"`
4. PR merge automatically closes the issue

### Commit Message Format
- `feat: 新機能 (#issue-number)`
- `fix: バグ修正 (#issue-number)`
- `docs: ドキュメント更新 (#issue-number)`
- `style: スタイル調整 (#issue-number)`

### Commit Best Practices
- メッセージは英語で記述
- **機能ごとに細かくコミット** - 1つのコミットは1つの機能や修正に集中
- 複数の変更がある場合は `git reset` で分割してコミット
- 例: SearchBox作成、Header統合、CSS追加を別々のコミットに分ける
- コミットメッセージは変更内容を具体的に記述
- **記事追加時は簡潔な1行メッセージ** - プライベートツール（Linear等）の詳細情報は不要
- 記事追加の場合は "Add [article-topic] article" 形式で十分（内容は記事を見れば分かるため）

## Deployment & Hosting

**Platform:** Cloudflare Workers
**Install Command:** `bun install -D ${PACKAGE_NAME}`
**Build Command:** `bun build`
**Output Directory:** `dist/`
**Environment:** Node.js

**Features:**
- RSS フィード (`/feed`)
- サイトマップ (`/sitemap.xml`)
- 記事検索・フィルタリング
- ダークモード切り替え
- ソーシャルシェア機能

## Maintenance Tasks

### Regular Tasks

- 記事の校正・更新
- 画像の最適化
- タグの整理
- SEO改善

### Development Tasks

- 依存関係の更新
- パフォーマンス最適化
- 新機能の実装
- バグ修正

## Contact & Links

- Email: yumenomatayume@yumenomatayume.net
- GitHub: @ymmmtym
- Blog: https://yumenomatayume.net

---
> Source: [ymmmtym/yumenomatayume.net](https://github.com/ymmmtym/yumenomatayume.net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
