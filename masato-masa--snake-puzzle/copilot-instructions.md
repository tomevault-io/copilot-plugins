## snake-puzzle

> Read the exact versioned docs at https://docs.expo.dev/versions/v57.0.0/ before writing any code.

# Expo HAS CHANGED

Read the exact versioned docs at https://docs.expo.dev/versions/v57.0.0/ before writing any code.

# このプロジェクト

- ゲームロジックは `src/engine/` に純 TypeScript で置く。React / React Native を import しないこと。
  UI・ソルバー・エディタ・テストがすべてここを共有するため、ルール変更は必ずここに入れる。
- 移動・クリア判定を変えたら `__tests__/engine.test.ts` の基準例（5x5 で ↓→ の 2 手クリア）を必ず通す。
- ステージを追加・変更したら `npm test` を通す。全ステージについて
  「定義が健全」「解ける」「`parMoves` が最少手数と一致」「`difficulty` が analyzeLevel の★と一致」
  「品質監査（auditLevel）に指摘がない」を検証している。
- **ステージ設計は `docs/level-design.md` に従う。** 1 面につき問いを 1 つ、逆算で作る。
  ソルバーは検証に使い、生成は候補出しにとどめる（採否は人が決める）。
  監査に引っかかる面を通したいときは、`level.audit` に理由を書いて明示的に例外にする。
- ギミック（砂マス・スイッチ/ゲート・ワープ穴）は `slidePath` の中だけで完結させる。
  ゲートの開閉は移動開始時点で固定せず、頭が1マス進むたびに判定し直す。
  スイッチに乗っている間だけゲートは開く。自分がスイッチから外れた瞬間、
  同じ一手の途中でもそのゲートは（動いている本人にとっても）すぐ閉じる。
  長い蛇は自分の体が壁になることもあるが、これも同様に頭が動くたびに数え直す
  （移動する瞬間は自分の胴体が壁だったが、動いた瞬間にその壁が消える、というパターンもあり得る）。
- 難易度は `analyzeLevel`。盤面の広さではなく「手数 × log2(分岐)」を主軸にしている。
  この方針を変えると既存ステージの `difficulty` が総崩れするので注意。
- 盤面サイズはステージ定義の `rows` / `cols` に従う。コードに盤面サイズを固定で書かない。
- ジェスチャーは React Native 標準の `PanResponder`、アニメーションは標準の `Animated` を使っている
  （reanimated / gesture-handler のワークレットには依存していない）。
- プレイ画面の中身は `src/components/game-view.tsx` に集約している。
  通常ステージ（`app/game/[levelId].tsx`）とデイリー（`app/daily.tsx`）はここに level を渡すだけ。
- ヘビは「丸い体節＋つなぎの四角」を、輪郭パス → 本体パス の 2 回に分けて描いている。
  体節ごとに輪郭を描くと継ぎ目が出るので、この 2 パス構成は崩さないこと。
- 移動アニメーションは `move()` が返す `path` を使い、`src/lib/snake-track.ts` の道すじに沿って
  各体節を 1 マスずつ進める（多点 interpolate）。始点と終点だけを補間すると曲がり角を斜めに
  突っ切ってしまうので、この方式を変えないこと。`__tests__/snake-track.test.ts` が守っている。
- デイリーは日付を seed にした決定的生成。`buildDailyLevel` の中身を変えると過去の日付の問題も変わる。

---
> Source: [masato-masa/snake-puzzle](https://github.com/masato-masa/snake-puzzle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
