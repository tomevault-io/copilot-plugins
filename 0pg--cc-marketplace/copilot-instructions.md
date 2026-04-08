## cc-marketplace

> **플러그인 수정 시 반드시 버전을 올려야 합니다.**

# Claude Plugins Repository

## 버전 관리

**플러그인 수정 시 반드시 버전을 올려야 합니다.**

- 버전 파일: `plugins/{plugin-name}/.claude-plugin/plugin.json`
- 형식: [SemVer](https://semver.org/) (MAJOR.MINOR.PATCH)
  - PATCH: 버그 수정, 문서 오타
  - MINOR: 기능 추가, 기존 기능 개선
  - MAJOR: 호환성 깨지는 변경

## Marketplace 등록

**신규 플러그인 추가 시 반드시 marketplace에 등록해야 합니다.**

- Marketplace 파일: `.claude-plugin/marketplace.json`
- 등록 항목:
  ```json
  {
    "name": "plugin-name",
    "description": "플러그인 설명",
    "version": "1.0.0",
    "author": { "name": "jhk" },
    "source": "./plugins/plugin-name",
    "category": "development"
  }
  ```
- 버전 업데이트 시에도 marketplace.json의 version 필드 동기화 필요

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/0pg)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/0pg)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
