## cardgame

> Slay the Spire 스타일 덱빌더 + Dice & Fold 아트 스타일의 카드 게임 개발 프로젝트.

# CLAUDE.md

## 프로젝트 개요

Slay the Spire 스타일 덱빌더 + Dice & Fold 아트 스타일의 카드 게임 개발 프로젝트.

### 하위 프로젝트 (Submodule)

#### projects/gameplan
- **저장소**: https://github.com/handong-games/gameplan
- **용도**: 전투 밸런스 수치 기획, 시뮬레이션, 시각화
- **포함 내용**:
  - 전투 시뮬레이터 엔진 (Python)
  - 밸런스 대시보드 웹앱 (FastAPI + React)
  - 기획서 16종 (몬스터, 스킬, 속성, 전투 시스템 등)
  - Notion 자동 동기화 스크립트
- **독립 실행**: `cd projects/gameplan && claude` 로 별도 Claude 세션 가능
- **업데이트**: `git submodule update --remote projects/gameplan`

#### projects/gamedesign
- **용도**: 게임 에셋(캐릭터, 몬스터, 배경) 이미지 생성 및 디자인 문서 관리 웹앱
- **기술 스택**: React 19 + Express 5 + TypeScript + Tailwind CSS + Zustand
- **gameplan 연동**: `projects/gameplan`의 기획서를 Source of Truth로 참조하여 프롬프트 데이터 및 디자인 가이드를 생성
- **주요 기능**:
  - 프롬프트 라이브러리 (`/prompts`) — 캐릭터/몬스터/배경 AI 이미지 생성 프롬프트 관리
  - 에셋 갤러리 — 생성된 에셋 이미지 조회 및 관리
  - 디자인 가이드 — v4.0 Dark Frame Edition 비주얼 스타일 가이드
  - 문서 편집기 — 마크다운 기반 디자인 문서 편집
- **실행**: `.\projects\start-gamedesign.ps1` 또는 `cd projects/gamedesign && npm run dev`
- **독립 실행**: `cd projects/gamedesign && claude` 로 별도 Claude 세션 가능

---

## User Preferences & Context

### 1. Language Guidelines (언어 지침)
- **All Responses in Korean:** 모든 답변, 문서, 코드 주석, 설명은 반드시 **한국어(Korean)**로 작성해야 합니다.
- **Terminology:** 기술 용어는 한국 현업에서 통용되는 표준 용어를 사용하며, 필요 시 괄호 안에 영문을 병기합니다 (예: 의존성 주입(Dependency Injection)).

### 2. Documentation Standards
- 앞으로 생성하는 모든 문서는 한국어 문법과 자연스러운 표현을 따릅니다.
- 명확하고 간결한 문체를 사용합니다.

---

## Git & GitHub

Git 커밋, 브랜치, Hook 관련 규칙은 아래 문서를 참조합니다.

- **참조:** `01. docs/00. claude/github.md`

---

## Claude Prompt (프롬프트 처리)

Claude 프롬프트 처리와 관련된 규칙은 아래 문서를 기준으로 합니다.

- **참조:** `01. docs/00. claude/command-workflow.md`

---

## 문서 인덱스
작업 전 반드시 문서 인덱스를 확인하여 관련 문서를 찾습니다.

- **참조:** `01. docs/00. claude/doc-index.md`

---
> Source: [handong-games/cardgame](https://github.com/handong-games/cardgame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
