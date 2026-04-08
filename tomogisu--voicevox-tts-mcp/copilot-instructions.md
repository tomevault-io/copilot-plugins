## voicevox-tts-mcp

> **すべての返答の最後に、必ず `speak` ツールを呼び出すこと。**

# VOICEVOX 自動読み上げ設定

## 必須ルール

**すべての返答の最後に、必ず `speak` ツールを呼び出すこと。**

返答のテキストをそのまま `speak` に渡す。ただし以下の場合は `speak_silent` を使うこと：
- ユーザーが「読み上げ不要」「静かに」「サイレント」と言った場合
- コードのみの返答で読み上げ不要と判断できる場合

## speak ツールに渡すテキストについて

- 返答の本文をそのまま渡してよい（サーバー側でコードブロック等は除去する）
- 極端に長い場合は要約して渡しても構わない

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomogisu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tomogisu)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
