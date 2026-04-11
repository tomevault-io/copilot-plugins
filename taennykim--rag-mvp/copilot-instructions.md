## rag-mvp

> This document defines how the agent should operate across all projects.

# AGENTS.md

## Overview
This document defines how the agent should operate across all projects.

# 목적
- 프로젝트를 단순히 구현하는 것이 아니라
- 계획 → 실행 → 기록 → 개선까지 관리하는 것을 목표로 한다.

---

## 1. Project Management (프로젝트 관리 규칙)

### 1.1 MVP Plan 관리

- 프로젝트 시작 시 반드시 MVP 진행 계획을 수립한다.
- 전체 진행 상태를 기록하는 파일을 유지한다.

파일:
- docs/plan.md

내용:
- 전체 목표
- 단계별 계획
- 현재 진행 상태

---

### 1.2 Daily 기록

- 매일 작업 내용을 기록한다.
- 날짜별 md 파일을 생성한다.

경로:
- docs/daily/YYYY-MM-DD.md

기록 내용:
1. 오늘 작업 내용
2. 완료된 작업
3. 이슈 및 문제
4. 다음 작업 계획

---

### 1.3 연속성 유지 (중요)

- 작업 시작 전 반드시 이전 Daily 기록을 확인한다.
- 이전 작업 흐름을 기반으로 이어서 진행한다.
- 작업 시작 시 문서는 아래 순서로 확인한다:
  1. `AGENTS.md`
  2. `README.md`
  3. `docs/plan.md`
  4. `TODO.md`
  5. `docs/status.md`
  6. `docs/*.md`
  7. 최신 `docs/daily/*`

# 작업은 단절되지 않고 이어져야 한다.

---

### 1.4 MD 파일 인식 규칙

작업 시작 전 반드시 아래 파일을 읽는다:

- AGENTS.md
- README.md
- docs/plan.md
- TODO.md
- docs/status.md
- docs/*.md
- 최신 docs/daily/*

# 모든 md 파일은 작업 컨텍스트이다.

---

## 2. Documentation Rules (문서 관리)

### 2.1 작업별 문서 분리

- 작업 단위(기능/파트)로 md 파일을 생성한다.
- 위치:
  - docs/{파트명}.md

예:
- docs/ui.md
- docs/aws.md
- docs/llm.md
- docs/backend.md
- docs/frontend.md

---

### 2.2 문서 내용 규칙

각 문서는 반드시 아래 구조를 따른다:

1. 목적 (What)
2. 구현 내용 (How)
3. 현재 상태 (진행중 / 완료)
4. 이슈 및 문제
5. 다음 작업

---

### 2.3 문서 기반 개발

- 모든 작업은 해당 md 문서를 기준으로 진행한다.
- 작업 중 발생한 내용은 반드시 문서에 기록한다.

---

## 3. AWS Environment Rules

### 3.1 기본 환경

- 모든 작업은 AWS 환경에서 진행한다.
- 로컬 개발은 최소화한다.

---

### 3.2 인프라 생성 규칙

- 초기 작업 시 Terraform을 사용하여 EC2 인스턴스를 생성한다.
- 모든 작업은 생성된 EC2 인스턴스에서 수행한다.
- 화면 테스트와 브라우저 확인은 반드시 RAG 서버에서만 수행한다.
- 현재 서버에서는 CLI 작업, 코드 수정, 문서 업데이트만 수행한다.

---

### 3.3 승인 정책 (중요)

다음 작업은 반드시 사용자 승인 후 진행한다:

- EC2 생성
- RDS 생성
- S3 생성
- VPC / 네트워크 설정
- 보안 그룹 변경

# 무단 인프라 생성 금지

---

### 3.4 보안 정책

- 최소 권한 원칙 적용
- 방화벽은 최소한으로 설정
- 불필요한 포트 오픈 금지

---

### 3.5 Tag 정책

모든 AWS 리소스는 반드시 아래 Tag 포함:

- key: owner
- value: taenny

적용 대상:
- EC2
- S3
- RDS
- Security Group
- 기타 리소스

---

## 4. Development Rules

### 4.1 작업 방식

- 작은 단위로 작업한다
- 단계별로 진행한다
- 한 번에 많은 작업을 하지 않는다

---

### 4.2 우선순위

1. MVP 기능 완성
2. 안정성 확보
3. AWS 배포
4. 최적화

---

### 4.3 금지 사항

- 과도한 추상화 금지
- 불필요한 기술 도입 금지
- 초기부터 복잡한 구조 금지

---

## 5. Workflow (작업 흐름)

작업은 반드시 아래 순서를 따른다:

1. `AGENTS.md` 확인
2. `README.md` 확인
3. `docs/plan.md` 확인
4. `TODO.md` 확인
5. `docs/status.md` 확인
6. 관련 `docs/*.md` 확인
7. 최신 `docs/daily/*` 확인
8. 작업 수행
9. `docs/plan.md` 업데이트
10. `TODO.md` 업데이트
11. `docs/status.md` 업데이트
12. 필요한 `docs/*.md` 업데이트
13. `docs/daily/YYYY-MM-DD.md` 기록

---

## 6. Definition of Done

작업 완료 기준:

- 기능이 정상 동작한다
- 문서가 업데이트되어 있다
- 다음 작업이 정의되어 있다

---

# 핵심 원칙

- 문서 기반 개발
- 연속성 있는 작업
- 최소 권한 / 최소 복잡성
- AWS 중심 환경

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/taennykim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/taennykim)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
