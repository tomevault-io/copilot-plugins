## game-pipeline-check

> > 全 AI 共通の作法。Claude は CLAUDE.md の `@AGENTS.md` で、Gemini は設定(contextFileName)でこのファイルを読む。Phaser の API は `.claude/skills/` の SKILL.md を参照。

# AGENTS.md

> 全 AI 共通の作法。Claude は CLAUDE.md の `@AGENTS.md` で、Gemini は設定(contextFileName)でこのファイルを読む。Phaser の API は `.claude/skills/` の SKILL.md を参照。

## これは何か
- Mukkii の個人開発の作品 1 本。まずパイロット版(遊べるモック)を最短で公開するための repo。
- 企画は `SPEC.md`。読んでから始めろ。冒頭に「追加で読むルール」があればそれも読め。書いてないことは自分で決めろ。

## 進め方(最重要)
1. **止まるな。** 伝えられたゴールまで、だめもとで作り切れ。途中で確認を取りに来るな。
2. **ゴールは改造の土台。** 「◯◯みたいなの」は、そっくりを仕上げるのではなく、人間がそこから改造できる状態まで作ること。
3. **質問は溜めろ。** 迷ったら仮決めして進み、`QUESTIONS.md` に「何を仮決めしたか、他の選択肢」を並べろ。人間は後でまとめて答える。
4. **複数案は `variants/a`, `variants/b` …** に、それぞれ動く状態で置け。
5. **パイロットは製品ではない。** セキュリティ監査、網羅テスト、文書の肥大化はするな。動いて公開できればよい。
6. 終わったら `HANDOFF.md` を更新しろ(現状、次の一手、既知の問題。別の AI が引き継ぐ前提で)。

## 技術の既定値
- Phaser 4 + Vite + TypeScript。テキスト中心の ADV や超小型なら素の canvas / DOM でもよい(SPEC.md で指定)。
- 3D は動きの激しいものは Babylon.js、見せるのが主なら Three.js。
- `src/core/`(save, i18n, audio, input, demo, meta)を先に使え。同じものを作り直すな。
- 型で止まるな。動く方を優先。ビルド出力は `dist/`、Vite の `base` は `./`。
- `?auto=1` で自動プレイが走るようにしろ(`core/demo.ts` に update を実装)。CI の確認と宣伝動画に使う。
- `?lang=en` で英語になるようにしろ。文字列は `i18n` を通せ。

## 素材
- 自作より外部素材を優先。順序: 海外 CC0(Kenney, Poly Haven, Freesound の CC0)→ 日本発の無料(索引の規約範囲)→ 契約済み有料 → AI 生成 → それでも駄目なら人間に聞く(QUESTIONS.md に「欲しい絵の説明 + 試した候補」)。
- 使った素材は全部 `CREDITS.md` に出典・規約・クレジット文・(AI 生成なら)どの AI かを書け。書いてない素材は使うな。
- 拾った素材への手入れ(文字入れ、色替え、切り抜き)は Pillow / ImageMagick / Blender で自分でやれ。

## 公開
- `docs/PUBLISH.md` を日本語と英語で埋めてから公開しろ(タイトル、一言、説明、操作、タグ)。
- main に push すれば Pages に出る。itch.io は `publish-itch` の手動ボタン。
- `.github/` を変える時は理由を `QUESTIONS.md` に書いてから変えろ。

## 文書
- `SPEC.md` の挙動を変えたら `SPEC.md` を直せ。長くするな。
- 設計判断は `DECISIONS.md` に 3 行で追記。書き換え禁止。
- 汎用に使える道具ができたら `QUESTIONS.md` に「dev-tools へ昇格候補」と書け。

## 禁止
- テストを消す・スキップする。ライセンス不明の素材。秘密情報のコミット。
- 同じ作品 repo に同時に push する経路を 2 つ作る(サブエージェントや他セッションへの質問は自由。push する者は 1 つ)。
- 会社・仕事関係のフォルダやドライブへの書き込み(この repo の外は触らない)。

## ローカル
```
npm ci && npm run dev      # 開発
npm run build              # dist/
node tools/check.mjs       # 起動確認 + スクショ
```

---
> Source: [mukkii-game/game-pipeline-check](https://github.com/mukkii-game/game-pipeline-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
