## lean-experiments

> このファイルは、`lean-experiments` ディレクトリにおいて Gemini (Antigravity) が作業する際のローカルルールとコンテキストを定義する「種」です。必要に応じて追記・修正し、AIとの協働環境を育てていってください。

# GEMINI.md (Agent Context & Rules)

このファイルは、`lean-experiments` ディレクトリにおいて Gemini (Antigravity) が作業する際のローカルルールとコンテキストを定義する「種」です。必要に応じて追記・修正し、AIとの協働環境を育てていってください。

## 1. プロジェクトのコンテキスト
- **目的**: Lean 4 による数学の形式化（Zorn の補題、順序論など）の実験と、対応する LaTeX (`TeX/`ディレクトリ) ノートの作成。
- **主要な構成**:
  - `src/`: Lean 4 の証明・定義。
  - `TeX/`: 人間向けの解説ノート（LuaTeX を想定）。

## 2. Gemini の基本ルール（常に意識すべきこと）
1. **言語**: 返答や追加するコメントは基本的に日本語で行う。
2. **安全第一**: Lean の既存証明を壊さないよう、最小限の変更を心がける。`sorry` は極力増やさない。
3. **同期意識**: `.lean` ファイルの数学的主張を変更した場合、対応する `.tex` ファイルの更新が必要か確認・提案する。

## 3. 関連ドキュメントとの棲み分け
- 詳細な運用手順やコマンドは既存の `AGENTS.md` も参照する。（今後、ルールがまとまってきたらこの `GEMINI.md` へ統合・整理するのもおすすめです）

## 4. TeXファイル作成時の定型ルール
TeXファイルを作成する際は、指示がなくても以下の要件を満たすこと：
1. **数学的な解説**を詳しくおこなう。
2. **図式**（`tikz-cd` など）を効果的に用いて可視化する。
3. 文章は**日本語**で作成し、コンパイラは **LuaTeX** を想定する。
4. 必ず文書の冒頭付近に以下の「AI assistance disclosure」を含める：
   ```latex
   \begin{center}
   \small
   \textit{AI assistance disclosure:}
   Lean ソースコードは Claude (Anthropic) で骨格を生成し、
   Codex (OpenAI) で修正した。
   \TeX 文書は Antigravity (Google DeepMind) で生成した。
   著者による加筆・修正は行っていない。
   内容の正確性は保証されず、誤りがあれば著者の責任である。
   \end{center}
   \begin{abstract}
   \end{abstract}
   ```

---
## 🌱 育て方（ここに追記していくためのメモ）
今後、以下のような内容をこのファイルに書き足していくことで、Geminiの精度が向上します：
- [ ] TeXのコンパイル時に使用する具体的なコマンド (`lualatex ...` など)
- [ ] Leanでよく使う・優先して使うべき Mathlib の補題リスト
- [ ] AIがよく間違えるポイントや、過去の `error-fix-log.md` から得た教訓
- [ ] TeXの記述における好みの記法（例: 空集合は `\emptyset` ではなく `\varnothing` を使う、など）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/suii00)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/suii00)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
