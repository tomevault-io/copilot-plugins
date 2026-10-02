## sampler81800

> 音MAD用ワンショット・サンプラープラグイン（VST3 / CLAP）。

# OtoMadSampler 開発規約

音MAD用ワンショット・サンプラープラグイン（VST3 / CLAP）。
D&D読み込み・ピッチオフセット・ポルタメント（カーブ調整可）・
ピッチシフトアルゴリズム選択・長さ制御（durationMode）を主軸にする。

**設計の唯一の正は [docs/DESIGN.md](docs/DESIGN.md)。** 本ファイルはそこから抽出した
「破ってはいけない不変条件」と作業手順の要約。判断に迷ったら DESIGN.md の該当節を読む。

- 言語: C++20 / フレームワーク: JUCE 8 / CLAP: clap-juce-extensions
- ビルド: CMake + FetchContent / テスト: Catch2 + pluginval
- 実装は DESIGN.md §7 のフェーズ単位で進める。各フェーズの受け入れ条件を満たしてから次へ。

## ビルド

```
cmake -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build --config RelWithDebInfo -j
ctest --test-dir build --output-on-failure
pluginval --strictness-level 10 --validate build/.../OtoMadSampler.vst3
```

各フェーズ末尾で必ず `ctest` と `pluginval` を通すこと。

## 不変条件 (これを破る変更は必ず事前に相談すること)

### スレッド
1. `processBlock` およびそこから呼ばれる全関数で、
   new/delete/malloc/lock/ファイルI/O/ログ出力/std::string操作を行わない。
   バッファは prepareToPlay で最大サイズ分を確保しきる。
2. REAPER API へのアクセスは `host::ReaperApi` の外で行わない。
   取得結果は常に null チェックしてから使う。REAPER 上でも null になりうる。
   プロジェクト状態を変える API はオーディオスレッドから呼ばない。
3. GUIからオーディオバッファを直接読まない。peaks配列 / atomic のみ参照。

### DSP
4. ピッチは常に半音（対数）ドメインで補間する。Hz直線補間は禁止。
5. **pitchRatio と timeRatio を癒着させない。** 長さ保持系エンジンの内部で
   `timeRatio = 1/pitchRatio` を決め打ちしてはいけない。
   解析hopは `hopSynth * timeRatio / pitchRatio`（掛けるのではなく割る）。
6. **エンジンはトリムを知らない。** trim/loop/reverse は `SourceReader` が吸収し、
   エンジンから見た位置0は常にトリム開始点。終端判定も原音長ではなくトリム長と比較する。
7. **エンジンは n == 1 で呼ばれても正しく動く。** また renderSlice を n == 0 で呼ばない。
   「ブロック先頭でフレームを1つ処理する」構造にしてはいけない。
8. 素材を読み切ってもボイスを即座に落とさない。`getTailSamples()` に
   固定レイテンシ整列バッファ分 `(FIXED_LATENCY − intrinsicLatency)` を足してドレインする。
9. **エンジンの可変状態（位相配列・OLA・FIFO・テンプレート）はボイス固有。**
   単一エンジンインスタンスを全ボイスで共有しない。共有してよいのは読み取り専用の
   `EngineResources`（sinc表・窓・FFTプラン・スクラッチ）だけ。
10. **フレーム系エンジンは pitchRatio をフレーム先頭値で固定**して1フレームを処理する
    （hop計算とリサンプルで同じ値を使う）。サンプル精度が要るなら Varispeed。

### パラメータ
11. パラメータは必ず APVTS 経由。生メンバ変数で状態を持たない。
12. **パラメータの個数・レンジ・選択肢をホストによって変えない。**
    REAPER 専用機能もパラメータ定義上は常に存在させ、UIでグレーアウトするだけにする。
13. トリムは正規化値 (0..1) で保持する。サンプル数で持つとサンプル差し替えで意味が壊れる。
14. スムージングは DESIGN.md §3.8 の表に従う。全部に掛けるのは誤り。

### 挙動
15. エンジンが使えない場合は代替エンジンにフォールバックする。**無音を返さない。**
    フォールバック中であることを UI に表示する。
16. サンプルは原音SRで保持・埋め込みする。SR変更時は原音から一度だけ変換し、
    再生用バッファを再リサンプルしない（二重変換禁止）。
17. `setLatencySamples()` を鳴動中に変えない。固定レイテンシ方式を守る。
18. 新しい IPitchEngine を追加したら、tests/ のパラメータ化テストに必ず登録する
    （ピッチ精度・出力長・ブロック分割不変性・ボイス状態独立性の4点）。

## コーディングスタイル
- C++20。JUCEの命名規則に合わせる。
- コメントは日本語可。DSPの数式は式そのものをコメントに残す。
- **非ASCIIを含む文字列リテラルを `juce::String` に渡すときは必ず `otomad::u8()`
  (`core/Utf8.h`) を通す。** `juce::String (const char*)` は `CharPointer_ASCII` 実装で、
  バイト値をそのまま code point にする（＝Latin-1 解釈）。ソースは `/utf-8` でコンパイル
  しているので直接渡すと化ける（"直" → "ç›´"）。デバッグビルドは jassert で気づけるが、
  **配布ビルド(RelWithDebInfo)では assert が消えるので黙って化ける。**
  コメント内の日本語は対象外（コンパイルされないため）。
- **外部から受け取った文字列も同じ罠にかかる。** REAPER API のモード名、ffmpeg の出力、
  ファイルパスなどは UTF-8 で来る。`std::string` は**そのまま渡せば UTF-8 として扱われる**が、
  **`.c_str()` を付けると `const char*` オーバーロードに落ちて ASCII 扱いになる**。
  `const char*` しか無いときは `juce::String::fromUTF8()` を使う。
  （リテラル走査だけでは見つからないので、`.c_str()` を書いたら必ず疑うこと）
- 1ファイル400行を超えたら分割を検討。
- 「なぜそうしたか」が非自明な箇所には理由をコメントに残す。
  特に符号・順序を間違えやすい式（hop計算、位相アンラップ）は導出ごと書く。

## 設計書
docs/DESIGN.md が唯一の正。実装が設計と乖離したら、
コードを直すか設計書を更新するかを必ず明示して判断を仰ぐ。

---
> Source: [neon-uriel/Sampler81800](https://github.com/neon-uriel/Sampler81800) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
