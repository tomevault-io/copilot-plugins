## tmrvc

> この文書は、TMRVC プロジェクトにおいて一貫性のある高品質な開発を維持するための技術的・運用的指針を定義する。

# TMRVC 開発ガイドライン (GEMINI.md)

この文書は、TMRVC プロジェクトにおいて一貫性のある高品質な開発を維持するための技術的・運用的指針を定義する。

## 1. データ整合性と前処理 (Data & Preprocessing)

- **音素 ID の品質**: UCLM v3 (Pointer Mode) の学習において、`phoneme_ids.npy` の品質はモデルの性能に直結する。前処理（Preprocessing）における G2P 変換と音素マッピングの正確性を最優先せよ。
- **TTS 教師あり学習**: 高品質な合成を保証するため、学習パイプライン実行時は可能な限り `--require-tts-supervision` を有効化し、テキストと音声の対応が明確なデータセットを優先的に使用せよ。
- **ディレクトリ管理の自動化**: データセットの同期や管理は `dev.py` を活用し、手動編集によるミス（パスの不整合など）を最小限に抑えよ。

## 2. 推奨アーキテクチャ (Modern Architecture Standards)

- **ポインタ・セマンティクス**: 進捗（Progress）は「速度（Velocity/Rate）」としての予測を標準とし、推論エンジン側で適切に時間積分を行う設計を維持せよ。
- **時間認識 RoPE**: ポインタが静止している間も Transformer が時間を認識できるよう、フレームオフセットを RoPE インデックスに考慮することを推奨する。ただし、既存モデルとの互換性が優先される場合は構成に応じて調整せよ。
- **話者 FiLM 注入**: 話者情報の注入には **FiLM (Feature-wise Linear Modulation)** の採用を標準とする。これにより、深い次元での話者性の維持を図る。
- **滑らかな停止条件 (EOS)**: 最後の音素のあとに適切な Tail Buffer（余韻）を設け、不自然なブツ切りを防止するよう配慮せよ。
- **MAS の安定化**: 教師なしアライメント（MAS）の距離計算において、特徴量の正規化等による安定性の確保を検討せよ。

## 3. 開発と運用の効率化 (Operational Efficiency)

- **TRAIN_GUIDE.md の参照**: 学習フローは `TRAIN_GUIDE.md` を基本としつつ、最新の実験結果に基づいた改良を歓迎する。
- **dev.py による開発効率の向上**: 全ての操作は `dev.py` 経由で行うことが推奨されるが、特定のデバッグや特殊なケースでは個別スクリプトの使用を妨げない。
- **キュレーション・フローの活用**: 収集した生データは AI キュレーション（Ingest -> Score -> Export）を経て、高品質なデータセットへと昇格させるサイクルを確立せよ。
- **検証の習慣**: コード変更後は `scripts/demo/tts_demo.py` 等で基本的な動作確認を行い、音声の品質（振幅、ノイズ等）および推論速度（RTF）に異常がないことを確認せよ。
- **透明性の確保**: 重要なアルゴリズムの変更や大規模な学習を行う際は、その意図と期待される効果を明確に説明せよ。

## 4. 継続的な改善

- 理論的な裏付けに基づいた柔軟な修正、広範なテスト、および保守性の高い実装を重視せよ。

---
**最高水準の音声合成（SOTA）を目指し、開発のしやすさと品質を両立させよ。**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kjranyone)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kjranyone)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
