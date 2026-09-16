## sam2-1-food-finetuning

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md



This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
必ずserena MCPが日本語で対応すること！
日本語で応答すること！
Web searchの際は、必ずo3-query MCPを利用すること！

## o3-query MCP について
エラーや未知の質問、技術的な調査が必要な場合は**必ず**o3-queryツールを呼び出し、
返ってきたクエリを私に表示して停止してください。
必ずクエリ生成→人間が結果を入力のフローに従ってください
※Queryの最後に「これらについて、Qwen2.5-VL、SAM2.1、LISA等の公式実装を参考にした解決策を教えてください。」という文言を入れること

### 使用例
- 技術的なエラーの解決方法を調べたい時
- 最新の実装方法を調査したい時  
- 公式ドキュメントやベストプラクティスを確認したい時
- 具体的なライブラリやモデルの使用方法を調べたい時


[命令]
md_files/spec1.mdの実装プランを参考に実装を進めてほしい。実装プランで曖昧な部分や細かな実装方法がわからない部分は、作業の途中でも積極的にAIに質問するのであなたの方でQuery prompt作成してmd_files/に保存して作業を止めて。

※実装の際に注意すること
・簡易な実装でとりあえず走るコードは必要ない、本質的に目標を達成するコードが欲しい
・自信のない部分は適宜正規（Qwen, SAM, Huggingface, Pytorchなど公式の実装）の実装をo3 queryでしらべながら予想や自前の実装を少なくして実装すること
・o3 queryを積極的に行い、Qwen, SAMの正規の実装をできるだけ用いること
・フォールバック的もしくはダミーコードはエラーを隠蔽するので適切にエラーを出して止め、次のデバッグに繋がる情報を提供するように修正すること

---
> Source: [SoyaOda/sam2_1_food_finetuning](https://github.com/SoyaOda/sam2_1_food_finetuning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
